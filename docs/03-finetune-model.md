# Caffe 模型微调笔记（Fine-tune Model：finetune_flickr_style / ROP / GoogLeNet）

> 来源文件：`FinetuneModel.txt`（基线）、`FinetuneModel(1).txt`、`FinetuneModel(2).txt`（2017 年前后整理的安装笔记，本文为三份内容的合并）

## 示例：Caffe 中的 finetune_flickr_style（Example: finetune_flickr_style in Caffe）

### solver.prototxt

```prototxt
net: "models/finetune_flickr_style/train_val.prototxt"
test_iter: 100
test_interval: 1000
# lr for fine-tuning should be lower than when starting from scratch. Original AlexNet uses 0.01
base_lr: 0.001
lr_policy: "step"
gamma: 0.1
# stepsize should also be lower, as we're closer to being done. Original AlexNet uses 100000
stepsize: 20000
display: 20
max_iter: 100000
momentum: 0.9
weight_decay: 0.0005
snapshot: 10000
snapshot_prefix: "models/finetune_flickr_style/finetune_flickr_style"
# uncomment the following to default to CPU mode solving
solver_mode: GPU
```

### train_val.prototxt（只列改动过的分类层）

```prototxt
layer {
  # rename layer name
  name: "fc8_flickr"
  type: "InnerProduct"
  bottom: "fc7"
  top: "fc8_flickr"
  # lr_mult is set to higher than for other layers, because this layer is starting from random while the others are already trained
  param {
    # Original AlexNet uses 1
    lr_mult: 10
    decay_mult: 1
  }
  param {
    # Original AlexNet uses 2
    lr_mult: 20
    decay_mult: 0
  }
  inner_product_param {
    # change to your number of classes. Original AlexNet has 1000 classes
    num_output: 20
    weight_filler {
      type: "gaussian"
      std: 0.01
    }
    bias_filler {
      type: "constant"
      value: 0
    }
  }
}
```

### 训练（Train）

```bash
./build/tools/caffe train -solver models/finetune_flickr_style/solver.prototxt -weights models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel -gpu 0
```

## 针对 ROP 的微调（Finetune for ROP）

`caffe-master/models/finetune_alexnet_rop/solver.prototxt`

```prototxt
net: "models/finetune_alexnet_rop/train_val.prototxt"
test_iter: 100
test_interval: 1000
# lr for fine-tuning should be lower than when starting from scratch. Original AlexNet uses 0.01
base_lr: 0.001
lr_policy: "step"
gamma: 0.1
# stepsize should also be lower, as we're closer to being done. Original AlexNet uses 100000
stepsize: 20000
display: 20
max_iter: 100000
momentum: 0.9
weight_decay: 0.0005
snapshot: 10000
snapshot_prefix: "models/finetune_alexnet_rop/finetune_alexnet_rop"
# uncomment the following to default to CPU mode solving
solver_mode: GPU
```

Comparison with caffe-master/models/bvlc_alexnet/solver.prototxt

### train_val.prototxt

`caffe-master/models/finetune_alexnet_rop/train_val.prototxt`

```prototxt
name: "RopCaffeNet"
layer {
  name: "data"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TRAIN
  }
  transform_param {
    mirror: true
    crop_size: 227
    # because it's finetuning. We use the mean file of ImageNet, other than RetCam
    mean_file: "data/ilsvrc12/imagenet_mean.binaryproto"
  }
  data_param {
    source: "data/rop/train_db"
    batch_size: 50
    backend: LMDB    
  }
}
layer {
  name: "data"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TEST
  }
  transform_param {
    mirror: false
    crop_size: 227
    mean_file: "data/ilsvrc12/imagenet_mean.binaryproto"
  }
  data_param {
    source: "data/rop/val_db"
    batch_size: 50
    backend: LMDB
  }
}
… ...
layer {
  name: "fc8_rop"
  type: "InnerProduct"
  bottom: "fc7"
  top: "fc8_rop"
  # lr_mult is set to higher than for other layers, because this layer is starting from random while the others are already trained
  param {
    lr_mult: 10
    decay_mult: 1
  }
  param {
    lr_mult: 20
    decay_mult: 0
  }
  inner_product_param {
    num_output: 2
    weight_filler {
      type: "gaussian"
      std: 0.01
    }
    bias_filler {
      type: "constant"
      value: 0
    }
  }
}
layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "fc8_rop"
  bottom: "label"
  top: "accuracy"
  include {
    phase: TEST
  }
}
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "fc8_rop"
  bottom: "label"
  top: "loss"
}
```

