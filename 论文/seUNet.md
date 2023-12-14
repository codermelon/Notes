# Abstract

- 利用UNet做为特征提取器，从输入图像生成多个特征图，然后将这些特征图送入一个桥阶层中，再送入transformer
- 采用了像素级嵌入技术，不使用位置嵌入向量
- 该模型不仅保留了局部和全局上下文信息，而且能够捕获输入元素之间的长期依赖关系



# Introduction

- UNet由于缺乏捕捉图像中长距离依赖性和全局上下文信息的能力，这些架构通常会产生较差的性能，尤其是对于在纹理、形状和大小方面表现出显著差异的目标信息
  - 虽然Transformer擅长捕捉上下文信息，但是在医学领域需要掌握细粒度的细节
  - 所以将Transformer和CNN结合

- UNet充当特征提取器，从输入图像中导出多个特征图。然后，这些映射被输入到简略层，战略性地放置，以按顺序连接UNet和Transformer组件
  - 值得注意的是，我们的方法采用了一种像素级嵌入技术，其中嵌入了前哨嵌入向量，以提高模型的效率

# Related Work

- CNN-based Medical Image Segmentation
  - 这些方法都是以细胞神经网络为基础的，天生就错过了捕捉长期依赖关系和理解上下文的机会
- Transformer-based Medical Image Segmentation
- CNN-Transformer - based Medical Image Segmentation
  - 尽管Transformer可以在所有阶段对全局上下文信息进行建模，但是把图片处理为1D序列，可能会导致低分辨率特征，缺乏了精确的定位
  - 简单地采用直接上采样来实现全分辨率并不能有效地恢复这些信息，从而导致不精确的分割结果



# Method

- Encoder
  - 编码器的作用就是从网络中的输入图像提取信息，通过卷积层实现，然后通过最大池化层下采样。
  - 空间维度会减小，特征图的深度会增大

