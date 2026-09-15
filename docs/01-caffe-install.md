# 服务端安装说明（Caffe Install User Manual）

> 来源文件：`CaffeInstallUserManual.txt`（2017 年前后整理的安装笔记）
> 合并变体：`CaffeInstallUserManual(1).txt`、`CaffeInstallUserManual(2).txt`（三份变体取并集去重，差异处以 HTML 注释标注）

## Hardware Info（服务端硬件配置）

| 项目 | 配置 |
| --- | --- |
| CPU | i7 6800K |
| Chipset | X99 |
| RAM | 32GB DDR4 |
| GPU | NVidia Titan X Pascal 12GB |

## OS（操作系统选择）

### These Fails

(Additional Hardware → install driver no response → relogin cannot enter desktop environment)

- `ubuntu-16.10-desktop-amd64`
- `UbuntuKylin-1604-ukui-amd64`

### This one succeeds

- YakketyYak 16.10 Minimal Boot CD | Requires an Internet LAN Connection. Wireless Wifi driver may be unavailable

## Install Titan X Driver（安装 Titan X 驱动）

1. Additional Drivers
2. Using NVIDIA binary driver
3. Apply Changes
4. reboot
5. Screen resolution becomes normal

## Install CUDA

- 下载地址：https://developer.nvidia.com/cuda-downloads
<!-- 原文此行为 Word 域代码残留： HYPERLINK "https://developer.nvidia.com/cuda-downloads"https://developer.nvidia.com/cuda-downloads -->
- Select Target Platform :  Linux > x_64_64 > Ubuntu > 16.04 (there is not a 16.10 yet) > deb (local)
- Download the 1.9GB deb file. Follow the instructions:

```bash
sudo dpkg -i cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda
```
<!-- 原文这三条命令是行内代码（反引号包裹），此处改为代码块，命令本身未改动 -->
<!-- 原文如此：目标平台一栏写作 x_64_64 -->

### Problem: /sbin/ldconfig.real: libEGL.so.1 is not a symbolic link

<!-- 本节仅见于 CaffeInstallUserManual(2).txt -->

```text
Processing triggers for libc-bin (2.23-0ubuntu7) ...
/sbin/ldconfig.real: /usr/lib/nvidia-375/libEGL.so.1 is not a symbolic link

/sbin/ldconfig.real: /usr/lib32/nvidia-375/libEGL.so.1 is not a symbolic link
```

Solution:

```bash
sudo mv /usr/lib/nvidia-375/libEGL.so.1 /usr/lib/nvidia-375/libEGL.so.1.org

sudo mv /usr/lib32/nvidia-375/libEGL.so.1 /usr/lib32/nvidia-375/libEGL.so.1.org

sudo ln -s /usr/lib/nvidia-375/libEGL.so.375.66 /usr/lib/nvidia-375/libEGL.so.1

sudo ln -s /usr/lib32/nvidia-375/libEGL.so.375.66 /usr/lib32/nvidia-375/libEGL.so.1
```

## Compile CUDA samples（编译 CUDA samples）

```bash
cd /usr/local/cuda-8.0/samples/
sudo make all
```
<!-- 变体差异：CaffeInstallUserManual(1).txt 与 CaffeInstallUserManual(2).txt 此处为 sudo make -j10 all -->

### Error1: gcc version not supported. The system uses gcc 6

Solution:

```bash
sudo gedit /usr/local/cuda/include/host_config.h
```

```text
// comment out the #error line

#if __GNUC__ > 5
// #error -- unsupported GNU version! gcc versions later than 5 are not supported!
```

```bash
// install gcc 5 and set it as default gcc
// To figure out the current priorities of gcc
update-alternatives --query gcc
// Install gcc 5
sudo apt-get install gcc-5
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50 
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 50
// Make sure gcc 5 is the default
update-alternatives --config gcc
```

### Error2: /usr/bin/ld: cannot find -lnvcuvid

Solution:

