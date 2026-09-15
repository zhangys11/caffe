# Caffe 深度学习环境手册（Ubuntu 16.10 + CUDA 8.0 + cuDNN 5 + DIGITS）

这是一份 2017 年前后在 **Ubuntu 16.10（Yakkety Yak）** 服务器上从裸机搭起 Caffe 深度学习环境的完整手册：装系统与驱动、装 CUDA 8.0 与 cuDNN 5.1、编译 BVLC Caffe 与 NVcaffe、部署 DIGITS、跑 MNIST、在自己数据上做微调，以及过程中遇到的全部报错与解决办法。

内容按「四篇 + 三个附录」重新组织，**逐段中英对照**：英文是笔记里的原始表述，紧跟的引用块是以「译：」开头的中文翻译。命令、路径、参数名、报错文本一律按原始记录保留，不做改写。

## 环境基线

| 项目 | 配置 |
| --- | --- |
| CPU / 主板 | Intel i7 6800K / X99 |
| 内存 | 32GB DDR4 |
| GPU | NVIDIA Titan X Pascal 12GB |
| 操作系统 | Ubuntu 16.10（Yakkety Yak Minimal Boot CD，需有线网络） |
| CUDA | 8.0（仓库包 8.0.61） |
| cuDNN | 5.1.10 |
| 主机编译器 | GCC 5.4.1（CUDA 8.0 不接受 GCC 6） |
| Python | 2.7 |
| OpenCV | 3（脚本中的目录为 opencv-3.2.0） |
| 框架 | BVLC 官方 Caffe（目录 `caffe-master`）与 NVIDIA NVcaffe 两套并存 |
| 可视化 | NVIDIA DIGITS（Web 界面 `http://localhost:5000`） |

两套 Caffe 的分工：**DIGITS 训练用 NVcaffe**，**pycaffe 预测与微调实验用 BVLC 官方 Caffe**（详见[篇一](docs/1-环境搭建篇.md) 0.3）。

## 怎么读

- **第一次从裸机开始搭**：按 [篇一 环境搭建](docs/1-环境搭建篇.md) 的顺序走完，再用 [篇二 使用与训练](docs/2-使用与训练篇.md) 跑通第一个例子。
- **环境已经有了，只想训练/微调**：直接看 [篇二 使用与训练](docs/2-使用与训练篇.md)：MNIST 入门 → DIGITS 流程 → 三个微调实例。
- **想查参数、脚本模板、编译开关**：[篇四 参考手册](docs/4-参考手册篇.md)：层目录、solver 参数、预测脚本与配置模板、编译开关表。
- **遇到报错**：[附录 C 报错速查](docs/C-附录-报错速查与不一致清单.md) 按现象检索，里面还有一份「跨笔记不一致清单」。
- **想理解为什么要这么调**：[篇三 调优与原理](docs/3-调优与原理篇.md)：solver 训练循环、六种优化方法、学习率与动量经验规则、权重衰减与正则化。

## 文档索引

