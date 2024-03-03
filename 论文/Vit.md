参考链接：https://www.bilibili.com/video/BV1AL411W7dT/?spm_id_from=333.788&vd_source=48f70b23d5e6bb70ac527b93e20bd72e

# Abstract

- Transformer在自然语言处理上是优秀的，在视觉上的帮助是有限的
- CNN并不是必要的，一个完全用transformer的模型也可以在图像分类上做得很好

# Introduction

- 计算机视觉领域，卷积结构还是占据主导地位的
- 把图片切成一个个的patch然后拉长成一个序列做为transformer的输出

# Method

- 模型的整体结构

![image-20231214134430333](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231214134430333.png)

- 为了处理2D图像，将图像x（H * W * C ）重塑成 X （N *（P * P * C））,其中HW是原来的图像的分辨率，(*P*,*P*) 是每个图像块的分辨率，而 N*=*H**W*/*P*2 是块的结果数量
- 会在序列块的前面加一个可学习的参数，表示分类。
- Position Embedding被添加到嵌入块中以保留位置信息



# Code

- class PatchEmbed

![image-20231214140206269](/Users/panjie/Library/Application Support/typora-user-images/image-20231214140206269.png)
