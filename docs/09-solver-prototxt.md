# solver.prototxt 参数说明（Caffe solver configuration）

> 来源文件：`solver.prototxt.txt`（2017 年前后整理的安装笔记）

The solver.prototxt is a configuration file used to tell caffe how you want the network trained.

原文为逐条参数说明文档。需要注意的是：提取出的文本里只有参数名与说明文字，并没有给出可直接使用的完整 prototxt 实例，因此下文只还原原文写出的参数名、说明与可选项，不补写任何原文之外的取值或示例配置。

原文中每个参数名前面带一个 `>` 标记（如 `> base_lr`），下面把这一标记转成小节标题，参数名本身与说明文字保持原样。

## 参数名清单（按原文顺序）

原文中的参数名按出现顺序列在下面。此处仅是参数名本身，不是一份可运行的配置；原文对 `snapshot_prefix` 与 `net` 的写法带冒号，这里保持原样。

```prototxt
base_lr
lr_policy
gamma
stepsize
stepvalue
max_iter
momentum
weight_decay
solver_mode
snapshot
snapshot_prefix:
net:
test_iter
test_interval
display
type
```

## 参数说明（Parameters）

### base_lr

This parameter indicates the base (beginning) learning rate of the network. The value is a real number (floating point).

### lr_policy

This parameter indicates how the learning rate should change over time. This value is a quoted string.

Options include:

- "step" - drop the learning rate in step sizes indicated by the gamma parameter.
- "multistep" - drop the learning rate in step size indicated by the gamma at each specified stepvalue.
- "fixed" - the learning rate does not change.
- "exp" - gamma^iteration
- "poly" - <!-- 原文此处疑似缺字 -->
- "sigmoid" - <!-- 原文此处疑似缺字 -->

### gamma

This parameter indicates how much the learning rate should change every time we reach the next "step." The value is a real number, and can be thought of as multiplying the current learning rate by said number to gain a new learning rate.

### stepsize

This parameter indicates how often (at some iteration count) that we should move onto the next "step" of training. This value is a positive integer.

### stepvalue

This parameter indicates one of potentially many iteration counts that we should move onto the next "step" of training. This value is a positive integer. There are often more than one of these parameters present, each one indicated the next step iteration.

### max_iter

This parameter indicates when the network should stop training. The value is an integer indicate which iteration should be the last.

### momentum

This parameter indicates how much of the previous weight will be retained in the new calculation. This value is a real fraction.

### weight_decay

This parameter indicates the factor of (regularization) penalization of large weights. This value is a often a real fraction.

### solver_mode

This parameter indicates which mode will be used in solving the network.

Options include:

- CPU
- GPU

### snapshot

This parameter indicates how often caffe should output a model and solverstate. This value is a positive integer.

### snapshot_prefix:

This parameter indicates how a snapshot output's model and solverstate's name should be prefixed. This value is a double quoted string.

（原文中该参数名紧接在 `snapshot` 的说明之后单独成行，未带其他参数名前面的 `>` 标记。）

### net:

This parameter indicates the location of the network to be trained (path to prototxt). This value is a double quoted string.

### test_iter

This parameter indicates how many test iterations should occur per test_interval. This value is a positive integer.

### test_interval

This parameter indicates how often the test phase of the network will be executed.

### display

This parameter indicates how often caffe should output results to the screen. This value is a positive integer and specifies an iteration count.

### type

This parameter indicates the back propagation algorithm used to train the network. This value is a quoted string.

Options include:

- Stochastic Gradient Descent "SGD"
- AdaDelta "AdaDelta"
- Adaptive Gradient "AdaGrad"
- Adam "Adam"
- Nesterov’s Accelerated Gradient "Nesterov"
- RMSprop "RMSProp"

## 参数与取值一览

表中取值/类型一栏直接取自原文的表述，未做补充。

| 参数 | 原文所述取值或类型 |
| --- | --- |
| `base_lr` | a real number (floating point) |
| `lr_policy` | a quoted string；原文列出可选值 "step"、"multistep"、"fixed"、"exp"、"poly"、"sigmoid" |
| `gamma` | a real number |
| `stepsize` | a positive integer |
| `stepvalue` | a positive integer；原文说明通常会有多个该参数 |
| `max_iter` | an integer（原文：indicate which iteration should be the last） |
| `momentum` | a real fraction |
| `weight_decay` | 原文：is a often a real fraction |
| `solver_mode` | 原文列出 CPU、GPU |
| `snapshot` | a positive integer |
| `snapshot_prefix:` | a double quoted string |
| `net:` | a double quoted string（原文：path to prototxt） |
| `test_iter` | a positive integer |
| `test_interval` | 原文未给出取值或类型说明 |
| `display` | a positive integer and specifies an iteration count |
| `type` | a quoted string；原文列出 "SGD"、"AdaDelta"、"AdaGrad"、"Adam"、"Nesterov"、"RMSProp" |
