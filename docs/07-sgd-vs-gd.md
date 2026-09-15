# SGD 与 GD 的对比（Stochastic Gradient Descent / Gradient Descent）

> 来源文件：`SGD_GD.txt`（2017 年前后整理的安装笔记）

原文为一段英文说明，按 GD（gradient descent）与 SGD（stochastic gradient descent）的差别逐段展开，下面保持原文顺序与英文原文。

## 共同点

In both gradient descent (GD) and stochastic gradient descent (SGD), you update a set of parameters in an iterative manner to minimize an error function.

## 每次参数更新所使用的样本

While in GD, you have to run through ALL the samples in your training set to do a single update for a parameter in a particular iteration, in SGD, on the other hand, you use ONLY ONE or SUBSET of training sample from your training set to do the update for a parameter in a particular iteration. If you use SUBSET, it is called Minibatch Stochastic gradient Descent.

## 训练样本数量对耗时的影响

Thus, if the number of training samples are large, in fact very large, then using gradient descent may take too long because in every iteration when you are updating the values of the parameters, you are running through the complete training set. On the other hand, using SGD will be faster because you use only one training sample and it starts improving itself right away from the first sample.

## 收敛速度与误差函数的最小化程度

SGD often converges much faster compared to GD but the error function is not as well minimized as in the case of GD. Often in most cases, the close approximation that you get in SGD for the parameter values are enough because they reach the optimal values and keep oscillating there.

## 两种方法对照

下表把原文中的对比表述集中列出，措辞取自原文。

| 对比项 | GD（gradient descent） | SGD（stochastic gradient descent） |
| --- | --- | --- |
| 单次参数更新所使用的样本 | ALL the samples in your training set | ONLY ONE or SUBSET of training sample |
| 使用 SUBSET 时的名称 | —— | Minibatch Stochastic gradient Descent |
| 训练样本数量很大时的耗时 | may take too long：每一次迭代更新参数都要跑完整个训练集 | faster：只用一个训练样本，从第一个样本起就开始改善 |
| 收敛速度 | 相比 SGD 更慢（原文以 SGD 为准作比较） | often converges much faster |
| 误差函数的最小化程度 | 最小化得更充分（原文以 GD 为准作比较） | not as well minimized as in the case of GD；但得到的参数近似值通常够用，会到达最优值并在该处持续震荡 |
