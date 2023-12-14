源码：https://github.com/Beckschen/TransUNet

# Abstract

Unet通常在long-range dependency中表现的不太好，因此专门对于sequence-to-sequence的transformer并且自带注意力机制的这种模型成为了一种选择。一方面，Transformer将来自卷积神经网络（CNN）特征图的标记化图像块编码为输入序列，用于提取全局上下文。另一方面，解码器对编码特征进行上采样，然后将其与高分辨率CNN特征图相结合，以实现精确定位。

# Introduction

- 基于CNN的方法非常好，但是具有局限性
  - 在长序列建模的能力弱
  - 对于在质地、形状和尺寸方面表现出较大患者间变化的目标结构的性能弱
- Transformer在建模全局上下文方面很有能力
- 直接使用Transformer对标记过的图像进行编码，然后上采样将这些特征输出并不能得到一个好的效果
  - 因为Transformer将输入视为1D序列，缺乏了详细的定位信息，缺乏了低分辨率特征，CNN可以很好的解决

# Method

## Transformer as Encoder

1. 我们首先将输入的x变成一个2D的序列，并且切割成一个N个大小为P*P的图片，N=H*W/P^2,N是patches
1. 不太理解，可能是类似于VIT进行编码的东西

![image-20231108110540910](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231108110540910.png)



## TransUnet

- 简单的方法就是将编码特征上采样到全分辨率，reshape操作和使用1*1的卷积，但是这样会丢失很多低级别的细节
- CNN-Transformer Hybrid as Encoder
  - 没有使用纯Tranformer作为编码器，首先使用CNN作为特征提取器，生成输入的特征图，将块嵌入应用于CNN提取的特征图而不是原始图像
  - 采用及联的上采样操作达到全分辨率