```text
// Make sure the libs exist
zys@ubuntu:~$ sudo find / -name libnvcuvid.*
[sudo] password for zys: 
zys
/usr/lib/nvidia-375/libnvcuvid.so.1
/usr/lib/nvidia-375/libnvcuvid.so.375.26
/usr/lib/nvidia-375/libnvcuvid.so
/usr/lib32/nvidia-375/libnvcuvid.so.1
/usr/lib32/nvidia-375/libnvcuvid.so.375.26
/usr/lib32/nvidia-375/libnvcuvid.so
```

```bash
// Enter the project dir that throws error. Edit the findglib.mk file
cd /usr/local/cuda-8.0/samples/3_Imaging/cudaDecodeGL
sudo gedit findgllib.mk

revise to make sure UBUNTU_PKG_NAME = "nvidia-375"
```

### Run deviceQuery to make sure CUDA works

```text
zys@ubuntu:/usr/local/cuda-8.0/samples/bin/x86_64/linux/release$ ./deviceQuery
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "TITAN X (Pascal)"
  CUDA Driver Version / Runtime Version          8.0 / 8.0
  CUDA Capability Major/Minor version number:    6.1
  Total amount of global memory:                 12189 MBytes (12781158400 bytes)
  (28) Multiprocessors, (128) CUDA Cores/MP:     3584 CUDA Cores
  GPU Max Clock rate:                            1531 MHz (1.53 GHz)
  Memory Clock rate:                             5005 Mhz
  Memory Bus Width:                              384-bit
  L2 Cache Size:                                 3145728 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 8.0, CUDA Runtime Version = 8.0, NumDevs = 1, Device0 = TITAN X (Pascal)
Result = PASS
```

## Install cudnn（安装 cuDNN）

Go to Nvidia site, download and install:

1. libcudnn5-dev_5.1.10-1+cuda8.0_amd64.deb   Developer library
2. libcudnn5-doc_5.1.10-1+cuda8.0_amd64.deb   Code samples
<!-- 变体差异：CaffeInstallUserManual(2).txt 的列表为三项，在第一项前多出 "1. libcudnn5_5.1.10-1+cuda8.0_amd64.deb   Runtime library"，其余两项顺延为 2./3. -->

To ensure cuDNN is installed correctly, go to /usr/src/cudnn_samples_v5/mnistCUDNN

“sudo make” and test the generated mnistCUDNN:

```text
root@ubuntu:/usr/src/cudnn_samples_v5/mnistCUDNN$ ./mnistCUDNN image=data/five_28x28.pgm
cudnnGetVersion() : 5110 , CUDNN_VERSION from cudnn.h : 5110 (5.1.10)
Host compiler version : GCC 5.4.1
There are 1 CUDA capable devices on your machine :
device 0 : sms 28  Capabilities 6.1, SmClock 1531.0 Mhz, MemSize (Mb) 12189, MemClock 5005.0 Mhz, Ecc=0, boardGroupID=0
Using device 0
Loading image data/five_28x28.pgm
Performing forward propagation ...
Testing cudnnGetConvolutionForwardAlgorithm ...
Fastest algorithm is Algo 1
Testing cudnnFindConvolutionForwardAlgorithm ...
^^^^ CUDNN_STATUS_SUCCESS for Algo 0: 0.022528 time requiring 0 memory
^^^^ CUDNN_STATUS_SUCCESS for Algo 2: 0.036864 time requiring 57600 memory
^^^^ CUDNN_STATUS_SUCCESS for Algo 1: 0.051200 time requiring 19508 memory
^^^^ CUDNN_STATUS_SUCCESS for Algo 7: 0.077824 time requiring 2057744 memory
^^^^ CUDNN_STATUS_SUCCESS for Algo 5: 0.112640 time requiring 205008 memory
Resulting weights from Softmax:
0.0000000 0.0000008 0.0000000 0.0000002 0.0000000 0.9999820 0.0000154 0.0000000 0.0000012 0.0000006 

Result of classification: 5
```

## Caffe prerequisite（Caffe 依赖库）

