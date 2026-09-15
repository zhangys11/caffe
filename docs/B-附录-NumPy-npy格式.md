# 附录 B：NumPy .npy 文件格式（NumPy .npy File Format）

Caffe 与 NumPy 的 `.npy` 文件直接相关：Caffe 自带的 ImageNet 均值文件 `python/caffe/imagenet/ilsvrc_2012_mean.npy` 就是这种格式，pycaffe 的预测示例中用 `numpy.load(caffe_root +'python/caffe/imagenet/ilsvrc_2012_mean.npy').mean(1).mean(1)` 把它读进来，交给 `caffe.Classifier` 作为预处理用的均值。数据准备脚本也常用 NumPy 来读写 `.npy` 文件。本附录给出 npy 格式的文档出处、一段生成示例代码、生成文件的逐字节布局，以及头部各字段的含义，便于把这几个环节里出现的 npy 文件与字节对照起来看。本附录只讲文件格式本身，不涉及 Caffe 的安装、编译与训练流程。

## B.1 格式文档（Format Documentation）

The npy file format is documented in numpy's https://github.com/numpy/numpy/blob/master/doc/neps/npy-format.rst.

> 译：npy 文件格式在 numpy 的 https://github.com/numpy/numpy/blob/master/doc/neps/npy-format.rst 中有说明。

这一句给出的是格式定义的出处，需要更完整的说明时应查阅该地址上的文档。本附录余下的内容，只针对下面这段示例代码所生成的那一个 `.npy` 文件。

## B.2 生成示例（Generation Example）

For instance, the code

> 译：例如，下面这段代码

```python
>>> dt=numpy.dtype([('outer','(3,)<i4'),
...                 ('outer2',[('inner','(10,)<i4'),('inner2','f8')])])
>>> a=numpy.array([((1,2,3),((10,11,12,13,14,15,16,17,18,19),3.14)),
...                ((4,5,6),((-1,-2,-3,-4,-5,-6,-7,-8,-9,-20),6.28))],dt)
>>> numpy.save('1.npy', a)
```

results in the file:

> 译：会生成如下文件：

这段代码先用 `numpy.dtype` 定义复合 dtype：字段 `outer` 的类型写作 `'(3,)<i4'`，字段 `outer2` 中又嵌套了 `inner`（`'(10,)<i4'`）与 `inner2`（`'f8'`）两个子字段；然后用 `numpy.array` 按该 dtype 构造两行数据；最后用 `numpy.save` 存入 `1.npy`。紧接着的两节给出这个 `1.npy` 的字节布局与头部字段解释。

## B.3 生成文件的字节布局（Byte Layout of the Generated File）

下面照录 `1.npy` 的全部字节：左侧是十六进制字节，右侧是原文给出的标注。

```text
93 4E 55 4D 50 59                      magic ("\x93NUMPY")
01                                     major version (1)
00                                     minor version (0)

96 00                                  HEADER_LEN (0x0096 = 150)
7B 27 64 65 73 63 72 27 
3A 20 5B 28 27 6F 75 74 
65 72 27 2C 20 27 3C 69 
34 27 2C 20 28 33 2C 29 
29 2C 20 28 27 6F 75 74 
65 72 32 27 2C 20 5B 28 
27 69 6E 6E 65 72 27 2C 
20 27 3C 69 34 27 2C 20 
28 31 30 2C 29 29 2C 20 
28 27 69 6E 6E 65 72 32                Header, describing the data structure
27 2C 20 27 3C 66 38 27                "{'descr': [('outer', '<i4', (3,)),
29 5D 29 5D 2C 20 27 66                            ('outer2', [
6F 72 74 72 61 6E 5F 6F                               ('inner', '<i4', (10,)), 
72 64 65 72 27 3A 20 46                               ('inner2', '<f8')]
61 6C 73 65 2C 20 27 73                            )],
68 61 70 65 27 3A 20 28                  'fortran_order': False,
32 2C 29 2C 20 7D 20 20                  'shape': (2,), }"
20 20 20 20 20 20 20 20 
20 20 20 20 20 0A 

01 00 00 00 02 00 00 00 03 00 00 00    (1,2,3)
0A 00 00 00 0B 00 00 00 0C 00 00 00
0D 00 00 00 0E 00 00 00 0F 00 00 00
10 00 00 00 11 00 00 00 12 00 00 00
13 00 00 00                            (10,11,12,13,14,15,16,17,18,19)
1F 85 EB 51 B8 1E 09 40                3.14

04 00 00 00 05 00 00 00 06 00 00 00    (4,5,6)
FF FF FF FF FE FF FF FF FD FF FF FF
FC FF FF FF FB FF FF FF FA FF FF FF
F9 FF FF FF F8 FF FF FF F7 FF FF FF 
EC FF FF FF                            (-1,-2,-3,-4,-5,-6,-7,-8,-9,-20)
1F 85 EB 51 B8 1E 19 40                6.28
```