| 文档 | 内容 | 适合 |
| --- | --- | --- |
| [篇一 环境搭建（Environment Setup）](docs/1-环境搭建篇.md) | 0 基线与选型 · 1 GPU 运行环境（驱动 / CUDA 8.0 / CUDA samples 排错 / cuDNN） · 2 编译 Caffe（依赖 / BVLC / NVcaffe / pycaffe） · 3 可选组件（DIGITS、py-faster-rcnn、OpenCV 3、Cython） · 4 数据存储 | 从零搭环境的人 |
| [篇二 使用与训练（Usage & Training）](docs/2-使用与训练篇.md) | 1 第一个例子 MNIST · 2 pycaffe 做预测 · 3 用 DIGITS 训练（数据集准备、界面、MNIST 入门、建数据集与训练） · 4 模型微调（flickr_style、ROP、GoogLeNet、RetCam） · 5 Faster R-CNN 演示 | 要跑训练和微调的人 |
| [篇三 调优与原理（Tuning & Principles）](docs/3-调优与原理篇.md) | 1 Solver 与训练循环 · 2 优化方法（SGD / AdaDelta / AdaGrad / Adam / NAG / RMSProp） · 3 学习率与动量经验规则 · 4 权重衰减与正则化 · 5 参数更新、快照与恢复 · 6 微调超参数取舍 · 7 网络约束与输入尺寸 · 8 性能基准与加速比 | 想弄清取值理由的人 |
| [篇四 参考手册（Reference）](docs/4-参考手册篇.md) | 1–9 Caffe 层目录（数据 / 视觉 / 循环 / 常用 / 归一化 / 激活 / 工具 / 损失） · 10 solver.prototxt 参数速查 · 11 脚本模板 · 12 配置模板 · 13 编译开关速查 | 边做边查的人 |
| [附录 A Ubuntu 文件系统层次结构](docs/A-附录-Ubuntu文件系统层次结构.md) | `man hier` 整理：目录树总览 + 137 条目录说明（中英对照表格） | 想知道库和配置装在哪一层的人 |
| [附录 B NumPy .npy 文件格式](docs/B-附录-NumPy-npy格式.md) | npy 格式出处、生成示例、逐字节布局、头部字段说明 | 要读写 Caffe 均值文件的人 |
| [附录 C 报错速查与不一致清单](docs/C-附录-报错速查与不一致清单.md) | 报错索引表、py-faster-rcnn 与数据层报错、9 条跨笔记不一致、转录约定 | 排错时先翻这里 |

## 阅读约定

- **中英对照**：英文段落为笔记原文，紧跟的 `> 译：` 引用块是中文翻译；列表与表格保持英文并加中文导语或说明列，不逐行对照。
- **命令与报错一字未改**：包括原文里的拼写（`momemtum`、`mum_output`、`scikit-imag`）与行尾空格。
- **不一致处不擅自统一**：同一件事在不同笔记里写法不同时，两处口径都保留并在正文加中文提示，汇总见[附录 C](docs/C-附录-报错速查与不一致清单.md) 的 C.4。
- **残缺处如实标注**：提取过程中断句或丢符号的地方，保留原样并标 `<!-- 原文此处疑似缺字 -->`，旁边用一句中文说明该段原本在说什么，不做猜测补全。
- **公式**：以 `text` 代码块给出的是从 PDF 提取的原始形态，符号可能有丢失；笔记里唯一一条渲染正常的 LaTeX 公式按原样保留。
- **文献编号**：沿用原笔记的分节编号，`[1]`、`[2]` 只在本节内有效，不是全篇连续编号。

## 仓库结构

```text
.
├── README.md
└── docs/
    ├── 1-环境搭建篇.md
    ├── 2-使用与训练篇.md
    ├── 3-调优与原理篇.md
    ├── 4-参考手册篇.md
    ├── A-附录-Ubuntu文件系统层次结构.md
    ├── B-附录-NumPy-npy格式.md
    └── C-附录-报错速查与不一致清单.md
```

## 说明

- 手册记录的是 **2017 年** 的软件版本与操作过程：CUDA 8.0、cuDNN 5、Ubuntu 16.10 均已停止更新，文中的下载链接与软件源可能失效，保留原样是为了可追溯。
- 笔记里的路径与用户名是原作者机器上的实际值（如 `/home/zys/`），复用时按自己的环境替换；DIGITS 与 py-faster-rcnn 的路径变量（`CAFFE_ROOT`、`$FRCN_ROOT`）在各篇对应章节有说明。
- 大型安装包未收录在本仓库（GitHub 单文件上限 100MB），需要时请从官方渠道获取：

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
| `libcudnn5_5.1.10-1+cuda8.0_amd64.deb`（runtime；笔记中另一处列表未将其单列） | 39 MB |
| `libcudnn5-dev_5.1.10-1+cuda8.0_amd64.deb` | 32 MB |
| `libcudnn5-doc_5.1.10-1+cuda8.0_amd64.deb` | 4.4 MB |
| `graphviz-working.tar.gz`、`meld-3.16.4.tar.xz`、`sogoupinyin_2.1.0.0082_amd64.deb`、`teamviewer_12.0.76279_i386.deb` | 25–47 MB |
