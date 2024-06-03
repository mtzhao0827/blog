---
title: 从源码编译 Python
description: ZFS is a state-of-the-art filesystem, and Docker is a revolutionary tool. But combining them can be a terrible idea...
slug: docker-and-zfs-a-tough-pair
date: 2022-08-07 08:54:00+0800
categories:
    - filesystem
    - containerization
    - kernel
tags:
    - overlayfs
    - zfs
    - docker
---

### apt换源
```sh
sudo vi /etc/apt/sources.list
```
在arm64下，使用ubuntu-ports

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
```

但是在amd64中，使用ubuntu

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

```sh
sudo apt update
sudo apt install wget
```

### 将源码直接下载到服务器中

```sh
mkdir -p ~/python3.7
cd ~/python3.7/
# CPython
wget https://www.python.org/ftp/python/3.7.17/Python-3.7.17.tgz
tar -xzvf Python-3.7.17.tgz

PYTHON_SRC=$HOME/python3.7/Python-3.7.17
cd $PYTHON_SRC
```

### 安装编译依赖
```sh
sudo apt-get install -y build-essential python3-pip libssl-dev libffi-dev libopenmpi-dev libbz2-dev liblzma-dev
```
### 开始编译

```sh
./configure
make -j$(nproc)
./python --version
```

### 创建虚拟环境

```sh
PROJ_DIR=$HOME/rl-stock-trading-1103
cd $PROJ_DIR
$PYTHON_SRC/python -m venv venv
source venv/bin/activate
```

### 安装环境

```sh
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```



- 为什么要用Python3.7（在pypi上看tensorflowxx的wheel包可用的python版本）

https://pypi.org/project/tensorflow/1.15.0/#files

- 为什么要从源码编译Python3.7（包管理器没有）


- 怎么编译

  - 下载源码

  - 安装编译依赖

  - 开始编译

  - 用编译完成的Python跑量化交易项目

  - 碰到问题

  - 回到步骤1

### apt换源
```sh
sudo vi /etc/apt/sources.list
```
在arm64下，使用ubuntu-ports

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
```

但是在amd64中，使用ubuntu

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```



```sh
sudo apt update
sudo apt install wget
```

### 将源码直接下载到服务器中

```sh
mkdir python3.7
cd python3.7/
wget https://www.python.org/ftp/python/3.7.17/Python-3.7.17.tgz
tar -xzvf Python-3.7.17.tgz
cd Python-3.7.17
```

### 安装编译依赖
CPython 
```sh
sudo apt-get install -y make build-essential
```
### 开始编译

```sh
./configure
make -j$(nproc)
./python --version
```

### 创建虚拟环境

```sh
../python3.7/Python-3.7.17/python -m venv venv
```

解释为什么这么写`../python3.7/Python-3.7.17/python`，`-m venv`, `venv`

```
Error: Command '['/home/justt/rl-stock-trading-1103/venv/bin/python', '-Im', 'ensurepip', '--upgrade', '--default-pip']' returned non-zero exit status 1.
```

报错没有pip

在系统中安装pip

```sh
sudo apt install python3-pip
cd ~/python3.7/Python-3.7.17 
make -j$(nproc)
cd ~/rl-stock-trading-1103
../python3.7/Python-3.7.17/python -m venv venv
source venv/bin/activate
```

为什么现在 `python --version`可以不用前面一堆东西了？`PATH` , `ls -al venv/bin`

```
lrwxrwxrwx 1 justt justt   67 Jun  2 16:32 python -> /home/justt/rl-stock-trading-1103/../python3.7/Python-3.7.17/python
```

安装环境

```
pip install -r requirements.txt
```

```WARNING: pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/numpy/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/numpy/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/numpy/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/numpy/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/numpy/
Could not fetch URL https://pypi.org/simple/numpy/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/numpy/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.")) - skipping
ERROR: Could not find a version that satisfies the requirement numpy (from versions: none)
ERROR: No matching distribution found for numpy
WARNING: pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.")) - skipping
```

发现无法安装，报错原因中有`however the ssl module in Python is not available`，因为下载源`https://pypi.org/simple`是HTTPS是HTTP使用TLS/SSL加密的，所以要再安装相关的依赖。

```sh
sudo apt install libssl-dev
cd ~/python3.7/Python-3.7.17 
./configure #检测刚刚装的libssl，添加进python编译选项中
make -j$(nproc)
cd -
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

```

报错

```
Collecting sklearn
  Downloading https://pypi.tuna.tsinghua.edu.cn/packages/46/1c/395a83ee7b2d2ad7a05b453872053d41449564477c81dc356f720b16eac4/sklearn-0.0.post12.tar.gz (2.6 kB)
  Preparing metadata (setup.py) ... error
  error: subprocess-exited-with-error
  
  × python setup.py egg_info did not run successfully.
  │ exit code: 1
  ╰─> [1 lines of output]
      ERROR: Can not execute `setup.py` since setuptools is not available in the build environment.
      [end of output]
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
error: metadata-generation-failed
```

https://github.com/romesco/hydra-lightning/issues/19#issuecomment-1116638713

缺少`libffi-dev`

```
sudo apt install libffi-dev
cd ~/python3.7/Python-3.7.17 
./configure 
make -j$(nproc)
cd -
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```



报错

```
ERROR: Failed building wheel for mpi4py
Successfully built gym
Failed to build mpi4py
ERROR: Could not build wheels for mpi4py, which is required to install pyproject.toml-based projects
```

​	

```sh
sudo apt-get install libopenmpi-dev
```



报错

```
 pip install h5py<3.0.0 -i https://pypi.tuna.tsinghua.edu.cn/simple
-bash: 3.0.0: No such file or directory
```

指定确切的版本号后可以安装成功

检查是否安装成功

```sh
echo $?
```



报错

```
    import bz2
  File "/home/justt/python3.7/Python-3.7.17/Lib/bz2.py", line 19, in <module>
    from _bz2 import BZ2Compressor, BZ2Decompressor
ModuleNotFoundError: No module named '_bz2'
```

```sh
sudo apt-get install libbz2-dev
cd ~/python3.7/Python-3.7.17 
./configure 
make -j$(nproc)
```



报错

```
/home/justt/rl-stock-trading-1103/venv/lib/python3.7/site-packages/pandas/compat/__init__.py:124: UserWarning: Could not import the lzma module. Your installed Python is incomplete. Attempting to use lzma compression will result in a RuntimeError.
```

```sh
sudo apt-get install liblzma-dev
cd ~/python3.7/Python-3.7.17 
./configure 
make -j$(nproc)
```

