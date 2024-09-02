---
title: 从源码编译 Python
description: 从源码编译 Python 3.7
slug: build-python-from-source
date: 2024-06-03 08:54:00+0800
categories:
    - build
tags:
    - python
---
## 概述
本科时期使用 Windows 系统，Python 都是预先编译好的程序，可以直接安装而无需自己编译。研一刚入学，实验室老师让我加入了一个深度强化学习相关的项目，首先就需要在 Ubuntu 上配置环境。实验代码使用的 tensorflow 版本较低，在 [pypi](https://pypi.org/project/tensorflow/1.15.0/#files) 上查找 tensorflow1.15.0 的 wheel 包可用的 Python 版本为 3.7，而 Ubuntu 的 APT 包管理器中，已经没有提供预编译的 Python3.7 的包，因此需要从源代码编译安装 Python，下面记录了整个编译安装流程以及其中遇到的问题，可以直接拉到最后给出了正确的流程。
## 编译思路及遇到的问题
- 下载源码
- 安装编译依赖
- 开始编译
- 用编译完成的 Python 跑实验代码
- 碰到问题
- 回到第 1 步
### APT换源
在下载源码前，我先进行了 APT 换源。编辑 /etc/apt/sources.list 文件，修改里面的地址。这里我使用的是清华源。
```sh
sudo vi /etc/apt/sources.list
```
在 arm64 下，使用 ubuntu-ports

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

但是在 amd64 中，使用 ubuntu

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
更改完成后，更新APT的包列表缓存来应用新的软件源。
```sh
sudo apt update
sudo apt install wget
```

### 将源码直接下载到服务器中
先创建一个 Python3.7 的文件夹，从 Python 官网中下载源码并解压。
```sh
mkdir python3.7
cd python3.7/
wget https://www.python.org/ftp/python/3.7.17/Python-3.7.17.tgz
tar -xzvf Python-3.7.17.tgz
cd Python-3.7.17
```

### 安装编译依赖
CPython 是 Python 的主要实现，也是最广泛使用的 Python 解释器。我们指导 C 语言中就是用 make 来编译程序， 在这也是如此。 因为一开始编译不知道要安装哪些依赖，所以先安装了 build-essential 包，其中就包含了 gcc 编译器。
```sh
sudo apt-get install -y make build-essential
```
### 开始编译
运行 `./configure` 脚本来生成 Makefile 文件，然后使用 make 命令编译 Python。`-j$(nproc)` 选项允许make使用 nproc 个并发任务来加速编译过程。可以根据 CPU 核心数来调整这个值。编译完成后，执行 `./python --version` 命令来查看是否成功，成功的话会显示正确的 Python 版本号。
```sh
./configure
make -j$(nproc)
./python --version
```

### 创建虚拟环境
这里我创建了一个虚拟环境，用来安装实验代码需要用到的包。
```sh
../python3.7/Python-3.7.17/python -m venv venv
```
其中 `../python3.7/Python-3.7.17/` 是我已经编译的 Python 目录，而 python 是 Python 解释器可执行文件，也就是说，我指定在 `../python3.7/Python-3.7.17/python` 这个直接路径下创建虚拟环境，如果没有指定的话，会在系统路径中的默认 Python 解释器中创建虚拟环境。`-m venv` 是在告诉 Python 解释器使用模块模式（`-m`）来运行 `venv` 模块,最后的 `venv` 是要创建的虚拟环境的名称。

但是运行该行命令后产生报错，得到关键信息如下。
```
Error: Command '['/home/justt/rl-stock-trading-1103/venv/bin/python', '-Im', 'ensurepip', '--upgrade', '--default-pip']' returned non-zero exit status 1.
```
可以看到是因为没有安装 pip，因此在系统中安装 pip，再一次重复前面的步骤。虚拟环境创建完成后，需要 `source venv/bin/activate` 激活它来开始使用。

```sh
sudo apt install python3-pip
cd ~/python3.7/Python-3.7.17 
make -j$(nproc)
cd ~/rl-stock-trading-1103
../python3.7/Python-3.7.17/python -m venv venv
source venv/bin/activate
```
到这里成功创建并激活了虚拟环境。注意，此时查看 Python 版本是执行 `python --version` 命令，而不是 2.4 节中提到的 `./python --version` 命令。这里没有写前面的路径，是因为虚拟环境激活后，任何我们运行的 Python 相关命令都会使用虚拟环境中的包和解释器，而不是系统默认的那些。执行 `ls -al venv/bin` 命令来进行验证。
```
lrwxrwxrwx 1 justt justt   67 Jun  2 16:32 python -> /home/justt/rl-stock-trading-1103/../python3.7/Python-3.7.17/python
```
可以看到 python 指向了虚拟环境中 Python 解释器。
### 安装环境
```
pip install -r requirements.txt
```
尝试安装实验代码中的环境，得到了以下报错信息（截取了关键部分）。
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

发现无法安装，报错原因中有 `however the ssl module in Python is not available`，查询资料发现是因为下载源 `https://pypi.org/simple` 是HTTPS是HTTP使用TLS/SSL加密的，所以要再安装 `libssl-dev` 依赖。注意每次安装完新的依赖后，都需要重新进行编译。

```sh
sudo apt install libssl-dev
cd ~/python3.7/Python-3.7.17 
./configure #检测刚刚装的libssl，添加进python编译选项中
make -j$(nproc)
cd -
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
再此执行上述步骤，仍有报错。
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
报错原因中有 `Can not execute ``setup.py`` since setuptools is not available in the build environment.`。参考 [解决方案](https://github.com/romesco/hydra-lightning/issues/19#issuecomment-1116638713)，发现是缺少 `libffi-dev` 依赖。
```
sudo apt install libffi-dev
cd ~/python3.7/Python-3.7.17 
./configure 
make -j$(nproc)
cd -
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
再此执行上述步骤，仍有报错。
```
ERROR: Failed building wheel for mpi4py
Successfully built gym
Failed to build mpi4py
ERROR: Could not build wheels for mpi4py, which is required to install pyproject.toml-based projects
```
报错原因中有 `Could not build wheels for mpi4py, which is required to install pyproject.toml-based projects`，查询资料发现是缺少 `libopenmpi-dev` 依赖。

```sh
sudo apt-get install libopenmpi-dev
cd ~/python3.7/Python-3.7.17 
./configure 
make -j$(nproc)
cd -
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
再此执行上述步骤，目前总算没有报错了。

最后来检查是否安装成功。
```sh
echo $?
```
其中 `$?` 是一个特殊的 Shell 变量，存储上一个命令的退出状态码。惯例上 `：0` 表示命令成功执行。

但是仍存在报错，信息如下。
```
    import bz2
  File "/home/justt/python3.7/Python-3.7.17/Lib/bz2.py", line 19, in <module>
    from _bz2 import BZ2Compressor, BZ2Decompressor
ModuleNotFoundError: No module named '_bz2'
```
可以看到是缺少 `libbz2-dev` 依赖。
```sh
sudo apt-get install libbz2-dev
cd ~/python3.7/Python-3.7.17 
./configure 
make -j$(nproc)
cd -
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
重复以上步骤，继续得到报错。
```
/home/justt/rl-stock-trading-1103/venv/lib/python3.7/site-packages/pandas/compat/__init__.py:124: UserWarning: Could not import the lzma module. Your installed Python is incomplete. Attempting to use lzma compression will result in a RuntimeError.
```
可以看到是缺少 `liblzma-dev` 依赖。
```sh
sudo apt-get install liblzma-dev
cd ~/python3.7/Python-3.7.17 
./configure 
make -j$(nproc)
cd -
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
最终，成功安装了我的实验所需环境。因为一开始我并不知道需要安装哪些依赖，所以通过不断地尝试和对报错信息的分析，来一步步补全所需的依赖，这些依赖当然可以一起安装，就不需要不断重复编译过程，我在最后总结了正确的流程。

## 正确流程
### apt换源
```sh
sudo vi /etc/apt/sources.list
```
在 arm64 下，使用 ubuntu-ports

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

但是在 amd64 中，使用 ubuntu

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
cd
# CPython
wget https://www.python.org/ftp/python/3.7.17/Python-3.7.17.tgz
tar -xzvf Python-3.7.17.tgz

PYTHON_SRC=$HOME/Python-3.7.17
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
