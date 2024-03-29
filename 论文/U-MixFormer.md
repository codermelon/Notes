# Abstract

- 在Transformer取得长足进步的同时，基于CNN的U-Net也取得了重大进展，尤其是在高分辨率医学成像和遥感方面
- 我们提出了U-MixFormer
  - 我们的方法与以前的转换器方法不同，它利用编码器和解码器阶段之间的横向连接作为注意力模块的特征查询，而不是传统的依赖跳过连接
  - 此外，我们创新地混合了来自不同编码器和解码器阶段的分层特征映射，形成了键和值的统一表示，从而产生了我们独特的混合注意模块。

# Introduction

- 我们的贡献总结如下
  - 我们提出了一种新的强大的转换-解码器架构，该架构由U-Net驱动，用于高效的语义分割。利用U-Net在捕获和传播层次特征方面的熟练程度，我们的设计独特地使用了变压器编码器的横向连接作为查询特征。这种方法确保了高级语义和低级结构的和谐融合。
  - 为了提高我们的类似UNet的转换器架构的效率，我们混合和更新了多个编码器和解码器输出作为键和值的集成功能，从而产生了我们提出的混合注意机制。该方法不仅具有丰富的特征表示，而且还提高了上下文理解能力。
  - 兼容不同的编码器

# Related Work

1. **变换器在语义分割中的应用**：文中提到了如SETR、PVT、Swin Transformer、SegFormer等变换器架构，它们通过不同的策略改进了特征的提取和处理，以适应语义分割的需求。这些改进包括多尺度特征的生成、自注意力模块的效率提高，以及位置编码的处理方式等。
2. **解码器架构的创新**：介绍了如DETR、Segmenter、MaskFormer等使用变换器解码器的方法，并讨论了它们在处理多尺度编码器特征时面临的挑战。特别是，FeedFormer提出了直接使用编码器阶段特征作为特征查询的高效解码策略，但这种方法没有在解码阶段之间增量传播特征图，限制了性能的进一步提升。
3. **UNet架构的变革**：指出了在医学和遥感图像分割中，从基于CNN的UNet架构向基于变换器的架构转变的尝试。TransUNet和Swin-UNet等混合架构展示了将变换器技术整合进UNet架构的潜力，但同时也指出了现有方法在解码器设计和特征传播机制上的不足。

# Method

![image-20240225205651122](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20240225205651122.png)



![image-20240225210050150](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20240225210050150.png)

在“方法”部分，U-MixFormer的设计和工作机制被详细阐述，分为以下几个关键部分：

- **总体架构**：U-MixFormer是一个结合了UNet结构和变换器（Transformer）的语义分割模型，采用了多阶段的编码器-解码器架构。这个架构利用了变换器的强大上下文感知能力和UNet的高效特征传播机制。

- **编码器**：输入图像首先通过编码器处理，编码器分为多个阶段，每个阶段产生不同分辨率的特征图。这些特征图提供了图像的多尺度表示，有助于捕获从粗到细的细节信息。

- **混合注意力（Mix-Attention）机制**：U-MixFormer引入了混合注意力机制，这是其核心创新之一。在这个机制中，查询（Q）来自于编码器的侧向连接，而键（K）和值（V）是由来自不同阶段的编码器和解码器特征混合而成的。这种设计允许模型有效地融合不同层次的特征，并增强模型对上下文信息的理解。

  - Mix-Attention是U-MixFormer中的一个关键创新点，它是对传统的自注意力（self-attention）和交叉注意力（cross-attention）机制的扩展。在自注意力机制中，查询（Q）、键（K）和值（V）都来自于同一个特征集（Xqkv），即同一个编码器/解码器阶段。而在交叉注意力机制中，查询（Xq）和键值对（Xkv）分别来自不同的源。U-MixFormer的混合注意力机制（Mix-Attention）则采用了一种创新的方法，其中键值对（Xkv）是通过混合多个多尺度阶段的特征来获取的。

    这种混合特征（Xkv）来源于多个编码器和解码器阶段，允许查询在不同阶段之间找到匹配项，即不同程度的上下文粒度，从而促进了特征的精细化。这种机制通过有效管理这些阶段之间的依赖关系，有助于捕获上下文并细化边界，从而提升了模型对高级语义和低级结构的融合能力。

- **解码器**：解码器部分由多个阶段组成，每一阶段都利用混合注意力机制来细化特征。通过逐渐融合编码器的多尺度特征和之前解码器阶段的输出，解码器能够逐步恢复图像的空间细节，并生成高质量的语义分割图。

- **特征合成和最终预测**：解码器的输出特征经过上采样和融合，最后通过一个多层感知器（MLP）生成最终的语义分割图。这个过程保证了模型能够有效地结合高级语义信息和低级空间信息，从而实现精确的像素级预测。

- **与不同编码器的兼容性**：U-MixFormer展示了与不同类型编码器（包括基于变换器和CNN的编码器）的兼容性，证明了其设计的灵活性和通用性。

通过这些设计，U-MixFormer不仅提高了语义分割的效率和准确性，而且展示了在处理多尺度特征和复杂场景时的优越性能。