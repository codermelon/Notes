# 1. 文章总体

可以参考：https://zhuanlan.zhihu.com/p/118543343

##  Abstract

- Unet++重新设计的跳跃的路径可以缩小特征图在encoder-docoder阶段上语义的差距

## 1.1 Introduction

- 这些encoder-decoder的结构都有一个相似的点：这些跳跃结构都是把decoder阶段深层次的语义分割图与encoder阶段低层次的分割图结合起来
- 医学图像需要更高的准确度
- 在自然语言分割中也许一个mask不是那么重要，但是在医学图像中一个微小的错误也会导致临床上糟糕的体验
- 当来自编码器网络的高分辨率特征图在与来自解码器网络的相应语义丰富的特征图融合之前逐渐丰富时，该模型可以更有效地捕获前景对象的细粒度细节。

- 与Unet中普通的跳跃不同的是，Unet++能直接将encoder阶段的高分辨率图片送到decoder阶段，促进语义的融合

## 1.2 Related Work

- 之前的研究都是倾向于融合encoder和decoder阶段的特征图，但是Unet++这样处理会降低准确度

## 1.3 UNet++

### 1. Re-designed skip pathways

- 每一层融合的结果都是前一层卷积之后与下采样再上采样进行融合

### 2. Deep supervision

- 使用深度监督使模型以两个模式进行：
  1. 精准模式，所有来自分割分支的输出被平均
  2. 快速模式，其中仅从一个分割分支中选择最终分割图，对该分割分支的选择确定模型修剪和速度增益的程度
- 总结：Unet++和原来的UNet有三个不同的地方：
  - 在跳跃路径上有卷积层，缩小了编码和解码阶段语义上的差距
  - 在跳跃路径上有密集的连接，改善了梯度流动
  - 具有深度监督，可以进行模型修剪