```bash
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
sudo apt-get install --no-install-recommends libboost-all-dev
sudo apt-get install libatlas-base-dev
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
```
<!-- 原文此处把下面两条命令合并在一行（提取时丢失换行），按两条命令拆开：
     sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler sudo apt-get install --no-install-recommends libboost-all-dev
     -->
<!-- 变体差异：CaffeInstallUserManual(2).txt 把后两条命令合并为一行：sudo apt-get install libatlas-base-dev libgflags-dev libgoogle-glog-dev liblmdb-dev -->

## Compile Caffe（编译 Caffe）

### Compile NV Caffe

<!-- 本节仅见于 CaffeInstallUserManual(2).txt，位于 BLVC Caffe 之前 -->

NVcaffe is the NVIDIA branch of Caffe with optimizations for GPU. NVcaffe uses cuDNN and is used by DIGITS for training DNNs. To install it, clone the NVcaffe repo from GitHub and compile from source. First some prequisite packages for Caffe are installed, including the Python bindings required by DIGITS:

```bash
$ sudo apt-get install --no-install-recommends build-essential cmake git gfortran libatlas-base-dev libboost-filesystem-dev libboost-python-dev libboost-system-dev libboost-thread-dev libgflags-dev libgoogle-glog-dev libhdf5-serial-dev libleveldb-dev liblmdb-dev libprotobuf-dev libsnappy-dev protobuf-compiler python-all-dev python-dev python-h5py python-matplotlib python-numpy python-opencv python-pil python-pip python-protobuf python-scipy python-skimage python-sklearn python-setuptools
$ sudo apt-get install libopencv-dev python-opencv
$ sudo pip install --upgrade pip
$ git clone http://github.com/NVIDIA/caffe
$ cd caffe
$ sudo pip install -r python/requirements.txt 
$ mkdir build
$ cd build
$ cmake ../ -DCUDA_USE_STATIC_CUDA_RUNTIME=OFF
$ make -j12 all
$ make pycaffe
```

### Compile BLVC Caffe

<!-- 基础版本的章节名即 Compile Caffe，内容与本节相同 -->

```bash
sudo apt-get install git
git clone https://github.com/BVLC/caffe.git
cp Makefile.config.example Makefile.config
sudo gedit Makefile.config
```
<!-- 原文此处为 Word 域代码残留：git clone “ HYPERLINK "https://github.com/BVLC/caffe"https://github.com/BVLC/caffe.git” -->

uncomment “USE_CUDNN := 1”

Revise:

```makefile
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib
```

To:

```makefile
# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/i386-linux-gnu/hdf5/serial
```

or

```makefile
# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu/hdf5/serial/
```

<!-- 原文此处把 Revise/To 两段说明与两段配置合并为同一行（提取时丢失换行）：Revise: INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib To: # Whatever else you find you need goes here. INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/i386-linux-gnu/hdf5/serial -->

```text
zys@ubuntu:~/caffe-master$ sudo find / -name libhdf5.so
/usr/lib/x86_64-linux-gnu/hdf5/serial/libhdf5.so
```

save & close

```bash
sudo make -j6 all
```
-j6 to enable parallel compile (i7 6800K has 6 cores)
<!-- 变体差异：CaffeInstallUserManual(1).txt 与 CaffeInstallUserManual(2).txt 此处为 sudo make -j12 all，其后注释同为 "-j12 to enable parallel compile (i7 6800K has 6 cores)" -->

### Error1: cannot find -lhdf5_hl / -lhdf5

```text
AR -o .build_release/lib/libcaffe.a
LD -o .build_release/lib/libcaffe.so.1.0.0-rc5
/usr/bin/ld: cannot find -lhdf5_hl
/usr/bin/ld: cannot find -lhdf5
collect2: error: ld returned 1 exit status
Makefile:572: recipe for target '.build_release/lib/libcaffe.so.1.0.0-rc5' failed
make: *** [.build_release/lib/libcaffe.so.1.0.0-rc5] Error 1
```

## 测试 mnist 例子：

```bash
    cd $CAFFE_ROOT
    ./data/mnist/get_mnist.sh
    ./examples/mnist/create_mnist.sh
    ./examples/mnist/train_lenet.sh
```

