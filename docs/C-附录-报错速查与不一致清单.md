# 附录 C：报错速查与跨笔记不一致清单（Troubleshooting Index & Known Discrepancies）

本附录不重复各篇正文里的排查过程，只做两件事：把散落在四篇里的报错处理汇成一张索引表，方便按现象直接跳到处理位置；把两处在正文里没有归属的运行期报错（py-faster-rcnn 与 DIGITS 数据层）完整收录在这里。最后列出这批笔记中前后不一致的地方，并给出一句话说明它们各自在哪个章节被讨论过——正文里保留了两处笔记的原始口径，本附录只做索引，不替原文下结论。

---

## C.1 报错索引（Error Index）

| 现象 / 报错关键词 | 出现环节 | 处理位置 |
| --- | --- | --- |
| `/sbin/ldconfig.real: libEGL.so.1 is not a symbolic link` | 装完 CUDA 后 apt 触发 ldconfig | 《环境搭建篇》1.2 |
| `gcc version not supported. The system uses gcc 6` | 编译 CUDA samples | 《环境搭建篇》1.3 |
| `/usr/bin/ld: cannot find -lnvcuvid` | 编译 CUDA samples | 《环境搭建篇》1.3 |
| `cannot find -lhdf5_hl` / `-lhdf5` | 编译 BVLC Caffe | 《环境搭建篇》2.2 |
| `numpy/arrayobject.h: No such file or directory` | 编译 pycaffe | 《环境搭建篇》2.4 |
| `"…/NVcaffe/" from CAFFE_ROOT does not point to a valid installation` | 启动 `digits-devserver` | 《环境搭建篇》3.1 |
| protobuf 版本导致 DIGITS 启动失败（`SOLUTION: reinstall protobuf`） | 启动 `digits-devserver` | 《环境搭建篇》3.1 |
| `ImportError: No module named skimage.io` | 运行 py-faster-rcnn 演示 | 本附录 C.2 |
| `ImportError: No module named gpu_nms` | 运行 py-faster-rcnn 演示 | 本附录 C.2 |
| `data_layer.cpp:73] Restarting data prefetching from start.` | Caffe / DIGITS 训练时数据层输出 | 本附录 C.3 |
| `'google.protobuf.pyext._message.RepeatedScalarConta' object has no attribute '_values'` | DIGITS 的 visualize model | 《使用与训练篇》4.2（含 `draw.py` 的修改） |

> 上表的「处理位置」是重构后的章节编号；若你手上的版本页眉号不同，按篇名 + 节标题检索即可。

---

## C.2 py-faster-rcnn 运行期报错（py-faster-rcnn Runtime Errors）

演示脚本跑起来之前的编译与环境准备见《环境搭建篇》第 3 章；下面两条是运行阶段才会遇到的：

```text
Error:  ImportError: No module named skimage.io
Solution: sudo -H pip install scikit-imag //this also install lots of dependencies
```

<!-- 原文此处疑似缺字 -->

> 译：报错：`ImportError: No module named skimage.io`；解决办法：`sudo -H pip install scikit-imag`（原文注释：这一步也会顺带装上一堆依赖）。原文的包名写作 `scikit-imag`，疑似被截断，代码块按原样保留、未作修改。

```text
Error: ImportError: No module named gpu_nms
Solution: run make in lib folder
```

> 译：报错：`ImportError: No module named gpu_nms`；解决办法：在 `lib` 目录下执行 `make`。这一步即《环境搭建篇》3.5 编译 Cython 扩展所生成的模块，编译没做或没成功时就会在看到界面前先报这个错。

---

## C.3 数据层输出过多（Data Layer Prefetch Restart）

If output two many:

> 译：如果（训练日志）输出太多：

```text
data_layer.cpp:73] Restarting data prefetching from start.
```

Means:

> 译：含义如下：

- You gave the wrong .txt file to data layer
- The format of the .txt file is not as expected by Caffe
- Very few number of data is present in the file.

> 译：给数据层（data layer）的 `.txt` 文件不对；`.txt` 文件的格式不是 Caffe 期望的格式；文件里的数据量非常少。

这条输出对应的标签文件格式问题，在《使用与训练篇》2.2 与 4.2 也各提示过一次（标签文件的分隔符假设与实际生成方式不一致）。

---

## C.4 跨笔记不一致清单（Known Discrepancies Between Notes）

同一件事在这批笔记的不同位置写法不同，重构时没有替原文统一，而是把两处口径都保留在正文中并加了提示。下表只做索引：

| # | 不一致项 | 两种口径 | 正文处理位置 |
| --- | --- | --- | --- |
| 1 | cuDNN 5.1 能否编译 | 一处记录本机 CUDA 8.0 + cuDNN 5.1 编译 Caffe 成功并跑通 `mnistCUDNN`；另一处写「cuDNN v5.1 cannot compile, disable cuDNN or downgrade to cuDNN v4」 | 《环境搭建篇》1.4 与 3.4（已按项目限定：py-faster-rcnn 自带的旧源码） |
| 2 | CUDA 8.0 能否编译 Caffe | 一处成功编译 BVLC 与 NV 两版；另一处写自带 Caffe 源码「cannot compile with CUDA 8.0」 | 《环境搭建篇》2.2 与 3.4 |
| 3 | `CAFFE_ROOT` 指向 | `/home/zys/NVcaffe/`（注明 should use NV flavor）与 `/home/zys/caffe-master/` | 《环境搭建篇》0.3 与 3.1 |
| 4 | `make` 并行度 | `-j6` 与 `-j12`，两处注释都写「6 cores」 | 《环境搭建篇》2.2（全篇只解释一次） |
| 5 | 数据盘路径 | `/mnt/seagate`、`/media/zys/Seagate Backup Plus Drive`、`~/seagate` | 《环境搭建篇》4.1 |
| 6 | 驱动小版本 | `libEGL.so.375.66` 与 `libnvcuvid.so.375.26` | 《环境搭建篇》1.2 |
| 7 | CPU / GPU 模式 | `caffe.set_mode_gpu()` 写在 `#cpu模式` 注释之下 | 《使用与训练篇》2.1 与 4.2 |
| 8 | 标签文件分隔符 | DIGITS 生成的 `labels.txt` 为每行一个数字；预测脚本按 `delimiter='\t'` 解析 | 《使用与训练篇》2.2 与 4.2、本附录 C.3 |
| 9 | 逐层衰减倍率的写法 | `decay_multi`（小节标题）与 `decay_mult`（prototxt 选项名） | 《调优与原理篇》第 4 章 |

---

## C.5 转录约定（Transcription Notes）

这批笔记最早的载体是 Word 文档与 PDF 讲义，正文里保留了几处提取过程中留下的痕迹，含义如下：

| 标记 / 现象 | 含义 |
| --- | --- |
| `<!-- 原文此处疑似缺字 -->` | 原文在此处断句或符号丢失，正文照录并标注，未作猜测补全 |
| 公式以 `text` 代码块给出、符号看起来不完整 | PDF 提取时数学符号丢失（例如分式被压成一行、`RMS` 变成 `MS`），按提取原文保留，旁边附一句中文说明该式在英文原文里表达什么 |
| 命令块内的 `//` 注释、`momemtum`、`mum_output`、`scikit-imag` 等 | 原文如此，未做修改 |
