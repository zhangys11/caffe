# weight_decay 与 decay_mult（weight decay and per-layer decay multiplier）

> 来源文件：`weight_decay decay_multi.txt`（2017 年前后整理的安装笔记）

原文分为两部分：`=== weight_decay ===` 说明全局正则化项，`=== decay_multi ===` 说明逐层的权重衰减倍率。

注意：原文章节标题写作 `decay_multi`，文件名中也写作 `decay_multi`，而 prototxt 里的选项名写作 `decay_mult`。下面全部按原文照录，未做统一。

## weight_decay

The weight_decay meta parameter govern the regularization term of the neural net.

During training a regularization term is added to the network's loss to compute the backprop gradient. The weight_decay value determines how dominant this regularization term will be in the gradient computation.

As a rule of thumb, the more training examples you have, the weaker this term should be. The more parameters you have (i.e., deeper net, larger filters, larger InnerProduct layers etc.) the higher this term should be.

Caffe also allows you to choose between L2 regularization (default, sum of weight squares) and L1 regularization (sum of weight absolute values), by setting

```prototxt
regularization_type: "L1"
```

However, since in most cases weights are small numbers (i.e., -1<w<1), the L2 norm of the weights is significantly smaller than their L1 norm. Thus, if you choose to use regularization_type: "L1" you might need to tune weight_decay to a significantly smaller value.

(There is also a L0 norm, which corresponds to the total number of zero-valued weights. L0 and L1 tends to get a sparse solution, i.e. only keep principle features).

While learning rate may (and usually does) change during training, the regularization weight is fixed throughout.

### 正则化类型对照

下表内容全部取自上面原文的表述。

| 正则化类型 | 原文描述 | 原文中的备注 |
| --- | --- | --- |
| L2 | sum of weight squares | default |
| L1 | sum of weight absolute values | 通过 `regularization_type: "L1"` 选择 |
| L0 | the total number of zero-valued weights | L0 and L1 tends to get a sparse solution, i.e. only keep principle features |

## decay_multi

In the solver file, we can set a global regularization loss using the weight_decay and regularization_type options.

In many cases we want different weight decay rates for different layers. This can be done by setting the decay_mult option for each layer in the network definition file, where decay_mult is the multiplier on the global weight decay rate, so the actual weight decay rate applied for one layer is decay_mult*weight_decay.

原文给出的关系式：

$$\text{actual weight decay rate} = \text{decay\_mult} * \text{weight\_decay}$$

For example, the following defines a convolutional layer with NO weight decay regardless of the options in the solver file.

```prototxt
layer {
  name: "Convolution1"
  type: "Convolution"
  bottom: "data"
  top: "Convolution1"
  param {
    decay_mult: 0
  }
  convolution_param {
    num_output: 32
    pad: 0
    kernel_size: 3
    stride: 1
    weight_filler {
      type: "xavier"
    }
  }
}
```

## 选项作用范围对照

| 选项 | 原文所述作用 | 原文所述设置位置 |
| --- | --- | --- |
| `weight_decay` | govern the regularization term of the neural net；determines how dominant this regularization term will be in the gradient computation | solver 文件（global regularization loss） |
| `regularization_type` | 在 L2 与 L1 正则化之间选择 | solver 文件（与 weight_decay 一起设置全局正则化） |
| `decay_mult` | the multiplier on the global weight decay rate；逐层实际生效的权重衰减率为 decay_mult*weight_decay | 网络定义文件中每一层（for each layer in the network definition file） |
