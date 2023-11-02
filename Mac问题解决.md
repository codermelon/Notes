# 1. 下载文件时显示文件已损坏

## 1.1 方法一

1. 打开终端
2. 输入命令

```
sudo spctl --master-disable
```

3. 输入电脑密码

## 1.2 方法二

打开终端输入：

```
sudo xattr -d com.apple.quarantine /Applications/xxxx.app
```

此处换成自己的app路径