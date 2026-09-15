# npy 文件格式（NumPy `.npy` file format）

> 来源文件：`npy.txt`（2017 年前后整理的安装笔记）

## 格式文档

The npy file format is documented in numpy's https://github.com/numpy/numpy/blob/master/doc/neps/npy-format.rst.

## 生成示例

For instance, the code

```python
>>> dt=numpy.dtype([('outer','(3,)<i4'),
...                 ('outer2',[('inner','(10,)<i4'),('inner2','f8')])])
>>> a=numpy.array([((1,2,3),((10,11,12,13,14,15,16,17,18,19),3.14)),
...                ((4,5,6),((-1,-2,-3,-4,-5,-6,-7,-8,-9,-20),6.28))],dt)
>>> numpy.save('1.npy', a)
```

results in the file:

## 生成文件的字节布局

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

## 头部字段说明

下表按原文右侧标注整理，字节值取自上方字节清单。

| 字段 | 原文给出的字节 | 原文标注 |
| --- | --- | --- |
| magic | `93 4E 55 4D 50 59` | magic ("\x93NUMPY") |
| major version | `01` | major version (1) |
| minor version | `00` | minor version (0) |
| HEADER_LEN | `96 00` | HEADER_LEN (0x0096 = 150) |
| Header | 其后 150 字节，原文逐行给出 | Header, describing the data structure |
| Header 内容 | 见上方清单右侧引号内文本 | `{'descr': [('outer', '<i4', (3,)), ('outer2', [('inner', '<i4', (10,)), ('inner2', '<f8')])], 'fortran_order': False, 'shape': (2,), }`（原文按字节逐行对齐给出） |
| Header 末尾填充 | `20 20 20 20 20 20 20 20` / `20 20 20 20 20 0A` | 原文未加标注 |
| 数据区（第 1 条记录） | `01 00 00 00 02 00 00 00 03 00 00 00`（完整字节见上方清单） | (1,2,3) / (10,11,12,13,14,15,16,17,18,19) / 3.14 |
| 数据区（第 2 条记录） | `04 00 00 00 05 00 00 00 06 00 00 00`（完整字节见上方清单） | (4,5,6) / (-1,-2,-3,-4,-5,-6,-7,-8,-9,-20) / 6.28 |
