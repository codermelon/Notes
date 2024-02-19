# 1. 卸载pytorch

- 卸载依赖库

```
sudo apt-get remove python3-pip python3-venv python3-dev
```

- 进入conda删除PyTorch

```
pip uninstall torch torchvision
```

- 清理残留文件

```
sudo rm -rf ~/.cache/pip ~/.cache/torch
```



# 2. 关于虚拟环境

- 查看当前系统的虚拟环境有哪些

```
conda info --env
```

- 激活虚拟环境

```
conda activate python36（环境名）
```

- 退出虚拟环境

```
conda deactivate
```





