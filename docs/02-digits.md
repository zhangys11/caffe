# NVIDIA DIGITS 安装与使用笔记（NVIDIA DIGITS Install & Usage Notes）

> 来源文件：`DIGITS.txt`（2017 年前后整理的安装笔记）

## 数据准备（Data Prepare）

linux way:

```bash
cd /mnt/seagate/ROP_Unclassified_2016/
for f in $(find . -name '*.jpg' -or -name '*.doc'); do cp $f /mnt/seagate/$RANDOM$RANDOM$RANDOM$RANDOM.jpg; done
```

windows way:

Use `System.App.Web.ROP.Tool.Label.exe`

## 安装 NVIDIA-DIGITS（INSTALL NVIDIA-DIGITS）

```bash
sudo apt-get install --no-install-recommends git graphviz python-dev python-flask python-flaskext.wtf python-gevent python-h5py python-numpy python-pil python-pip python-protobuf python-scipy
git clone https://github.com/NVIDIA/DIGITS.git
// this session only
# export CAFFE_ROOT=/home/zys/caffe-master/  
export CAFFE_ROOT=/home/zys/NVcaffe/   # should use NV flavor

// write to user profile
echo "export CAFFE_ROOT=/home/zys/NVcaffe/" >> ~/.profile 
source ~/.profile
echo $CAFFE_ROOT
```

## 报错与解决：CAFFE_ROOT 指向的 Caffe 无法被导入（ERROR / SOLUTION）

ERROR:

```text
zys@ubuntu:~/DIGITS$ ./digits-devserver 
  ___ ___ ___ ___ _____ ___
 |   \_ _/ __|_ _|_   _/ __|
 | |) | | (_ || |  | | \__ \
 |___/___\___|___| |_| |___/ 5.1-dev

"/home/zys/NVcaffe/" from CAFFE_ROOT does not point to a valid installation of Caffe.
Use the envvar CAFFE_ROOT to indicate a valid installation.
Traceback (most recent call last):
  File "/usr/lib/python2.7/runpy.py", line 174, in _run_module_as_main
    "__main__", fname, loader, pkg_name)
  File "/usr/lib/python2.7/runpy.py", line 72, in _run_code
    exec code in run_globals
  File "/home/zys/DIGITS/digits/__main__.py", line 70, in <module>
    main()
  File "/home/zys/DIGITS/digits/__main__.py", line 53, in main
    import digits.config
  File "digits/config/__init__.py", line 7, in <module>
    from . import (  # noqa
  File "digits/config/caffe.py", line 226, in <module>
    executable, version, flavor = load_from_envvar('CAFFE_ROOT')
  File "digits/config/caffe.py", line 37, in load_from_envvar
    import_pycaffe(python_dir)
  File "digits/config/caffe.py", line 126, in import_pycaffe
    import caffe
  File "/home/zys/NVcaffe/python/caffe/__init__.py", line 1, in <module>
    from .pycaffe import Net, SGDSolver, NesterovSolver, AdaGradSolver, RMSPropSolver, AdaDeltaSolver, AdamSolver
  File "/home/zys/NVcaffe/python/caffe/pycaffe.py", line 15, in <module>
    import caffe.io
  File "/home/zys/NVcaffe/python/caffe/io.py", line 8, in <module>
    from caffe.proto import caffe_pb2
  File "/home/zys/NVcaffe/python/caffe/proto/caffe_pb2.py", line 23, in <module>
    0\x64\x65t_fg_threshold\x18\x36 \x01(\x02:\x03\x30.5\x12\x1d\n\x10\x64\x65t_bg_threshold\x18\x37 \x01(\x02:\x03\x30.5\x12\x1d\n\x0f\x64\x65t_fg_fraction\x18\x38 \x01(\x02:\x04\x30.25\x12\x1a\n\x0f\x64\x65t_context_pad\x18: \x01(\r:\x01\x30\x12\x1b\n\rdet_crop_mode\x18; \x01(\t:\x04warp\x12\x12\n\x07new_num\x18< \x01(\x05:\x01\x30\x12\x17\n\x0cnew_channels\x18= \x01(\x05:\x01\x30\x12\x15\n\nnew_height\x18> \x01(\x05:\x01\x30\x12\x14\n\tnew_width\x18? \x01(\x05:\x01\x30\x12\x1d\n\x0eshuffle_images\x18@ \x01(\x08:\x05\x66\x61lse\x12\x15\n\nconcat_dim\x18\x41 \x01(\r:\x01\x31\x12\x36\n\x11hdf5_output_param\x18\xe9\x07 \x01(\x0b\x32\x1a.caffe.HDF5OutputParameter\".\n\nPoolMethod\x12\x07\n\x03MAX\x10\x00\x12\x07\n\x03\x41VE\x10\x01\x12\x0e\n\nSTOCHASTIC\x10\x02\"W\n\x0ePReLUParameter\x12&\n\x06\x66iller\x18\x01 \x01(\x0b\x32\x16.caffe.FillerParameter\x12\x1d\n\x0e\x63hannel_shared\x18\x02 \x01(\x08:\x05\x66\x61lse*\x1c\n\x05Phase\x12\t\n\x05TRAIN\x10\x00\x12\x08\n\x04TEST\x10\x01')
TypeError: __init__() got an unexpected keyword argument 'syntax'
```