实测发现，Titan X Pascal开启cuDNN的情况下，train耗时约10s；不开启cuDNN约2min。
而ThinkPad Carbon X1 （CPU模式）和NVIDIA Jetson TK1 （GPU模式+cuDNN）耗时均在12min左右。
速度差距明显，达到了70多倍

## Use pycaffe for prediction（用 pycaffe 做预测）

```bash
sudo make -j6
sudo make -j6 pycaffe
```
<!-- 变体差异：CaffeInstallUserManual(1).txt 与 CaffeInstallUserManual(2).txt 此处为 sudo make -j12 与 sudo make -j12 pycaffe -->

error:

```text
fatal error: numpy/arrayobject.h: No such file or directory
```

solution:

```text
python 
>>> import numpy
```

check if numpy is installed

check site-packages directory

```text
>>> import site; site.getsitepackages()
['/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages']
```

Check which folder numpy is installed in.

revise Makefile.config accordingly:

```makefile
# NOTE: this is required only if you will compile the python interface.
# We need to be able to find Python.h and numpy/arrayobject.h.
PYTHON_INCLUDE := /usr/include/python2.7 \
		/usr/local/lib/python2.7/dist-packages/numpy/core/include
#		/usr/lib/python2.7/dist-packages/numpy/core/include
```

```bash
export PYTHONPATH=/home/zys/caffe-master/distribute/python
export LD_LIBRARY_PATH=/home/zys/caffe-master/distribute/lib
```

Or persist the parameter by conf file:

```bash
sudo gedit /etc/ld.so.conf.d/caffe.conf
```

add the line: /home/zys/caffe-master/distribute/lib

```bash
sudo ldconfig
```

<!-- 原文此处把这四段合并成了同一行（提取时丢失换行）：Or persist the parameter by conf file: sudo gedit /etc/ld.so.conf.d/caffe.conf add the line: /home/zys/caffe-master/distribute/lib sudo ldconfig -->

write a test python script file (test_googlenet.py) as follows:

```python
#coding=utf-8

import numpy
import os
import sys
import caffe


caffe_root = '/home/zys/caffe-master/'
sys.path.insert(0,caffe_root+'Python')


MODEL_FILE =caffe_root+'models/bvlc_googlenet/deploy.prototxt'
PRETRAINED =caffe_root+'models/bvlc_googlenet/bvlc_googlenet.caffemodel'

#cpu模式

caffe.set_mode_gpu()

#定义使用的神经网络模型
#image_dims : dimensions to scale input for cropping/sampling.
#        Default is to scale to net input size for whole-image crop.
#    mean, input_scale, raw_scale, channel_swap: params for
#        preprocessing options.

net = caffe.Classifier(MODEL_FILE, PRETRAINED, 
               mean=numpy.load(caffe_root +'python/caffe/imagenet/ilsvrc_2012_mean.npy').mean(1).mean(1),
               channel_swap=(2,1,0),
               raw_scale=255,
               # image_dims=(224, 224)
)

imagenet_labels_filename = caffe_root +'data/ilsvrc12/synset_words.txt'
labels =numpy.loadtxt(imagenet_labels_filename, str, delimiter='\t')

#对目标路径中的图像，遍历并分类

for root,dirs,files in os.walk(caffe_root + "examples/images/"):
   for file in files:
       #加载要分类的图片

       IMAGE_FILE = os.path.join(root,file).decode('gbk').encode('utf-8');
       input_image = caffe.io.load_image(IMAGE_FILE)   
       print("\n---- {} ----".format(IMAGE_FILE))
 
       #预测图片类别

       prediction = net.predict([input_image])
       print 'predicted class:',prediction[0].argmax()
 
       # 输出概率最大的前5个预测结果

       top_k = net.blobs['prob'].data[0].flatten().argsort()[-1:-6:-1]
       print labels[top_k]
```
<!-- 原文如此：#cpu模式 注释的下一行调用的是 caffe.set_mode_gpu() -->

```bash
python test_googlenet.py
```

