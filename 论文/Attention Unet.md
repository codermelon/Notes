# Abstract

- 提出了一种用于医学成像的新型注意门（AG）模型，该模型可以自动抑制输入图像中的不相关部分，同时突出显示对特定任务有用的显著特征。



# Introduction

- 人工标记大量的医学图像容易出错，自动医学图像分割得到广泛研究
- 具有AGs的CNN模型可以以类似于FCN模型的方式从头开始训练，并且AGs可以自动学习专注目标
- 在保持高预测精度的同时，可以消除使用外部器官定位模型的必要性



## Contributions

- 提出了基于网格的划分，允许注意力系数更具体地针对局部区域
- 首次在医学图像中的CNN中使用软注意力
- 改善了Unet模型，提高模型对前景像素的敏感性



## Methodology

![image-20231107111551285](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231107111551285.png)

图1：所提出的注意力U-Net分割模型的框图。在网络的编码部分（例如H4=H1/8），输入图像按每个比例按因子2进行逐步滤波和下采样。Nc表示类的数量。注意门（AG）过滤通过跳过连接传播的特征。AGs的示意图如图2所示。AGs中的特征选择性是通过使用在较粗尺度中提取的上下文信息（门控）来实现的。

![image-20231107111740884](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231107111740884.png)

图2：拟议的附加注意门（AG）示意图。输入特征（xl）与AG中计算的注意力系数（α）成比例。通过分析选通信号（g）提供的激活和上下文信息来选择空间区域，选通信号是从更大的比例中收集的。注意力系数的网格重采样是使用三线性插值来完成的。

- 整体框架

```python
def forward(self,x):
        # encoding path
        x1 = self.Conv1(x) #1*3*512*512 ->conv(3,64)->conv(64,64)-> 1*64*512*512

        x2 = self.Maxpool(x1) #1*64*512*512 -> 1*64*256*256
        x2 = self.Conv2(x2)  #1*64*256*256 ->conv(64,128)->conv(128,128)-> 1*128*256*256
        x3 = self.Maxpool(x2) #1*128*256*256 -> 1*128*128*128
        x3 = self.Conv3(x3) #1*128*128*128 ->conv(128,256)->conv(256,256)->  1*256*128*128

        x4 = self.Maxpool(x3)#1*256*128*128 -> 1*256*64*64
        x4 = self.Conv4(x4) #1*256*64*64 ->conv(256,512)->conv(512,512)-> 1*512*64*64

        x5 = self.Maxpool(x4)#1*512*64*64 -> 1*512*32*32
        x5 = self.Conv5(x5) #1*512*32*32->conv(512,1024)->conv(1024,1024)-> 1*1024*32*32

        # decoding + concat path
        d5 = self.Up5(x5)  #1*1024*32*32 ->Upsample-> 1*1024*64*64 -> conv(1024,512) ->1*512*64*64
        x4 = self.Att5(g=d5,x=x4) #2(1*512*64*64) -> 1*1*64*64 ->1*512*64*64
        d5 = torch.cat((x4,d5),dim=1) #1*1024*64*64       
        d5 = self.Up_conv5(d5)  #1*1024*64*64 ->conv(1024,512)->conv(512,512)-> 1*512*64*64
        
        d4 = self.Up4(d5)
        x3 = self.Att4(g=d4,x=x3)
        d4 = torch.cat((x3,d4),dim=1)
        d4 = self.Up_conv4(d4)

        d3 = self.Up3(d4)
        x2 = self.Att3(g=d3,x=x2)
        d3 = torch.cat((x2,d3),dim=1)
        d3 = self.Up_conv3(d3)

        d2 = self.Up2(d3)
        x1 = self.Att2(g=d2,x=x1)
        d2 = torch.cat((x1,d2),dim=1)
        d2 = self.Up_conv2(d2)

        d1 = self.Conv_1x1(d2)

        return d1
```

- 上面的代码时forward的整体框架，unet的框架就不多做介绍，直接看attention的实现。对于一张输入为1x3x512x512（1是batchsize，3是通道）的2D图,执行到x5的时候（经过五次下采样）已经是最小的feature map了（1x1024x32x32）。对其进行上采样得到d5（1x512x64x64）。对d5和x4执行Att5(g=d5,x=x4)

```python
self.Att5 = Attention_block(F_g=512,F_l=512,F_int=256)

class Attention_block(nn.Module):
    def __init__(self,F_g,F_l,F_int):
        super(Attention_block,self).__init__()
        self.W_g = nn.Sequential(
            nn.Conv2d(F_g, F_int, kernel_size=1,stride=1,padding=0,bias=True),
            nn.BatchNorm2d(F_int)
            )
        
        self.W_x = nn.Sequential(
            nn.Conv2d(F_l, F_int, kernel_size=1,stride=1,padding=0,bias=True),
            nn.BatchNorm2d(F_int)
        )

        self.psi = nn.Sequential(
            nn.Conv2d(F_int, 1, kernel_size=1,stride=1,padding=0,bias=True),
            nn.BatchNorm2d(1),
            nn.Sigmoid()
        )
        
        self.relu = nn.ReLU(inplace=True)
        
    def forward(self,g,x):
        g1 = self.W_g(g) #1x512x64x64->conv(512，256)/B.N.->1x256x64x64
        x1 = self.W_x(x) #1x512x64x64->conv(512，256)/B.N.->1x256x64x64
        psi = self.relu(g1+x1)#1x256x64x64di
        psi = self.psi(psi)#得到权重矩阵  1x256x64x64 -> 1x1x64x64 ->sigmoid 结果到（0，1）

        return x*psi #与low-level feature相乘，将权重矩阵赋值进去
```

