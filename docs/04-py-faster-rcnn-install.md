# py-faster-rcnn 服务端安装说明（Server Installation Manual）

> 来源文件：`py-faster-rcnn-InstallUserManual.txt`（2017 年前后整理的安装笔记）

## 硬件信息（Hardware Info）

| 项目 | 配置 |
| --- | --- |
| CPU | i7 6800K |
| Chipset | X99 |
| RAM | 32GB DDR4 |
| GPU | NVidia Titan X Pascal 12GB |

## 操作系统（OS）

`Ubuntu YakketyYak 16.10`

## 前置说明

* For instructions about installing Titan X Pascal driver, CUDA, cuDNN, Caffe, please refer to user manual of caffe installation.

## Python

```bash
sudo -H pip install cython
sudo -H apt-get install python-pip
sudo -H apt-get install python-dev
sudo -H pip install --upgrade pip
sudo -H pip install easydict
sudo apt-get install python-tk
sudo apt-get install python-protobuf
sudo apt-get install python-yaml
```

## OpenCV

```bash
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

Download OpenCV source code (latest version is 3) and extract to local dir (e.g. opencv)

```bash
cd ~/opencv
mkdir release
cd release
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
sudo make -j6
sudo make -j6 install
```

```bash
// test OpenCV
root@ubuntu:~/opencv-3.2.0/samples/python$ python demo.py 
```

## Caffe

```bash
// Download src
git clone --recursive https://github.com/rbgirshick/py-faster-rcnn.git
```

* py-faster-rcnn has a subdir of Caffe src, but this version of Caffe src cannot compile with CUDA 8.0
Use the latest src of Caffe to replace it.

Revise the Makefile.config of Caffe:

```bash
# In your Makefile.config, make sure to have this line uncommented
WITH_PYTHON_LAYER := 1
```

cuDNN v5.1 cannot compile, disable cuDNN or downgrade to cuDNN v4

```bash
# cuDNN acceleration switch (uncomment to build with cuDNN).
# USE_CUDNN := 1
```

Installed OpenCV 3

```bash
# Uncomment if you're using OpenCV 3
OPENCV_VERSION := 3
```

## 编译 Cython 模块

```bash
// build the Cython modules
cd $FRCN_ROOT/lib
make
```

```bash
cd $FRCN_ROOT/caffe-fast-rcnn
sudo make -j6 && sudo make -j6 pycaffe
```

## 运行示例（./tools/demo.py）

```bash
./tools/demo.py
```

## 常见错误与解决（Errors and Solutions）

```text
Error:  ImportError: No module named skimage.io
Solution: sudo -H pip install scikit-imag //this also install lots of dependencies
```

<!-- 原文此处疑似缺字 -->

```text
Error: ImportError: No module named gpu_nms
Solution: run make in lib folder
```
