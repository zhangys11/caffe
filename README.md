# Ubuntu + CUDA 8.0 + cuDNN 5 + Caffe + DIGITS 安装与使用手册（2017）

本仓库整理自 2017 年前后在 **Ubuntu 16.10（Yakkety Yak）** 服务器上搭建 Caffe 深度学习环境时留下的一批操作笔记：原始文件是 Word 文档（`.doc`）、PDF 讲义与若干纯文本片段，现已逐份转录、合并为结构化 Markdown。

整理原则：**忠实转录**。原文中的命令、路径、文件名、报错信息与解决步骤原样保留（含原文拼写），仅做排版结构化；提取过程中疑似缺字/串行之处用 `<!-- 原文此处疑似缺字 -->` 就地标注，不做猜测补写。原始文件见 [`source/`](source/)。

## 环境与版本

| 项目 | 配置 / 版本 |
| --- | --- |
| CPU / 主板 | Intel i7 6800K / X99 |
| 内存 | 32GB DDR4 |
| GPU | NVIDIA Titan X Pascal 12GB |
| 操作系统 | Ubuntu 16.10 Yakkety Yak Minimal Boot CD（需有线网络；`ubuntu-16.10-desktop`、`ubuntukylin-16.04-ukui` 安装驱动失败，见 01） |
| NVIDIA 驱动 | `nvidia-375`（Additional Drivers → NVIDIA binary driver） |
| CUDA | 8.0（`cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb`） |
| cuDNN | v5.1.10（`libcudnn5_5.1.10-1+cuda8.0_amd64.deb`） |
| 编译器 | gcc-5（CUDA 8.0 不支持 gcc 6，需 `update-alternatives` 切换） |
| 深度学习框架 | BVLC Caffe（`BVLC/caffe`）与 NVcaffe（`NVIDIA/caffe`，DIGITS 使用） |
| 可视化 | NVIDIA DIGITS（Web：`http://localhost:5000`） |
| 其他 | py-faster-rcnn（`rbgirshick/py-faster-rcnn`）、OpenCV 3.2.0 |

## 文档索引

| 文档 | 内容 | 原始文件 |
| --- | --- | --- |
| [01 Caffe 安装手册](docs/01-caffe-install.md) | 硬件与操作系统选型、Titan X 驱动、CUDA 8.0 安装、编译 CUDA samples 及报错、cuDNN 安装与验证、Caffe 依赖库、编译 NVcaffe/BVLC Caffe、pycaffe 预测、MNIST 测试 | `CaffeInstallUserManual.doc`（并合并 `(1)`、`(2)` 两个变体） |
| [02 DIGITS 安装与使用](docs/02-digits.md) | NVIDIA DIGITS 安装、`CAFFE_ROOT` 与 protobuf 报错、启动 Web 界面、MNIST 示例、Create Dataset 与训练、数据集挂载、GoogLeNet 的 256×256 尺寸要求、RetCam 微调 | `DIGITS.doc` |
| [03 模型微调笔记](docs/03-finetune-model.md) | `finetune_flickr_style` 示例、ROP 分类器微调（solver / train_val / deploy.prototxt）、TRAIN/TEST 命令、`test_rop.py`、`draw.py` 报错修复、GoogLeNet 微调 | `FinetuneModel.doc`（并合并 `(1)`、`(2)` 两个变体） |
| [04 py-faster-rcnn 安装](docs/04-py-faster-rcnn-install.md) | Python/OpenCV 依赖、用新版 Caffe 源码替换自带源码、`Makefile.config` 修改（`WITH_PYTHON_LAYER`、cuDNN v5.1 问题、`OPENCV_VERSION := 3`）、编译 Cython 模块、`tools/demo.py` 报错 | `py-faster-rcnn-InstallUserManual.doc` |
| [05 Caffe 层目录](docs/05-caffe-layers.md) | Layer Catalogue：Data / Vision / Recurrent / Common / Normalization / Activation-Neuron / Utility / Loss 各层清单 | `caffe layters.pdf` |
| [06 Caffe Solver 与模型优化](docs/06-solver-types.md) | Solver 职责、SGD / AdaDelta / AdaGrad / Adam / NAG / RMSprop、学习率与动量经验规则、Scaffolding（Net initialization / Loss / Completion）、参数更新、快照与恢复 | `solver types.pdf` |
| [07 SGD 与 GD 对比](docs/07-sgd-vs-gd.md) | 随机梯度下降与梯度下降在样本使用、耗时、收敛上的差异 | `SGD_GD.txt` |
| [08 NumPy `.npy` 文件格式](docs/08-numpy-npy.md) | npy 格式文档、生成示例、字节布局与头部字段说明 | `npy.txt` |
| [09 solver.prototxt 参数说明](docs/09-solver-prototxt.md) | `base_lr`、`lr_policy`、`stepsize`、`max_iter`、`momentum`、`weight_decay`、`snapshot_prefix` 等参数含义与取值一览 | `solver.prototxt.txt` |
| [10 Ubuntu 文件系统层次结构](docs/10-ubuntu-filesystem-hierarchy.md) | `man hier` 全文整理：目录树总览与逐目录说明 | `ubuntu_filesystem_hierarchy.txt` |
| [11 weight_decay 与 decay_mult](docs/11-weight-decay-decay-mult.md) | 权重衰减、正则化类型（L1/L2）、逐层 `decay_mult` 的作用范围 | `weight_decay decay_multi.txt` |