SOLUTION: reinstall protobuf

```bash
zys@ubuntu:~/DIGITS$ sudo pip uninstall protobuf  // this uninstalls 2.6.1
zys@ubuntu:~/DIGITS$ sudo pip install protobuf  // this installs 3.2.0
```

## 启动并访问 Web 界面（Open browser）

Open browser, go to http://localhost:5000

## 示例：MNIST（Example: MNIST）

https://github.com/NVIDIA/DIGITS/blob/master/docs/GettingStarted.md

Use the python script "download_mnist.py" to prepare MNIST data set:

```python
# Download and extract the MNIST dataset
# Modified by zys for Pyhont 3

import gzip
import os
import struct
import urllib.request
import shutil

import numpy as np
import PIL.Image

class MnistDownloader:
    """
    See details about the MNIST dataset here:
    http://yann.lecun.com/exdb/mnist/
    """
    def __init__(self, outdir, clean=False, file_extension='png'):
        """
        Arguments:
        outdir -- directory where to download and create the dataset
        if this directory doesn't exist, it will be created
        Keyword arguments:
        clean -- delete outdir first if it exists
        file_extension -- image format for output images
        """
        self.outdir = outdir
        self.mkdir(self.outdir, clean=clean)
        self.file_extension = file_extension.lower()


    def getData(self):
        """
        This is the main function that should be called by the users!
        Downloads the dataset and prepares it for DIGITS consumption
        """
        for url in self.urlList():
            self.__downloadFile(url)

        self.uncompressData()

        self.processData()
        print ("Dataset directory is created successfully at {}".format(self.outdir))

    def __downloadFile(self, url):
        """
        Downloads the url
        """
        download_path = os.path.join(self.outdir, os.path.basename(url))
        if not os.path.exists(download_path):
            print ("Downloading url={} ...".format(url))
            urllib.request.urlretrieve(url, download_path)

    def mkdir(self, d, clean=False):
        """
        Safely create a directory
        Arguments:
        d -- the directory name
        Keyword arguments:
        clean -- if True and the directory already exists, it will be deleted and recreated
        """
        if os.path.exists(d):
            if clean:
                shutil.rmtree(d)
            else:
                return
        os.mkdir(d)

    def urlList(self):
        return [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz',
        ]

    def uncompressData(self):
        for zipped, unzipped in [
                ('train-images-idx3-ubyte.gz',  'train-images.bin'),
                ('train-labels-idx1-ubyte.gz',  'train-labels.bin'),
                ('t10k-images-idx3-ubyte.gz',   'test-images.bin'),
                ('t10k-labels-idx1-ubyte.gz',   'test-labels.bin'),
        ]:
            zipped_path = os.path.join(self.outdir, zipped)
            assert os.path.exists(zipped_path), 'Expected "%s" to exist' % zipped
            unzipped_path = os.path.join(self.outdir, unzipped)
            if not os.path.exists(unzipped_path):
                print ("Uncompressing file={} ...".format(zipped))
                with gzip.open(zipped_path) as infile, open(unzipped_path, 'wb') as outfile:
                    outfile.write(infile.read())

    def processData(self):
        self.__extract_images('train-images.bin', 'train-labels.bin', 'train')
        self.__extract_images('test-images.bin', 'test-labels.bin', 'test')

    def __extract_images(self, images_file, labels_file, phase):
        """
        Extract information from binary files and store them as images
        """
        labels = self.__readLabels(os.path.join(self.outdir, labels_file))
        images = self.__readImages(os.path.join(self.outdir, images_file))
        assert len(labels) == len(images), '%d != %d' % (len(labels), len(images))

        output_dir = os.path.join(self.outdir, phase)
        self.mkdir(output_dir, clean=True)
        with open(os.path.join(output_dir, 'labels.txt'), 'w') as outfile:
            for label in range(10):
                outfile.write('%s\n' % label)
        with open(os.path.join(output_dir, '%s.txt' % phase), 'w') as outfile:
            for index, image in enumerate(images):
                dirname = os.path.join(output_dir, labels[index])
                self.mkdir(dirname)
                filename = os.path.join(dirname, '%05d.%s' % (index, self.file_extension))
                image.save(filename)
                outfile.write('%s %s\n' % (filename, labels[index]))

    def __readLabels(self, filename):
        """
        Returns a list of ints
        """
        print ('Reading labels from {} ...'.format(filename))
        labels = []
        with open(filename, 'rb') as infile:
            infile.read(4)  # ignore magic number
            count = struct.unpack('>i', infile.read(4))[0]
            data = infile.read(count)
            for byte in data:
                # http://stackoverflow.com/questions/42347498/translating-python-2-byte-checksum-calculator-to-python-3                
                label = byte #struct.unpack('>B', byte)[0]
                labels.append(str(label))
        return labels

    def __readImages(self, filename):
        """
        Returns a list of PIL.Image objects
        """
        print ('Reading images from {} ...'.format(filename))
        images = []
        with open(filename, 'rb') as infile:
            infile.read(4)  # ignore magic number
            count = struct.unpack('>i', infile.read(4))[0]
            rows = struct.unpack('>i', infile.read(4))[0]
            columns = struct.unpack('>i', infile.read(4))[0]

            for i in range(count):
                data = infile.read(rows * columns)
                image = np.fromstring(data, dtype=np.uint8)
                image = image.reshape((rows, columns))
                image = 255 - image  # now black digit on white background
                images.append(PIL.Image.fromarray(image))
        return images
    
    
# Execute
MnistDownloader("g:/mnist", False).getData()
```

