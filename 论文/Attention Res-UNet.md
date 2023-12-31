# 1. 文章

## 主要贡献：

- 网络架构由一个新的引导的解码器组成，监督解码器的学习过程，改进特征
- 所提出的加权引导损失提高了解码器每层的预测能力，从而提高了最终输出层的预测精度
- 混合网络架构，通过Res-UNet架构的主干向注意力门灌输，该架构侧重于激活相关区域

## 相关工作：

- 引入注意力门是为了抑制不相关的工作
- 促进了模型中与肿瘤分割相关的空间区域参数的更新
- 如下方图所示，注意力门接收两个输入——一个来自包含该层中所有上下文和空间信息的相应编码器，另一个来自其下方解码器层的选通信号

![](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231109155933253-20231109160014348.png)

## 具体实现方法

- 讨论我们提出的带有引导解码器的注意力资源UNet（ARU-GD）模型的网络架构细节，以及训练过程中使用的损失函数，讨论具有引导解码的注意力资源UNet的数据集、预处理步骤和实现细节

### 有引导的解码器

- 用单独的损失函数明确训练每个解码器层，从而预测每一层的输出
- 此外，在单个解码器层的预测以及将每个中间层的加权损失转移到最终层有助于提高模型的分段精度
- D1，D2,D3,D4层的输出还原为输入的240*240，并且每一层都会增加一个输出的权重，越底层的权重越低，越往上的权重越高。
- 如图所示：

![image-20231109160718771](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231109160718771.png)



### 注意力门

- 注意力机制专注于图像的特定区域，而忽略其他区域
- 注意力门接收两个输入——一个来自包含该层中所有上下文和空间信息的相应编码器，另一个来自其下方解码器层的选通信号
- 如图所示：

![image-20231109161527873](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231109161527873.png)

### Encoder Path

- 编码器路径由四个层组成，每个层包含一个残差块，然后是一个2×2Max池化层。我们的模型采用240×240×4大小的输入，其中图像分辨率为240×240，并且我们有四个通道对应于MRI数据集的四种模态（Flair、T1、对比度增强T1和T2）。残差块由两个3×3的扩张卷积层组成，每个层后面都有一个批量归一化（BN）层和Leaky整流线性单元（Leaky ReLU）激活函数。第一残差块具有64个大小为240×240的特征图。在下采样过程中，每个层的特征图数量将加倍，特征图大小将减半。因此，第四层的残差块具有512个大小为30×30的特征图。第一和第二编码器层中的卷积块具有1的膨胀率，而第三和第四编码器层分别具有2和4的膨胀率
- 如图所示：

![image-20231109162814356](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231109162814356.png)

### bottleneck

- 瓶颈层连接所提出模型的编码器和解码器。它包含一个具有1024个大小为15×15的特征图的残差块。残差块中的卷积层具有8的膨胀率。瓶颈层的输出沿两条不同的路径移动。



### Decoder Path

- 解码器路径也由四个层组成，每个层都有一个残差块，前面有一个2×2卷积转置层。每个解码器层使用跳过连接和注意门与相应的编码器层链接。上采样是使用2×2卷积转置完成的。来自注意力门的输出与来自前一解码器层的上采样输出连接。拼接之后，输出被传递到剩余块。使用与编码器路径中的残差块相同的残差块。在每个解码器层中，特征图的数量减半，而它们的大小加倍。每个层的残差块的输出被上采样到240×240的大小，并通过分类层和softmax激活函数。分类层用于将特征图的数量减少到N个通道（N=4）。softmax激活将特征图转换为概率。因此，模型最终输出中的每个通道都成为特定类别的概率图。然后将这些输出与地面实况进行比较，并计算损失。只有最后一个解码器层D4的输出被认为是模型的主要输出。
- 如图所示：

![image-20231109162814356](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231109162814356.png)

### Loss Function

#### Weighted dice loss

#### Weighted cross-entropy loss 

#### Combined Loss

#### Guided Loss



