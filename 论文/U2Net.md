# 1. 网络结构

- 类似于UNet结构，但是每一个Encoder和Decoder部分都经过很多个卷积 bn relu

![](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231019140348198.png)

![image-20231019140534139](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231019140534139.png)

- RSU结构中先经过一个卷积，然后是5个encoder，5个decoder，采用线性插值的方法

  - En_1用的RSU-7 下采样32倍
  - En_2用的RSU-6下采样16倍
  - En_3用的RSU-5
  - En_4用的RSU-4

- 注意En_5 En_6 De_5 用的是RSU-4F

  - 为什么用RSU-4F？此时特征图已经很小了，没有必要继续下采样，使用膨胀卷积

  ![image-20231109115615343](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231109115615343.png)



# 2. 结合代码

- 因为经常使用到ConvBNReLU，写成一个类，即上图中对应的绿色部分

```python
class ConvBNReLU(nn.Module):
    def __init__(self, in_ch: int, out_ch: int, kernel_size: int = 3, dilation: int = 1):
        super().__init__()

        padding = kernel_size // 2 if dilation == 1 else dilation
        self.conv = nn.Conv2d(in_ch, out_ch, kernel_size, padding=padding, dilation=dilation, bias=False)
        self.bn = nn.BatchNorm2d(out_ch)
        self.relu = nn.ReLU(inplace=True)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.relu(self.bn(self.conv(x)))
```

- 下采样采用最大池化来实现,继承ConvBNReLU中的参数

```python
class DownConvBNReLU(ConvBNReLU):
    def __init__(self, in_ch: int, out_ch: int, kernel_size: int = 3, dilation: int = 1, flag: bool = True):
        super().__init__(in_ch, out_ch, kernel_size, dilation)
        self.down_flag = flag

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if self.down_flag:
            x = F.max_pool2d(x, kernel_size=2, stride=2, ceil_mode=True)

        return self.relu(self.bn(self.conv(x)))

```

- 上采样需要拼接下采样的部分,用线性插值的方法

```python
class UpConvBNReLU(ConvBNReLU):
    def __init__(self, in_ch: int, out_ch: int, kernel_size: int = 3, dilation: int = 1, flag: bool = True):
        super().__init__(in_ch, out_ch, kernel_size, dilation)
        self.up_flag = flag

    def forward(self, x1: torch.Tensor, x2: torch.Tensor) -> torch.Tensor:
        if self.up_flag:
            x1 = F.interpolate(x1, size=x2.shape[2:], mode='bilinear', align_corners=False)
        return self.relu(self.bn(self.conv(torch.cat([x1, x2], dim=1))))

```

- RSU模块需要定义一个height，因为每层对应的层数不一样

```python
class RSU(nn.Module):
    def __init__(self, height: int, in_ch: int, mid_ch: int, out_ch: int):
        super().__init__()

        assert height >= 2
        self.conv_in = ConvBNReLU(in_ch, out_ch) # 对应第一个卷积
				
        # encoder中第二个卷积并没有下采样所以flag为false
        encode_list = [DownConvBNReLU(out_ch, mid_ch, flag=False)] 
        # decoeder中第一个卷积并没有上采样所以flag为false
        decode_list = [UpConvBNReLU(mid_ch * 2, mid_ch, flag=False)]
        # 将对应上采样下采样的操作放入一个model中
        for i in range(height - 2):
            encode_list.append(DownConvBNReLU(mid_ch, mid_ch))
            # 注意上采样的最后一个卷积的输出
            decode_list.append(UpConvBNReLU(mid_ch * 2, mid_ch if i < height - 3 else out_ch))
				# 最下面的的卷积操作mid_ch,mid_ch
        encode_list.append(ConvBNReLU(mid_ch, mid_ch, dilation=2))
        self.encode_modules = nn.ModuleList(encode_list) 
        self.decode_modules = nn.ModuleList(decode_list)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x_in = self.conv_in(x) # 第一个卷积

        x = x_in # 因为最后要融合需要保存第一个操作之后的结果
        encode_outputs = [] #定义一个列表保存每次操作之后的数据方便后续进行上采样的操作
        for m in self.encode_modules:
            x = m(x)
            encode_outputs.append(x)

        x = encode_outputs.pop() # 将最后一个下采样的数据弹出进行上采样 对应途中最下方卷积操作之后的数据
        for m in self.decode_modules:
            x2 = encode_outputs.pop() # 依次弹出encoder部分的
            x = m(x, x2)

        return x + x_in
```

![image-20231019141646469](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231019141646469.png)

- 在en-5，en-6，de-5中采用的是膨胀卷积

![image-20231019143625107](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231019143625107.png)

```python
class RSU4F(nn.Module):
    def __init__(self, in_ch: int, mid_ch: int, out_ch: int):
        super().__init__()
        self.conv_in = ConvBNReLU(in_ch, out_ch)
        self.encode_modules = nn.ModuleList([ConvBNReLU(out_ch, mid_ch),
                                             ConvBNReLU(mid_ch, mid_ch, dilation=2),
                                             ConvBNReLU(mid_ch, mid_ch, dilation=4),
                                             ConvBNReLU(mid_ch, mid_ch, dilation=8)])

        self.decode_modules = nn.ModuleList([ConvBNReLU(mid_ch * 2, mid_ch, dilation=4),
                                             ConvBNReLU(mid_ch * 2, mid_ch, dilation=2),
                                             ConvBNReLU(mid_ch * 2, out_ch)])

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x_in = self.conv_in(x)

        x = x_in
        encode_outputs = []
        for m in self.encode_modules:
            x = m(x)
            encode_outputs.append(x)

        x = encode_outputs.pop()
        for m in self.decode_modules:
            x2 = encode_outputs.pop()
            x = m(torch.cat([x, x2], dim=1))

        return x + x_in
```

# 3. 膨胀卷积

- 增大感受野
- 保持原输入特征图W H