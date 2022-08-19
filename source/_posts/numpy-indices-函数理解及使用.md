---
title: numpy.indices()函数理解及使用
date: 2022-08-19 07:24:02
categories:
tags:
top_img:
---

在阅读一段相机标定代码的时候，看到代码里面用到了 numpy.indices() 函数，查阅了 numpy.indices() 函数的官方文档后还是不能理解它的作用和使用方法，所以后面又查阅了很多关于该函数的博客，最后终于想明白了。

## 从一个例子说起
为了方便理解，我们先不解释它的具体作用，而是选择从一个简单的例子说起。

假如你想得到一个矩阵 $M$，并且希望矩阵 $M$ 第 $(i, j)$ 个元素值为 $M(i, j) = i + j$。

### 方法一：使用循环赋值

```python
M = numpy.zeros((2, 3))
for i in range(2):
    for j in range(3):
        M(i, j) = i + j
```
得到矩阵 $M$ 结果如下：
$$
 \begin{bmatrix}0&1&2\\1&2&3\end{bmatrix} 
$$

### 方法二：使用两个矩阵相加得到
矩阵 $M$ 也可以由两个矩阵相加得到，其中矩阵 $A(i, j) = i$ 而 $B(i, j) = j$

```python
import numpy

A = numpy.zeros((2, 3))
for i in range(2):
    A[i, :] = i

B = numpy.zeros((2, 3))
for j in range(3):
    B[:, j] = j

M = A + B
```

得到
$$
 A = \begin{bmatrix}0&0&0\\1&1&1\end{bmatrix}
$$

$$
 B = \begin{bmatrix}0&1&2\\0&1&2\end{bmatrix}
$$

$$
 M = \begin{bmatrix}0&1&2\\1&2&3\end{bmatrix} 
$$


### 方法三：使用 numpy.indices() 函数
其实，方法二中的矩阵 $A$ 和 $B$ 可以使用 numpy.indices() 函数得到：

```python
import numpy

A, B = numpy.indices((2, 3))
M = A + B
```

最后得到的矩阵 $M$ 和前两个方法一样。

## 函数理解
由以上举例，我们可以看到 numpy.indices() 函数返回的是一个<b>由索引组成的数组</b>, numpy.indices((2, 3)) 返回的是一个尺寸为 $2*2*3$ 的矩阵 $N$，$N[0,:,:]$ 是由横坐标索引组成的矩阵(即方法二中的矩阵 $A$)，$N[1,:,:]$ 是由纵坐标索引组成的矩阵(即方法二中的矩阵 $B$)。

## 举一反三
如果想得到一个矩阵 $M$, 其元素是 $M_ij = 2*i + 3*j$，
```python
import numpy

A, B = numpy.indices((2, 3))
M = 2*A + 3*B
```

结果为:
$$
 M = \begin{bmatrix}0&3&6\\2&5&8\end{bmatrix} 
$$





