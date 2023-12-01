# 1. 指针

## 1.1 指针的简单应用

![image-20231130172211692](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231130172211692.png)

in t a=10

10存放在320893这个地址

现在定义一个指向a的指针，这个指针存放的是a的地址



```c++
int* p1  //p1是一个指向int类型数据的指针
long* p2  //p2是一个指向long类型的指针
```

![image-20231130174842979](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231130174842979.png)

```c++
// p指向的是a的地址，a的地址是64位的，所以需要8个字节
// 第一行存放的就是88，第二行存放的是F8 
int* p = &a
```

## 1.2 指针的指针

![image-20231130184134388](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20231130184134388.png)