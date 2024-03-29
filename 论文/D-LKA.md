# Abstract

- Transformer模型在医学图像分割中的局限性，这些局限性主要是由于它们的高计算需求和处理3D体积图像数据的挑战。
- 作者提出了一种名为D-LKA Net的网络，利用大的卷积核和可变形卷积有效地处理体积上下文并适应多样的数据模式。

# Introduction

- D-LKA Net，它采用了可变形大核心注意力（D-LKA Attention）机制。这种机制能够有效地处理大范围上下文信息，并适应不同的数据模式，从而在医学图像分割任务上实现了优越的性能。
- 作者还讨论了其方法与现有技术的比较，展示了D-LKA Net在多个公开可用的分割数据集上取得的最先进结果，证明了其作为医学图像分割强大和可靠选择的潜力。

# Method

