# 1. Self-Attention

- 参考博文：https://blog.csdn.net/qq_37541097/article/details/117691873

为了方便理解，假设输入的长度为2，即两个节点x1,x2,然后通过Input Embedding也就是图中的f(x)映射到a1,a2,然后通过三个变换矩阵Wq,Wk,Wv(这三个参数是可训练的)，得到对应的q,k,v

![image-20231018092317842](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018092317842.png)

输入的x映射成a1，然后和矩阵w乘起来得到q1,k1,v1

- 注意：此处的wq,wk,wv是共享的，即a1,a2都是共用这个矩阵的，并且这是一个可训练参数

- q代表query，后续会去和每一个k进行匹配

- k代表key，后续会被每个q匹配
- v代表从a中提取得到的信息
- 后续q和k匹配的过程可以理解成计算两者的相关性，相关性越大对应v的权重也就越大

![image-20231018093128632](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018093128632.png)

q1和k1点乘得到a1,1 

q1和k2点乘得到a1,2

a1,1和a1,2通过soft— max得到a1,1 a1,2

右边同理q2和k1,q2和k2

![image-20231018093453489](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018093453489.png)

a1,1和v1乘起来 a1,2和v2乘起来

## 2. Multi-Head Attention

![image-20231018094405667](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018094405667.png)

q1得到的是（1101）拆分成q11,q12

![image-20231018094546658](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018094546658.png)

将所有第二个数字是1的放到一起组成head1，将所有第二个数字是2的放到一起组成head2

![image-20231018094615635](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018094615635.png)

对于每一个head执行self—Attention操作得到b

![image-20231018095010172](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018095010172.png)

将b中第一个数字是1的放到一起，第一个数字是2的放到一起

![image-20231018095106816](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231018095106816.png)

然后拼接后的结果按照W0（可学习的参数进行融合）