从清单开头可以看到：先是 6 字节 magic 与 1 字节主版本号、1 字节次版本号，随后是 2 字节的 `HEADER_LEN`（原文标注为 `0x0096 = 150`）；再往后的 150 字节是头部，原文把头部文本逐行印在右侧，其中出现 `'descr'`、`'fortran_order'`、`'shape'` 三个键，头部末尾是一串 `20` 字节与一个 `0A` 字节。头部之后的字节即数据区，按原文标注依次对应第一条记录的 `(1,2,3)`、`(10,11,12,13,14,15,16,17,18,19)`、`3.14`，以及第二条记录的 `(4,5,6)`、`(-1,-2,-3,-4,-5,-6,-7,-8,-9,-20)`、`6.28`。

## B.4 头部字段说明（Header Field Reference）

下表按原文右侧标注整理，字节值取自上方字节清单。

| 字段 | 英文原文说明 | 中文译注 |
| --- | --- | --- |
| magic | magic ("\x93NUMPY") | 文件开头 6 字节的 magic，原文标注为 `"\x93NUMPY"`，对应上方清单的 `93 4E 55 4D 50 59` |
| major version | major version (1) | 主版本号，取值为 1，对应 `01` |
| minor version | minor version (0) | 次版本号，取值为 0，对应 `00` |
| HEADER_LEN | HEADER_LEN (0x0096 = 150) | 头部长度字段，原文标注 `0x0096 = 150`，对应 `96 00` |
| Header | Header, describing the data structure | 头部，用于描述数据结构；其后 150 字节，原文逐行给出，即 `HEADER_LEN` 之后的 150 字节 |
| Header 内容 | `{'descr': [('outer', '<i4', (3,)), ('outer2', [('inner', '<i4', (10,)), ('inner2', '<f8')])], 'fortran_order': False, 'shape': (2,), }` | 头部文本的内容，见上方清单右侧引号内文本（原文按字节逐行对齐给出），这里合成一行 |
| Header 末尾填充 | `20 20 20 20 20 20 20 20` / `20 20 20 20 20 0A` | 头部末尾的字节，原文未加标注 |
| 数据区（第 1 条记录） | (1,2,3) / (10,11,12,13,14,15,16,17,18,19) / 3.14 | 头部之后的第 1 条记录，原文按此顺序标注三个值；其起始字节为 `01 00 00 00 02 00 00 00 03 00 00 00`（完整字节见上方清单） |
| 数据区（第 2 条记录） | (4,5,6) / (-1,-2,-3,-4,-5,-6,-7,-8,-9,-20) / 6.28 | 头部之后的第 2 条记录，原文按此顺序标注三个值；其起始字节为 `04 00 00 00 05 00 00 00 06 00 00 00`（完整字节见上方清单） |

表中前四行对应字节清单最上面的三行：magic、主版本号、次版本号各占一行，`HEADER_LEN` 紧随其后；`Header` 与 `Header 内容` 说的是同一个头部，只是原文把头部文本逐行对齐印在字节清单右侧。