## 创建数据集并训练模型（Create Dataset and Train Model）

Example: MNIST fine tune

https://github.com/NVIDIA/DIGITS/tree/master/examples/fine-tuning

## 设置数据集（SET DATASETS）

```text
zys@ubuntu:~/DIGITS$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0          11:0    1  1024M  0 rom  
sda           8:0    0 931.5G  0 disk 
└─sda1        8:1    0 931.5G  0 part /media/zys/Seagate Backup Plus Drive
nvme0n1     259:0    0 238.5G  0 disk 
├─nvme0n1p5 259:5    0  14.9G  0 part [SWAP]
├─nvme0n1p3 259:3    0    16M  0 part 
├─nvme0n1p1 259:1    0   450M  0 part 
├─nvme0n1p6 259:6    0 105.1G  0 part /
├─nvme0n1p4 259:4    0   118G  0 part /media/zys/0A24AF0D24AEFAB9
└─nvme0n1p2 259:2    0   100M  0 part 
```

```text
zys@ubuntu:~$ ll /dev | grep sda1
brw-rw----   1 root disk      8,   1 Mar  7 22:31 sda1
```

```bash
mkdir -p ~/seagate  // create if not exist
sudo mount /dev/sda1 ~/seagate
```

## 尺寸提示：使用 GoogLeNet 需要 256×256

Use 256X256 to use GoogLeNet

## 为 RetCam / 非 RetCam 分类器做微调（Finetune for RetCam and non-RetCam classifier）

generate_train_test_txt.sh:

<!-- 原文此处疑似缺字 -->

## data_layer.cpp:73 报错说明

If output two many:

```text
data_layer.cpp:73] Restarting data prefetching from start.
```

Means:

- You gave the wrong .txt file to data layer
- The format of the .txt file is not as expected by Caffe
- Very few number of data is present in the file.