### 相应修改 deploy.prototxt（change deploy.prototxt accordingly）

```prototxt
name: "RopCaffeNet"
layer {
  name: "data"
  type: "Input"
  top: "data"
  input_param { shape: { dim: 10 dim: 3 dim: 227 dim: 227 } }
}
… … 
layer {
  name: "fc8_rop"
  type: "InnerProduct"
  bottom: "fc7"
  top: "fc8_rop"
  # lr_mult is set to higher than for other layers, because this layer is starting from random while the others are already trained
  param {
    lr_mult: 10
    decay_mult: 1
  }
  param {
    lr_mult: 20
    decay_mult: 0
  }
  inner_product_param {
    num_output: 2
    weight_filler {
      type: "gaussian"
      std: 0.01
    }
    bias_filler {
      type: "constant"
      value: 0
    }
  }
}
layer {
  name: "prob"
  type: "Softmax"
  bottom: "fc8_rop"
  top: "prob"
}
```

### TRAIN

```bash
./build/tools/caffe train -solver models/finetune_alexnet_rop/solver.prototxt -weights models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel -gpu 0
```

### 报错：DIGITS 中 visualize model（Problem / Solution）

Problem:

In DIGITS, visualize the model generates error:

```text
'google.protobuf.pyext._message.RepeatedScalarConta' object has no attribute '_values'
```

Solution:

Edit draw.py, revise as:

```python
                      1, #layer.convolution_param.kernel_size[0] if len(layer.convolution_param.kernel_size._values) else 1,
                      separator,
                      1, #layer.convolution_param.stride[0] if len(layer.convolution_param.stride._values) else 1,
                      separator,
                      0) # layer.convolution_param.pad[0] if len(layer.convolution_param.pad._values) else 0)
```

### TEST

```bash
export PYTHONPATH=/home/zys/caffe-master/distribute/python
export LD_LIBRARY_PATH=/home/zys/caffe-master/distribute/lib
python test_rop.py
```

`test_rop.py`：

```python
#coding=utf-8

import numpy
import os
import sys
import caffe


caffe_root = '/home/zys/caffe-master/'
sys.path.insert(0,caffe_root+'Python')


MODEL_FILE =caffe_root+'models/finetune_alexnet_rop/deploy.prototxt'
PRETRAINED =caffe_root+'models/finetune_alexnet_rop/finetune_alexnet_rop_iter_100000.caffemodel'

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

imagenet_labels_filename = caffe_root +'models/finetune_alexnet_rop/rop/labels.txt'
labels =numpy.loadtxt(imagenet_labels_filename, str, delimiter='\t')

#对目标路径中的图像，遍历并分类

for root,dirs,files in os.walk(caffe_root + "examples/images/ROP/"):
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

## 微调 GoogLeNet（Finetuning for GoogLeNet）

Assuming you are trying to do image classification. These should be the steps for finetuning a model:

1. Classification layer

The original [classification layer "loss3/classifier"](https://github.com/BVLC/caffe/blob/master/models/bvlc_googlenet/train_val.prototxt) outputs predictions for 1000 classes (it's mum_output is set to 1000). You'll need to replace it with a new layer with appropriate num_output. Replacing the classification layer:

- Change layer's name (so that when you read the original weights from caffemodel file there will be no conflict with the weights of this layer).
- Change num_output to the right number of output classes you are trying to predict.
- Note that you need to change ALL classification layers. Usually there is only one, but GoogLeNet happens to have three: [loss1/classifier](https://github.com/BVLC/caffe/blob/master/models/bvlc_googlenet/train_val.prototxt#L904), [loss2/classifier](https://github.com/BVLC/caffe/blob/master/models/bvlc_googlenet/train_val.prototxt#L1667) and [loss3/classifier](https://github.com/BVLC/caffe/blob/master/models/bvlc_googlenet/train_val.prototxt).

```prototxt
#
name: "GoogLeNet"
…

