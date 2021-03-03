---
layout: AI
title: 一起来学 Einsum，Einops
categories: AI
comments: false
date: 2021-03-03
tags: AI
urlname: learn-einsum-and-einops
---

# 一起来学 Einsum，Einops

## Einsum

### 啥？做啥的？学这玩意儿干啥？

numpy, pytorch, tensorflow中有个函数叫做einsum, 它的来源是Einstein notation, 中文译作爱因斯坦求和约定，这约定是由阿尔伯特·爱因斯坦于1916年提出的。

采用爱因斯坦求和约定，可以使数学表达式显得简洁明快。而将这种简洁记法运用到矩阵运算代码中会带来什么好处呢？

我们来看numpy官方关于einsum的文档是怎么说的，让我们来 ~~抄~~ 引用一下

> Using the Einstein summation convention, many common multi-dimensional, linear algebraic array operations can be represented in a simple fashion. In *implicit* mode [`einsum`](https://numpy.org/doc/stable/reference/generated/numpy.einsum.html#numpy.einsum) computes these values.

许多常见的多维度操作都可以被einsum这一个函数表示，那么有哪些常见操作呢？~~抄之~~ 引用之

> A non-exhaustive list of these operations, which can be computed by [`einsum`](https://numpy.org/doc/stable/reference/generated/numpy.einsum.html#numpy.einsum), is shown below along with examples:
>
> - Trace of an array, [`numpy.trace`](https://numpy.org/doc/stable/reference/generated/numpy.trace.html#numpy.trace).
> - Return a diagonal, [`numpy.diag`](https://numpy.org/doc/stable/reference/generated/numpy.diag.html#numpy.diag).
> - Array axis summations, [`numpy.sum`](https://numpy.org/doc/stable/reference/generated/numpy.sum.html#numpy.sum).
> - Transpositions and permutations, [`numpy.transpose`](https://numpy.org/doc/stable/reference/generated/numpy.transpose.html#numpy.transpose).
> - Matrix multiplication and dot product, [`numpy.matmul`](https://numpy.org/doc/stable/reference/generated/numpy.matmul.html#numpy.matmul) [`numpy.dot`](https://numpy.org/doc/stable/reference/generated/numpy.dot.html#numpy.dot).
> - Vector inner and outer products, [`numpy.inner`](https://numpy.org/doc/stable/reference/generated/numpy.inner.html#numpy.inner) [`numpy.outer`](https://numpy.org/doc/stable/reference/generated/numpy.outer.html#numpy.outer).
> - Broadcasting, element-wise and scalar multiplication, [`numpy.multiply`](https://numpy.org/doc/stable/reference/generated/numpy.multiply.html#numpy.multiply).
> - Tensor contractions, [`numpy.tensordot`](https://numpy.org/doc/stable/reference/generated/numpy.tensordot.html#numpy.tensordot).
> - Chained array operations, in efficient calculation order, [`numpy.einsum_path`](https://numpy.org/doc/stable/reference/generated/numpy.einsum_path.html#numpy.einsum_path).

”奥这样啊，看起来还行“，看你若有所思的样子我就知道你在想什么。

你可能会说这不就是一种语法糖么？相当于原来我用代码抄公式要花5行代码，现在就一行了，但是so what? 只是个小trick罢了。

的确如果只是处理较低的维度矩阵的话，似乎是这样。

但当我们涉及到需要频繁处理复杂的高维度矩阵操作的时候，这个方法的简洁高效性就体现出来了。

让我们来一些复杂点的例子。



首先让我们来看看对一个7维矩阵中的部分维度进行转置的例子

假设我们有一个矩阵$A$，它的形状为$1\times3\times4\times2\times3\times64\times64$，现在希望互换部分维度，转化为$1\times3\times4\times64\times64\times2\times3$，我们来看看两种方法分别做起来是什么样子的:

```python
import numpy as np
a = np.random.randn(1,3,4,2,3,64,64)
legacy_result = a.transpose(0,1,2,5,6,3,4)	# 通常方法
einsum_result = np.einsum("bcijkwh->bciwhjk", a)	# Einsum
print((legacy_result==einsum_result).all())	 # 查看是否等价
#----输出True
```



从上面这个例子中，我们可以看出利用einsum的任意标签特性，我们可以给各个维度赋予一个有含义的标签（比如b代表batch，c代表channel等等）。如此便比比直接使用index操作更加直观可读，不容易出错。

但这也不过如此，einsum的强大之处不只体现在这里。

我们再来看一个例子。



如何用写一个维度为$1\times3\times4\times2\times3\times64\times64$和$1\times3\times4\times(2\times3)^T\times64\times64$的倒数第4，3个维度的矩阵积并得到$1\times3\times4\times2\times2\times64\times64$的结果?

```python
a = np.random.randn(1,3,4,2,3,64,64)
b = np.random.randn(1,3,4,2,3,64,64)
# 传统做法, 先把要乘的转置到最后，完成运算后再转置回来
a1=a.transpose(0,1,2,5,6,3,4) 		# Shape(1, 3, 4, 64, 64, 2, 3)
b1=b.transpose(0,1,2,5,6,4,3)  # Shape(1, 3, 4, 64, 64, 3, 2)
legacy_result = np.matmul(a1,b1) # Shape(1, 3, 4, 64, 64, 2, 2)
legacy_result = legacy_result.transpose(0,1,2,5,6,3,4)  # Shape(1, 3, 4, 2, 2, 64, 64)

# 很麻烦对不对？那如果用einsum来做呢？
einsum_result = np.einsum('...ijhw,...kjhw->...ikhw', a,b) # Shape(1, 3, 4, 2, 2, 64, 64)

print((legacy_result==einsum_result).all())		# 我们来验证一下结果是否一致
# --------输出--------
# True
```

非常amazing是不是？显然einsum在这个例子上体现出了绝对的优越性。

### 不升华一下似乎很难收场(´ο｀*)))~

其实有时候同样的事情换个角度看就会有很大的不同。你可以说这只是一种新的记法，记法嘛，无外乎只是写起来麻烦些和简单些的区别，但是你往往会发现不同的记法往往不仅仅是节省墨水，更可以为你提供更简洁的思维封装单元，同时还能腾出更多的思维时间。

就像集成电路里的芯片，14纳米和5纳米的芯片虽然整体实现的功能完全一样，尺寸听起来似乎也不是差得很多，但它在相同资源空间内带来的不同堆集效应是无法忽视的。

回到我们探讨的问题上，你往往会发现，随着矩阵维度的增高，你的精力会逐渐被思考这个地方该如何转置，那个维度该怎么堆叠这些无关宏观目标的具体细节所分散。

(好吧，听起来丝毫没有说服力~)

在此我只是试图浅薄地理解由Numpy最早引入的einsum函数在具体工程问题上的一些好处，更深层次的东西我就没有能力再去探讨了。

不过说实在的，我目前也只是直觉上觉得学这个东西具有巨大的作用。实质上我也是刚开始学而已，暂时也说不出什么太实质性的感悟。只是我在读[`vit-pytorch`](https://github.com/lucidrains/vit-pytorch/blob/main/vit_pytorch/vit_pytorch.py) 代码的时候，发现它对Transformer的核心实现大量的采用了einsum以及受它启发的einops，感觉非常简洁明快，并且在你理解它的意思后，会发现这种符号记法远远比`np.inner(a,b)`又或是`transpose(2,1,3,2)`来得直观地多。

正是它给我带来的这种直观印象促使了我决定放下手头的工作，好好地把它学一下，我相信它会给我以后的工作和思考带来持续的好处，从这个角度看，我这么做是很值得的。百忙中走一步闲棋，后效如何，还待他日探讨。

（当然我才不会承认还有一部分原因是因为这个方法冠有爱因斯坦的鼎鼎大名，实为炫技装逼之利器也~）

### Reference

[1] [Numpy 文档](https://numpy.org/doc/stable/reference/generated/numpy.einsum.html)

[2] Einstein notation, [Wikipedia](https://en.wikipedia.org/wiki/Einstein_notation)

[3] 爱因斯坦求和约定，[百度词条](https://baike.baidu.com/item/%E7%88%B1%E5%9B%A0%E6%96%AF%E5%9D%A6%E6%B1%82%E5%92%8C%E7%BA%A6%E5%AE%9A/2477100?fr=aladdin)

## Einops

### 这又是啥？(⊙o⊙)学不动了，告辞。

哎别走嘛，来来，我们来看个小视频。（骗你的，其实是个动图）

![官方文档盗过来的一张动图](http://arogozhnikov.github.io/images/einops/einops_video.gif)

不得不说作者还是蛮会的，这个动图简洁明了，相信你看完之后一定会很直观地get到这个项目能实现什么效果：厉害吧？你要不要学？

接着放几个大佬的Tweets评价来背书：快来用呀，大佬们都说好哦~

还支持Numpy，Pytorch，TensorFlow，Jax等诸多框架，哇，满满的都是高级感呀，2.5K个Star一点也不奇怪。

不知道你有没有心动了，反正我是心动了，感觉学完就能和爱因斯坦以及硅谷大佬肩并肩，走上人生巅峰啊~  `^▽^`

（醒醒，你只是为了读懂别人的代码）

那就一起来看看吧~。
（原来哄自己学习也这么辛苦的么）



额，突然不想写了，先酱紫吧，回头再更，拜~(害，还是没哄成)

### Reference

[1]https://github.com/arogozhnikov/einops#supported-frameworks