## 从裸机到跑通 DIGITS 的速查流程

1. 安装系统：用 Yakkety Yak 16.10 *Minimal Boot CD*（有线网络），避开 01 中记录的失败镜像 → [01](docs/01-caffe-install.md)
2. 装 NVIDIA 驱动：Additional Drivers → NVIDIA binary driver → 重启 → [01](docs/01-caffe-install.md#install-titan-x-driver安装-titan-x-驱动)
3. 装 CUDA 8.0：下载 1.9GB deb → `dpkg -i` → `apt-get update` → `apt-get install cuda` → [01](docs/01-caffe-install.md#install-cuda)
4. 编译 CUDA samples 排错：gcc 版本过高（注释 `host_config.h` 的 `#error` 或装 gcc-5）、`cannot find -lnvcuvid` → [01](docs/01-caffe-install.md#compile-cuda-samples编译-cuda-samples)
5. 用 `deviceQuery` 确认 CUDA 可用 → [01](docs/01-caffe-install.md#run-devicequery-to-make-sure-cuda-works)
6. 装 cuDNN 5.1.10（dev + doc + runtime 三个 deb），用 `mnistCUDNN` 样例验证 → [01](docs/01-caffe-install.md#install-cudnn安装-cudnn)
7. 装 Caffe 依赖库（protobuf / leveldb / snappy / opencv / hdf5 / boost / atlas / gflags / glog / lmdb） → [01](docs/01-caffe-install.md#caffe-prerequisitecaffe-依赖库)
8. 编译 NVcaffe（cmake + `make -j12 all` + `make pycaffe`）与 BVLC Caffe（`Makefile.config` 改动：`USE_CUDNN := 1`、HDF5 路径、`WITH_PYTHON_LAYER := 1`、`OPENCV_VERSION := 3`） → [01](docs/01-caffe-install.md#compile-caffe编译-caffe)
9. 跑 mnist 例子并用 pycaffe 做预测 → [01](docs/01-caffe-install.md#测试-mnist-例子)
10. 装并启动 DIGITS（`digits-devserver`，浏览器打开 `http://localhost:5000`），创建数据集、训练模型 → [02](docs/02-digits.md)
11. 在已有模型上微调（flickr_style / ROP / GoogLeNet，含 deploy.prototxt 与 TRAIN/TEST 命令） → [03](docs/03-finetune-model.md)
12. 需要做目标检测时再装 py-faster-rcnn → [04](docs/04-py-faster-rcnn-install.md)

## 目录结构

```text
.
├── README.md
├── docs/                  # 整理后的 Markdown 手册（01–11）
└── source/                # 原始文档：.doc / .pdf / .txt（未改动）
```

## 原始文档与配套安装包

`source/` 中保留了全部原始文件：

- `CaffeInstallUserManual.doc`、`CaffeInstallUserManual(1).doc`、`CaffeInstallUserManual(2).doc`
- `DIGITS.doc`
- `FinetuneModel.doc`、`FinetuneModel(1).doc`、`FinetuneModel(2).doc`
- `py-faster-rcnn-InstallUserManual.doc`
- `caffe layters.pdf`、`solver types.pdf`
- `SGD_GD.txt`、`npy.txt`、`solver.prototxt.txt`、`ubuntu_filesystem_hierarchy.txt`、`weight_decay decay_multi.txt`

同一主题的多份变体（`CaffeInstallUserManual`、`FinetuneModel`）在 Markdown 中已合并为一份，差异处在正文以注释标注；各变体原文仍保留在 `source/` 中以便核对。

安装笔记配套的**大型二进制安装包未收录在本仓库**（GitHub 单文件上限 100MB），需要时请从官方渠道获取：

| 文件 | 大小 |
| --- | --- |
| `ubuntu-16.04.2-desktop-amd64.iso` | 1.45 GB |
| `ubuntu-16.10-desktop-amd64.iso` | 1.48 GB |
| `UbuntuKylin-1604-ukui-amd64.iso` | 1.85 GB |
| `ubuntukylin-16.04.2-enhanced-amd64.iso` | 1.98 GB |
| `YakketyYak.iso` | 55 MB |
| `TrustyTahr.iso` | 31 MB |
| `cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb` | 1.78 GB |
| `cuda-repo-ubuntu1604_8.0.61-1_amd64.deb` | 2.6 KB |
| `libcudnn5_5.1.10-1+cuda8.0_amd64.deb`（runtime） | 39 MB |
| `libcudnn5-dev_5.1.10-1+cuda8.0_amd64.deb` | 32 MB |
| `libcudnn5-doc_5.1.10-1+cuda8.0_amd64.deb` | 4.4 MB |
| `graphviz-working.tar.gz`、`meld-3.16.4.tar.xz`、`sogoupinyin_2.1.0.0082_amd64.deb`、`teamviewer_12.0.76279_i386.deb` | 25–47 MB |

## 说明

- 文档记录的是 **2017 年** 的环境与版本，其中的下载链接、软件源与依赖包名可能已经失效或被取代（例如 CUDA 8.0 / cuDNN 5 均已停止更新）；保留原样是为了可追溯。
- 笔记中的路径、用户名为原作者机器上的实际值（如 `/home/zys/`），复用时请按自己的环境替换。
- 报错与解决方案按“原样复现的现场记录”整理，未做验证性重跑。