layer {
  bottom: "loss1/fc/bn"
  top: "loss1/fc/bn"
  name: "loss1/fc/bn/relu"
  type: "ReLU"
}
layer {
  bottom: "loss1/fc/bn"
  top: "loss1_c5/classifier"
  name: "loss1_c5/classifier"
  type: "InnerProduct"
  param {
    lr_mult: 10
    decay_mult: 1
  }
  param {
    lr_mult: 20
    decay_mult: 0
  }
  inner_product_param {
    num_output: 5
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
      value: 0
    }
  }
}
```

## 数据准备（Data Prepare）

（仅基线文件 `FinetuneModel.txt` 有此段）

```bash
cd /mnt/seagate/ROP_Unclassified_2016/
for f in $(find . -name '*.jpg' -or -name '*.doc'); do cp $f /mnt/seagate/$RANDOM$RANDOM$RANDOM$RANDOM.jpg; done
```

## 安装 NVIDIA-DIGITS（INSTALL NVIDIA-DIGITS）

（仅基线文件 `FinetuneModel.txt` 有此段）

```bash
sudo apt-get install --no-install-recommends git graphviz python-dev python-flask python-flaskext.wtf python-gevent python-h5py python-numpy python-pil python-pip python-protobuf python-scipy
git clone https://github.com/NVIDIA/DIGITS.git
// this session only
export CAFFE_ROOT=/home/zys/caffe-master/  

// write to user profile
echo "export CAFFE_ROOT=/home/zys/caffe-master/" >> ~/.profile 
source ~/.profile
echo $CAFFE_ROOT
```

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

## 设置数据集（SET DATASETS）

（仅基线文件 `FinetuneModel.txt` 有此段）

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

## 变体差异（Variant Differences）

三份文件共同的内容：`finetune_flickr_style` 示例（solver.prototxt、train_val.prototxt 中的 `fc8_flickr` 层、训练命令）、`Finetune for ROP` 的 solver.prototxt 与 train_val.prototxt（两个 data layer、`fc8_rop`、`accuracy`、`loss`）。

`FinetuneModel(1).txt` 比基线多出的内容：

- `change deploy.prototxt accrodingly` 及 deploy.prototxt 片段（`Input` 层 + `fc8_rop` + `prob`）；
- TRAIN 段的训练命令 `./build/tools/caffe train -solver models/finetune_alexnet_rop/solver.prototxt -weights models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel -gpu 0`；
- TEST 段的 `export PYTHONPATH` / `export LD_LIBRARY_PATH` / `python test_rop.py` 以及 `test_rop.py` 全文。

`FinetuneModel(2).txt` 比基线多出的内容：

- DIGITS 中 visualize 模型报错 `'google.protobuf.pyext._message.RepeatedScalarConta' object has no attribute '_values'` 及 `draw.py` 的改法；
- `Finetuning for GoogLeNet` 全文（文字说明 + `loss1/fc/bn/relu`、`loss1_c5/classifier` 片段）；
- 与 (1) 相同的 deploy.prototxt / TRAIN 命令 / TEST 段 / `test_rop.py`（(1) 与 (2) 中 `test_rop.py` 内容逐行一致）。

基线独有、两个变体都没有的段落：`Data Prepare:`、`INSTALL NVIDIA-DIGITS`（含 `digits-devserver` 报错与 protobuf 重装）、`SET DATASETS`、`Use 256X256 to use GoogLeNet`。任务说明中提到“其中一份删掉了 Data Prepare 段”，实际 `diff` 核对结果是 (1) 和 (2) 都删掉了上述四段；本文按并集保留，因此这几段与 `02-digits.md` 内容重复。

基线内部的冲突（原文如此，未做统一）：

- `INSTALL NVIDIA-DIGITS` 段写入的是 `export CAFFE_ROOT=/home/zys/caffe-master/`，而紧随其后的报错文本引用的是 `"/home/zys/NVcaffe/" from CAFFE_ROOT does not point to a valid installation of Caffe.`；
- 同一段在 `DIGITS.txt` 中写作 `export CAFFE_ROOT=/home/zys/NVcaffe/   # should use NV flavor`，并把 `# export CAFFE_ROOT=/home/zys/caffe-master/` 注释掉；
- 基线里 `TRAIN`、`TEST` 只有标题、没有命令，命令来自变体 (1) 与 (2)。

抽取痕迹：两份变体里 GoogLeNet 段落中的 Word 域代码 `HYPERLINK "…"` 已按可见文本还原为链接（`\l "L904"`、`\l "L1667"` 还原为 `#L904`、`#L1667`）；`DIGITS.txt` 中 `Open browser, go to HYPERLINK "http://localhost:5000/"http://localhost:5000` 同样按可见文本还原为 `http://localhost:5000`。
