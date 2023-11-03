# 1. Mac配置Maven

## 1.1 下载maven

地址：[Maven – Download Apache Maven](https://maven.apache.org/download.cgi)

![image-20230818234720750](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230818234720750.png)

## 1.2 配置环境变量

### 1. 打开终端输入

```
open ~/.zshrc
```

### 2. 配置环境

此处是我自己的maven文件夹的位置

```
export M2_HOME=/Users/panjie/Documents/devTools/apache-maven-3.9.4
export PATH=$PATH:$M2_HOME/bin
```

- tips：打开访达然后点击显示可以显示文件夹的位置

### 3. 在终端中生效

```
source ~/.zshrc
```

### 4. 检验

```
mvn -v
```

## 1.3 配置阿里云镜像

打开/Users/panjie/Documents/devTools/apache-maven-3.9.4/conf/settings.xml

![image-20230818235510151](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230818235510151.png)

    <!-- 配置阿里云镜像仓库 -->
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>

## 1.4 配置仓库

1. 在maven文件夹中创建repository

2. 修改settings.xml

   ![image-20230818235635159](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230818235635159.png)

```
<localRepository>/Users/panjie/Documents/devTools/apache-maven-3.9.4/repository</localRepository>
```

# 2. Mac配置Mysql

## 2.1 下载mysql

[MySQL :: Download MySQL Community Server](https://dev.mysql.com/downloads/mysql/)

![image-20230819210858148](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230819210858148.png)

## 2.2 配置环境变量

### 1. 进入/.zshrc

```
open ~/.zshrc
```

### 2. 配置mysql

```
#mysql
export MYSQL_HOME=/usr/local/mysql
export PATH=$MYSQL_HOME/bin:$PATH
```

### 3. 生效

```
source ~/.zshrc
```

## 2.3 验证

```
mysql --version
```



# 3. Mac配置Anaconda

## 3.1 安装Anaconda

打开网站：[Free Download | Anaconda](https://www.anaconda.com/download/)

无脑下一步就行

## 3.2 安装检查

```
conda --version
```

```
python --version
```

```
conda info --envs
```



## 3.3 Pytorch安装

### 1. 创建一个新的虚拟环境

退出某一环境命令如下：

```
conda deactivate
```

虚拟环境创建命令如下：

```
conda create -n pytorch
```

截图如下：

![image-20230913171253037](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230913171253037.png)

- 可以看到输入conda deactivate后，退出base环境，下一行前面已经没有"(base)"字样
- 创建新的环境后，会告诉我们如何进入和退出这个环境
- 如果想每次打开终端，不自动进入任何环境（现在打开终端自动进入base环境）可以设置如下操作：

```
conda config --set auto_activate_base false
```

### 2. 配置pytorch虚拟环境

- 首先终端进入pytorch环境：

```
conda activate pytorch
```

![image-20230913171717709](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230913171717709.png)

- 安装python 3.8版本

```
conda install python==3.8
```

- 安装pytorch

打开官网：[Start Locally | PyTorch](https://pytorch.org/get-started/locally/)

![image-20230913172554656](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230913172554656.png)

### 3. 测试一下

在pytorch环境下打开python，import torch没有出错：

![image-20230913172842075](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230913172842075.png)

### 4. 其他命令

如果想删除某个虚拟环境，则在终端中输入：

```
conda remove -n env_name --all # env_name用要删除的虚拟环境的名字替换即可
```

## 3.4 在PyCharm中配置PyTorch虚拟环境

![image-20230913175610647](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230913175610647.png)

![image-20230913175633214](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230913175633214.png)

## 遇到的问题：

**新版的pycharm找不到python可执行文件**

解决：

将bin目录下的conda复制到虚拟环境中的bin文件
