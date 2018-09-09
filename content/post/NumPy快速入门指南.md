---
title: NumPy快速入门指南
date: 2017-12-19T22:08:10+08:00
tags:
    - numpy
categories:
    - python数据分析
slug: numpy-qucikstart
description: >
    Numpy库是python数据分析的基础库, 大部分数据分析的操作都离不开这个库。本文主要根据Numpy官网的快速入门指南整理而来, 介绍了常用的Numpy操作。

---


参考:
<https://docs.scipy.org/doc/numpy-dev/user/quickstart.html>

## 准备
安装numpy

	$ pip install numpy


## 基础

> In NumPy dimensions are called axes. The number of axes is rank.

Numpy中，维度被称作`axes`, 维度数被称作`rank`。

Numpy的数组类是`ndarray`, 与标准python库的数组不太一样，它包含的元素必须是**相同类型**的。

`ndarray`的常见属性如下:

* `ndarray.ndim`数组的轴数(即`rank`)
* `ndarray.shape`数组的维度，返回的是一个元组，元组的长度值刚好是`ndim`
* `ndarray.size`数组元素的个数
* `ndarray.dtype`数组元素的类型
* `ndarray.itemsize`数组元素的字节大小
* `ndarray.data`数组包含的实际数据(一般情况下不会用到这个属性，都是通过索引来访问元素)

<!--more-->

### 数组例子

```python
>>> import numpy as np

>>> a = np.arange(15).reshape(3,5)

>>> a
array([[ 0,  1,  2,  3,  4],
       [ 5,  6,  7,  8,  9],
       [10, 11, 12, 13, 14]])

>>> type(a)
numpy.ndarray

>>> a.ndim
2

>>> a.shape
(3, 5)

>>> a.size
15

>>> a.dtype
dtype('int64')

>>> a.itemsize
8

>>> a.data
<memory at 0x11221b120>
```

### 创建数组

创建数组一般有如下几种方法:

通过`array`函数，可以从普通的python列表或元组来创建

```python
>>> a=np.array([2,3,4])

>>> a
array([2, 3, 4])

>>> a.dtype
dtype('int64')

>>> b = np.array([1.2, 3.5, 5.1])

>>> b.dtype
dtype('float64')
```

一个常见的错误就是在调用`array`函数时， 传递多个参数，而不是一个列表

```python
>>> a = np.array(1,2,3,4)    # 错误
>>> a = np.array([1,2,3,4])  # 正确
```

`array`同样可以接受列表序列并将它转换为多维数组

```python
>>> c = np.array([(1.5, 2.3), (4,5,6)])

>>> c
array([(1.5, 2.3), (4, 5, 6)], dtype=object)
```

同样可以在创建数组的时候，指定数据类型

```python
>>> d = np.array([[1,2], [3,4]], dtype=complex)

>>> d
array([[ 1.+0.j,  2.+0.j],
       [ 3.+0.j,  4.+0.j]])
```

通常情况下， 数组元素的初始数据是不知道的，但是知道数组的大小。因此Numpy提供了一些函数来创建指定大小的数组，并用占位符来填充数组。

* `zeros`函数创建初始值为0的数组
* `ones`创建初始值为1的数组
* `empty`创建未初始化的随机数组

默认情况下，上面三个函数创建数组的元素类型都是`float64`。

```python
>>> np.zeros((3,4))
array([[ 0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.]])

>>> np.ones((3,4))
array([[ 1.,  1.,  1.,  1.],
       [ 1.,  1.,  1.,  1.],
       [ 1.,  1.,  1.,  1.]])

>>> np.empty((2,5))
array([[  2.68156159e+154,   2.68679227e+154,   2.37663529e-312,
          2.56761491e-312,   8.48798317e-313],
       [  9.33678148e-313,   8.70018275e-313,   2.02566915e-322,
          0.00000000e+000,   6.95335581e-309]])
```

为了创建序列函数，Numpy也提供了类似`range`函数的方法

```python
>>> np.arange(10, 30, 5)
array([10, 15, 20, 25])

>>> np.arange(0, 2, 0.3)
array([ 0. ,  0.3,  0.6,  0.9,  1.2,  1.5,  1.8])
```

当使用`arange`函数的生成`float`类型的序列时，生成的序列有时候并不会按照我们预期的步长来生成，要实现这个效果，最好是用`linspace`函数来代替。例如:

```python
>>> from numpy import pi

>>> np.linspace(0, 2, 9)
array([ 0.  ,  0.25,  0.5 ,  0.75,  1.  ,  1.25,  1.5 ,  1.75,  2.  ])

>>> x = np.linspace( 0, 2*pi, 100)

>>> f=np.sin(x)
```

除此之外，还可以使用下面的函数来创建数组

* `array`
* `zeros`
* `zeros_like`  # 创建一个和给定数组相同shape的全是0的数组
* `ones`
* `ones_like`
* `empty`
* `empty_like`
* `arange`
* `linspace`
* `numpy.random.rand` # 从 [0, 1) 中返回一个或多个样本值。
* `numpy.random.randn` # 从标准正态分布中返回一个或多个样本值。
* `fromfunction`
* `fromfile`


### 打印数组

当使用`print`函数打印数组时，numpy会输出一个嵌套的列表形式

```python
# 一维数组
>>> a = np.arange(6)
>>> print(a)
[0 1 2 3 4 5]
# 二维数组
>>> b = np.arange(12).reshape(4,3)
>>> print(b)
[[ 0  1  2]
 [ 3  4  5]
 [ 6  7  8]
 [ 9 10 11]]
# 三维数组
>>> c = np.arange(24).reshape(2,3,4)
>>> print(c)
[[[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]

 [[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]]
```

当数组包含的元素太多时，会省略中间的元素，只打印角落的元素

```python
>>> print(np.arange(10000))
[   0    1    2 ..., 9997 9998 9999]
>>> print(np.arange(10000).reshape(100,100))
[[   0    1    2 ...,   97   98   99]
 [ 100  101  102 ...,  197  198  199]
 [ 200  201  202 ...,  297  298  299]
 ...,
 [9700 9701 9702 ..., 9797 9798 9799]
 [9800 9801 9802 ..., 9897 9898 9899]
 [9900 9901 9902 ..., 9997 9998 9999]]
```
如果想禁用这个行为，强制打印所有的元素，可以开启`set_printoptions`选项

```python
>>> np.set_printoptions(threshold=np.nan)
>>> print(np.arange(100))
[ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24
 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49
 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74
 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99]
```
还原成省略效果

```python
>>> np.set_printoptions(threshold=1000)
```

设置打印浮点数的小数位数:

```python
>>> np.set_printoptions(precision=4) # 设置打印浮点数的小数位数，默认是8位
```

### 基本操作

数组的算术运算会自动作用于每个元素，并返回一个新的数组

```python
>>> a = np.array([20,30,40,50])

>>> b = np.arange(4)

>>> c = a - b

>>> c
array([20, 29, 38, 47])

>>> b**2
array([0, 1, 4, 9])

>>> 10 * np.sin(a)
array([ 9.12945251, -9.88031624,  7.4511316 , -2.62374854])

>>> a < 35
array([ True,  True, False, False], dtype=bool)
```

`*`返回的是每个元素相乘的结果，要实现矩阵乘法，需要使用`dot`函数

```python
>>> a = np.array([ [1, 1],
...                [0, 1]])
...

>>> b = np.array([ [2, 0],
...                [3, 4]])
...

>>> a * b       # 对应位置的元素相乘
array([[2, 0],
       [0, 4]])

>>> a.dot(b)   # 矩阵乘法
array([[5, 4],
       [3, 4]])

>>> np.dot(a, b) # 另一种形式的矩阵乘法
array([[5, 4],
       [3, 4]])
```

一些操作， 如`+=`和`*=`是直接修改原有的数组，而不是新建一个

```python
>>> a = np.ones((2,3), dtype=int)

>>> a
array([[1, 1, 1],
       [1, 1, 1]])

>>> b = np.random.random((2,3))

>>> b
array([[ 0.7216234 ,  0.5813183 ,  0.21175569],
       [ 0.11697569,  0.89835328,  0.06088455]])

>>> a.dtype
dtype('int64')

>>> b.dtype
dtype('float64')

>>> b+=a

>>> b
array([[ 1.7216234 ,  1.5813183 ,  1.21175569],
       [ 1.11697569,  1.89835328,  1.06088455]])

# 报错的原因是因为a数组原来是保存int64类型，现在没法保存float64类型
>>> a+=b
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-11-0a45668e3cc6> in <module>()
----> 1 a+=b

TypeError: Cannot cast ufunc add output from dtype('float64') to dtype('int64') with casting rule 'same_kind'
```

当不同类型的数组运算操作时，总是向精度更高的自动转换

```python
>>> a = np.ones(3, dtype=np.int32)

>>> b = np.linspace(0, np.pi, 3)

>>> b.dtype.name
'float64'

>>> c = a + b

>>> c
array([ 1.        ,  2.57079633,  4.14159265])

>>> c.dtype.name
'float64'

>>> d = np.exp(c*1j)

>>> d
array([ 0.54030231+0.84147098j, -0.84147098+0.54030231j,
       -0.54030231-0.84147098j])

>>> d.dtype.name
'complex128'
```

`ndarray`包含了很多一元运算。如求和等

```python
>>> a = np.arange(15).reshape(3, 5)

>>> a
array([[ 0,  1,  2,  3,  4],
       [ 5,  6,  7,  8,  9],
       [10, 11, 12, 13, 14]])

>>> a.sum()
105

>>> a.min()
0

>>> a.max()
14
```
默认情况下，这些操作都是作用于每一个元素，而不管它的维度。但是，我们也可以通过`axis`参数来限定操作的轴

```python
>>> b = np.arange(12).reshape(3, 4)

>>> b
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

# 计算每一列的和
>>> b.sum(axis=0)
array([12, 15, 18, 21])

# 计算每一行的最小值
>>> b.min(axis=1)
array([0, 4, 8])

# 每一行累积和
>>> b.cumsum(axis=1)
array([[ 0,  1,  3,  6],
       [ 4,  9, 15, 22],
       [ 8, 17, 27, 38]])
```

### 通用函数

Numpy提供了很多常见的数学上的运算，如`sin`, `cos`, `exp`。在Numpy中，我们称这些为"universal functions"（`ufunc`）

```python
>>> B = np.arange(3)

>>> B
array([0, 1, 2])

>>> np.exp(B)
array([ 1.        ,  2.71828183,  7.3890561 ])

>>> 2.71828183 * 2.71828183
7.389056107308149

>>> np.sqrt(B)
array([ 0.        ,  1.        ,  1.41421356])

>>> C = np.array([2., -1., 4.])

>>> np.add(B, C)
array([ 2.,  0.,  6.])
```


### 索引，切片和迭代

一维数组的索引，切片，迭代跟普通的python列表一样

```python
>>> a = np.arange(10) ** 3

>>> a
array([  0,   1,   8,  27,  64, 125, 216, 343, 512, 729])

>>> a[2]
8

>>> a[2:5]
array([ 8, 27, 64])

>>> a[:6:2]  # 等价于a[0:6:2]
array([ 0,  8, 64])

>>> a[:6:2] = -1000

>>> a
array([-1000,     1, -1000,    27, -1000,   125,   216,   343,   512,   729])

>>> a[::-1]  # 反转数组a
array([  729,   512,   343,   216,   125, -1000,    27, -1000,     1, -1000])

>>> for i in a:
...     print(i**(1/3.))
...
nan
1.0
nan
3.0
nan
5.0
6.0
7.0
8.0
9.0
```

多维数组可以在每个轴上索引，多个索引用`,`分隔

```python
>>> def f(x,y):
...     return 10*x +y
...

>>> b=np.fromfunction(f, (5,4), dtype=int)

>>> b
array([[ 0,  1,  2,  3],
       [10, 11, 12, 13],
       [20, 21, 22, 23],
       [30, 31, 32, 33],
       [40, 41, 42, 43]])

>>> help(np.fromfunction)


>>> b[2,3]
23

>>> b[0:5, 1]
array([ 1, 11, 21, 31, 41])

>>> b[:, 1]
array([ 1, 11, 21, 31, 41])

>>> b[1:3, :]
array([[10, 11, 12, 13],
       [20, 21, 22, 23]])
```

当索引数少于轴数时，缺失的索引认为是全切片`:`

```python
>>> b[-1] # 等价于 b[-1,:]
array([40, 41, 42, 43])
```

同样可以使用`...`来表示全切片，它代表补全剩下的所有索引。例如数组`x`, rank是5.那么

* `x[1,2,...]`等价于`x[1,2,:,:,:]`
* `x[...,3]`等价于`x[:,:,:,:,3]`
* `x[4,...,5,:]`等价于`x[4,:,:,5,:]`

```python
>>> c = np.array([[[0,1,2],
...                [10,12,13]],
...                [[100,101,102],
...                [110,112,113]]])
...

>>> c.shape
(2, 2, 3)

>>> c[1,...]
array([[100, 101, 102],
       [110, 112, 113]])

>>> c[...,2]
array([[  2,  13],
       [102, 113]])
```

多维数组的迭代是根据第一个轴来操作的

```python
>>> b
array([[ 0,  1,  2,  3],
       [10, 11, 12, 13],
       [20, 21, 22, 23],
       [30, 31, 32, 33],
       [40, 41, 42, 43]])

>>> for row in b:
...     print(row)
...
[0 1 2 3]
[10 11 12 13]
[20 21 22 23]
[30 31 32 33]
[40 41 42 43]
```

如果想遍历每个元素，可以使用`flat`属性

```python
>>> for element in b.flat:
...     print(element)
...
0
1
2
3
10
11
12
13
20
21
22
23
30
31
32
33
40
41
42
43
```

## shape操作

### 改变数组的shape

许多函数都可以改变数组的shape，但是它们都是返回一个新的修改后的数组，并不会改变原数组

```python
>>> a = np.floor(10*np.random.random((3,4)))

>>> a.shape
(3, 4)

>>> a
array([[ 7.,  8.,  0.,  9.],
       [ 8.,  4.,  9.,  8.],
       [ 4.,  3.,  7.,  0.]])

# 返回降维的数组
>>> a.ravel()
array([ 7.,  8.,  0.,  9.,  8.,  4.,  9.,  8.,  4.,  3.,  7.,  0.])

# 直接修改shape
>>> a.reshape(6,2)
array([[ 7.,  8.],
       [ 0.,  9.],
       [ 8.,  4.],
       [ 9.,  8.],
       [ 4.,  3.],
       [ 7.,  0.]])

# 数组转置
>>> a.T
array([[ 7.,  8.,  4.],
       [ 8.,  4.,  3.],
       [ 0.,  9.,  7.],
       [ 9.,  8.,  0.]])

>>> a.T.shape
(4, 3)

>>> a.shape
(3, 4)
```


`reshape`返回修改后的数组，不改变数组本身，但是`resize`函数直接修改原数组

```python
>>> a
array([[ 7.,  8.,  0.,  9.],
       [ 8.,  4.,  9.,  8.],
       [ 4.,  3.,  7.,  0.]])

>>> a.resize((2,6))

>>> a
array([[ 7.,  8.,  0.,  9.,  8.,  4.],
       [ 9.,  8.,  4.,  3.,  7.,  0.]])
```

如果一个维度为的是`-1`, 那么`reshape`函数会自动计算它的值。

```python
>>> a
array([[ 7.,  8.,  0.,  9.,  8.,  4.],
       [ 9.,  8.,  4.,  3.,  7.,  0.]])

>>> a.reshape(3, -1)
array([[ 7.,  8.,  0.,  9.],
       [ 8.,  4.,  9.,  8.],
       [ 4.,  3.,  7.,  0.]])
```

### 数组合并

多个数组可以根据不同的轴组合在一起

```python
>>> a = np.floor(10*np.random.random((2,2)))

>>> a
array([[ 1.,  1.],
       [ 4.,  4.]])

>>> b = np.floor(10*np.random.random((2,2)))

>>> b
array([[ 2.,  9.],
       [ 0.,  3.]])

>>> np.vstack((a, b))
array([[ 1.,  1.],
       [ 4.,  4.],
       [ 2.,  9.],
       [ 0.,  3.]])

>>> np.hstack((a,b))
array([[ 1.,  1.,  2.,  9.],
       [ 4.,  4.,  0.,  3.]])
```

`column_stack`函数把1维数组当做列来拼成2维数组，如果只是操作2维数组，跟`hstack`的等效。

```python
>>> a
array([[ 1.,  1.],
       [ 4.,  4.]])

>>> b
array([[ 2.,  9.],
       [ 0.,  3.]])

# 操作2维数组，等效于hstack
>>> np.column_stack((a, b))
array([[ 1.,  1.,  2.,  9.],
       [ 4.,  4.,  0.,  3.]])

>>> a = np.array([4., 2.])

>>> b = np.array([3., 8.])

# 操作1维数组，返回2维数组，a,b分别为2维数组的列
>>> np.column_stack((a, b))
array([[ 4.,  3.],
       [ 2.,  8.]])

>>> from numpy import newaxis
# 将1维数组变成2维数组
>>> a[:,newaxis]
array([[ 4.],
       [ 2.]])

# 都是操作二维数组，下面两个操作column_stack和hstack等效
>>> np.column_stack((a[:, newaxis], b[:, newaxis]))
array([[ 4.,  3.],
       [ 2.,  8.]])

>>> np.hstack((a[:, newaxis], b[:, newaxis]))
array([[ 4.,  3.],
       [ 2.,  8.]])
```

另外不论什么数组，`row_stack`函数等效于`vstack`。通常来说，2维以上的数组，`hstack`基于第2根轴做运算，`vstack`基于第1根轴，`concatenate`函数额外多接受一个参数，可以指定基于哪根轴做数组的合并操作。


另外, `r_`和`c_`函数对于在一个轴上组合数据相当哟偶用，他们允许使用范围符号`:`

```python
>>> np.r_[1:4, 0, 4]
array([1, 2, 3, 0, 4])
```
### 数组切割

使用`hsplit`函数，你可以在水平方向切割一个数组

```python
>>> a
array([[ 9.,  0.,  2.,  0.,  0.,  4.,  1.,  6.,  4.,  8.,  3.,  9.],
       [ 5.,  3.,  0.,  5.,  5.,  8.,  0.,  5.,  6.,  3.,  8.,  7.]])

# 切割成3个数组
>>> np.hsplit(a, 3)
[array([[ 9.,  0.,  2.,  0.],
        [ 5.,  3.,  0.,  5.]]), array([[ 0.,  4.,  1.,  6.],
        [ 5.,  8.,  0.,  5.]]), array([[ 4.,  8.,  3.,  9.],
        [ 6.,  3.,  8.,  7.]])]

 >>> np.vsplit(a, 2)
[array([[ 9.,  0.,  2.,  0.,  0.,  4.,  1.,  6.,  4.,  8.,  3.,  9.]]),
 array([[ 5.,  3.,  0.,  5.,  5.,  8.,  0.,  5.,  6.,  3.,  8.,  7.]])]

# 基于第3和第4列切割
>>> np.hsplit(a, (3,4))
[array([[ 9.,  0.,  2.],
        [ 5.,  3.,  0.]]), array([[ 0.],
        [ 5.]]), array([[ 0.,  4.,  1.,  6.,  4.,  8.,  3.,  9.],
        [ 5.,  8.,  0.,  5.,  6.,  3.,  8.,  7.]])]
```

`vsplit`可以基于垂直轴切割，`array_split`可以指定基于哪个轴切割


## 复制和视图(Views)

当进行数组运算和改变数组时，有时候数据是被复制到一个新的数组，有时候不是。对于初学者来说，对于具体是哪种操作，很容易混淆。 主要分三种情况。

### 一点也不复制

```python
>>> a = np.arange(12)

>>> b = a   # 不会有新对象产生

>>> b is a  # a和b是同一个数组
True

>>> b.shape
(12,)

>>> b.shape = 3, 4  # 改变b的shape, a也同样变化

>>> a.shape
(3, 4)

>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])
```

python中使用可变参数时，可以看做是引用传参，因此函数调用不会复制

```python
>>> def f(x):
...     print(id(x))
...

>>> id(a)
4361164320

>>> f(a)
4361164320
```

### 视图(View)和浅复制(Shallow Copy)

不同的数组可以共享数据，`view`函数可以创造一个数据相同的新数组

```python
>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

>>> c = a.view()

>>> c is a   # c和a不是同一个数组
False

>>> c.base is a  # c是a的数据的视图
True

>>> c.flags.owndata
False

>>> c.shape = 2, 6  # a的不会改变

>>> a.shape
(3, 4)

>>> c[0, 4] = 1234  # a的数据发生改变

>>> a
array([[   0,    1,    2,    3],
       [1234,    5,    6,    7],
       [   8,    9,   10,   11]])

>>> c
array([[   0,    1,    2,    3, 1234,    5],
       [   6,    7,    8,    9,   10,   11]])
```

一个数组的切片返回的就是它的视图

```python
>>> a
array([[   0,    1,    2,    3],
       [1234,    5,    6,    7],
       [   8,    9,   10,   11]])

>>> s = a [:, 1:3]

>>> s
array([[ 1,  2],
       [ 5,  6],
       [ 9, 10]])

>>> s[:] = 10 # s[:]是s的视图

>>> s
array([[10, 10],
       [10, 10],
       [10, 10]])
```

### 深度复制(Deep Copy)

`copy`方法可以完全复制数组和它的数据

```python
>>> a = np.arange(12).reshape((3,4))

>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

>>> d = a.copy()

>>> d is a
False

>>> d.base is a
False

>>> d[0, 0] = 9999

>>> d
array([[9999,    1,    2,    3],
       [   4,    5,    6,    7],
       [   8,    9,   10,   11]])

>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])
```

### 函数和方法概览

如下是按照分类整理的常用函数和方法，完整的分类可以参考[Routines](https://docs.scipy.org/doc/numpy-dev/reference/routines.html#routines)

**数组创建**

* arange
* array
* copy
* empty
* empty_like
* eye  # 创建一个对角线全是1的二维数组
* fromfile
* fromfunction
* identity # 创建一个对角线全是1的方形矩阵，与eye方法差不多，只是可以接受的参数不同
* linspace
* logspace  # 创建等比数列
* mgrid
* orgid
* ones
* ones_like
* zeros
* zeros_like

**转换**

* ndarray.astype # 改变数组的元素格式
* atleast_1d  # 将输入转换为至少1维数组
* atleast_2d
* alteast_3d
* mat         # 将输入转换为矩阵

**处理**

* array_split
* column_stack
* concatenate
* diagonal
* dsplit
* dstack
* hsplit
* hstack
* ndarray.item
* newaxis
* ravel
* repeat
* reshape
* resize
* squeeze
* swapaxes
* take
* transpose
* vsplit
* vstack

**Questions**

* all
* any
* nonezero
* where

**排序**

* argmax # 返回最大值的索引
* argmin # 返回最小值的索引
* argsort  # 返回排序后的索引
* max
* min
* ptp
* searchsorted
* sort


**运算**

* choose
* compress
* cumprod
* cumsum
* inner
* ndarray.fill
* imag
* prod
* put
* putmask
* real
* sum

**基本统计**

* cov
* mean
* std
* var

**线性代数**

* cross
* dot
* outer
* linalg
* svd
* vdot

## Less Basic

### 广播机制

属于广播主要描述于numpy对于不同shape的数组如何进行算术运算。受限于一些特定约束。
一般都是小的数组扩展为大的数组，以便能计算。

通常情况下，numpy操作的数组必须是相同shape的。

```python
>>> import numpy as np

>>> a = np.array([1.0, 2.0, 3.0])

>>> b = np.array([2.0, 2.0, 2.0])

>>> a * b
array([ 2.,  4.,  6.])
```

当数组的shape满足某些特定约束时，numpy的广播机制可以使这个约束更宽松。最简单的就是广播例子就是当数组和一个标量操作时。

```python
>>> a = np.array([1.0, 2.0, 3.0])

>>> b = 2.0

>>> a * b
array([ 2.,  4.,  6.])
```

上面两个例子的结果是一样的，我们可以认为标量b被扩展为了和a同样shape的数组，b中的新元素就是原来标量的拷贝。这个扩展策略仅仅是概念上的，实际上Numpy足够聪明，能自动使用标量做运算，而不需要复制任何东西。所以广播运算从计算内存上来说更优秀。

要能满足广播，必须符合下面两条规则：

1. 广播之后，输出数组的shape是输入数组shape的各个轴上的最大值，然后沿着较大shape属性的方向复制延伸；
2. 要进行广播机制，要么两个数组的shape属性一样，要么其中有一个数组的shape属性必须有一个等于1；

更多可以参考:

* <https://docs.scipy.org/doc/numpy-dev/user/basics.broadcasting.html>
* <http://www.labri.fr/perso/nrougier/from-python-to-numpy/?utm_source=mybridge&utm_medium=blog&utm_campaign=read_more#broadcasting>

### 索引

numpy除了支持普通的python方式的索引和切片之外，还支持整数数组或布尔数组索引

### 数组索引

```python
>>> a = np.arange(12) ** 2

>>> i = np.array([1,1,3,8,5])

>>> a
array([  0,   1,   4,   9,  16,  25,  36,  49,  64,  81, 100, 121])

>>> i
array([1, 1, 3, 8, 5])

# 返回a中在索引i的元素
>>> a[i]
array([ 1,  1,  9, 64, 25])

>>> j = np.array([[3,4], [9,7]])

# 二维数组索引，返回a中在索引j的元素
>>> a[j]
array([[ 9, 16],
       [81, 49]])
```

当数组索引作用在多维数组时，是根据数组的第一个维度来索引的。

```python
>>> palette = np.array([ [0 ,0, 0],
...                      [255, 0, 0],
...                      [0, 255, 0],
...                      [0, 0, 255],
...                      [255, 255, 255] ])
...

>>> image = np.array([ [0, 1, 2, 0],
...                    [0, 3, 4, 0] ])
...

>>> palette[image]
array([[[  0,   0,   0],
        [255,   0,   0],
        [  0, 255,   0],
        [  0,   0,   0]],

       [[  0,   0,   0],
        [  0,   0, 255],
        [255, 255, 255],
        [  0,   0,   0]]])
```

索引同样可以是多维的，但是必须是相同的shape

```python
>>> a = np.arange(12).reshape(3, 4)

>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

# indices for the first dim of a
>>> i = np.array([[0,1],
...                [1,2] ])
...

# indices for the second dim
>>> j = np.array([[2, 1],
...               [3, 3] ])
...

# i and j must have equal shape
# 返回的结果是是[ [a[0,2], a[1,1]
#                [a[1,3], a[2, 3] ]
>>> a[i, j]
array([[ 2,  5],
       [ 7, 11]])

# [ a[0,2], a[1, 2],
#   a[1,2], a[2, 2] ]
>>> a[i, 2]
array([[ 2,  6],
       [ 6, 10]])

# [[[ a[0,2], a[0,1],
#     a[0,3], a[0,3]],
#
#     a[1,2], a[1,1],
#     a[1,3], a[1,3]],
#
#     a[2,2], a[2,1],
#     a[2,3], a[2,3]]]
>>> a[:, j]
array([[[ 2,  1],
    [ 3,  3]],

   [[ 6,  5],
    [ 7,  7]],

   [[10,  9],
    [11, 11]]])
```

同样，我们可以把`i`和`j`放在一个列表里，然后用列表做索引

```python
>>> l = [i, j]

# 等价于a[i, j]
>>> a[l]
array([[ 2,  5],
       [ 7, 11]])
```

但是，我们不可以把`i`和`j`放在一个数组里，因为数组索引是作用在第一个维度上的。

```python
>>> s = np.array([i, j])

>>> s
array([[[0, 1],
        [1, 2]],

       [[2, 1],
        [3, 3]]])

>>> a[s]
---------------------------------------------------------------------------
IndexError                                Traceback (most recent call last)
<ipython-input-42-c057dc68e5fe> in <module>()
----> 1 a[s]

IndexError: index 3 is out of bounds for axis 0 with size 3

# 等价于 a[i, j]
>>> a[tuple(s)]
array([[ 2,  5],
       [ 7, 11]])
```

我们同样可以给数组索引赋值

```python
>>> a = np.arange(5)

>>> a
array([0, 1, 2, 3, 4])

>>> a[[1,3,4]] = 0

>>> a
array([0, 0, 2, 0, 0])
```

但是当列表包含相同的索引时，这个位置会被赋值多次，最终只保留最后一次的值

```python
>>> a = np.arange(5)

>>> a[[0, 0, 2]] = [1,2,3]

>>> a
array([2, 1, 3, 3, 4])
```

上面看起来很合理，但是当使用`+=`符号的时候，结果和我们想的可能不太一样

```python
>>> a = np.arange(5)

>>> a[[0, 0, 2]] += 1

>>> a
array([1, 1, 3, 3, 4])
```

尽管索引中出现了两次0，但是第0个元素它只加了1次。

### 布尔数组索引

当使用数字数组索引时，我们提供了哪些元素要被索引的信息。但是当使用布尔数组时，我们是明确哪些元素需要，哪些元素不需要。

```python
>>> a = np.arange(12).reshape((3,4))

>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

>>> b = a > 4

>>> b
array([[False, False, False, False],
       [False,  True,  True,  True],
       [ True,  True,  True,  True]], dtype=bool)

>>> a[b]
array([ 5,  6,  7,  8,  9, 10, 11])
```

这个特性非常适合用来赋值

```python
# 所有大于4的元素都赋值为0
>>> a[b] = 0

>>> a
array([[0, 1, 2, 3],
       [4, 0, 0, 0],
       [0, 0, 0, 0]])
```

一个使用布尔数组索引的例子就是[曼德博集合(Mandelbrot set)](https://zh.wikipedia.org/wiki/%E6%9B%BC%E5%BE%B7%E5%8D%9A%E9%9B%86%E5%90%88)

```python
import numpy as np
import matplotlib.pyplot as plt
def mandelbrot( h,w, maxit=20 ):
    """Returns an image of the Mandelbrot fractal of size (h,w)."""
    y,x = np.ogrid[ -1.4:1.4:h*1j, -2:0.8:w*1j ]
    c = x+y*1j
    z = c
    divtime = maxit + np.zeros(z.shape, dtype=int)

    for i in range(maxit):
        z = z**2 + c
        diverge = z*np.conj(z) > 2**2            # who is diverging
        div_now = diverge & (divtime==maxit)  # who is diverging now
        divtime[div_now] = i                  # note when
        z[diverge] = 2                        # avoid diverging too much

    return divtime
plt.imshow(mandelbrot(400,400))
plt.show()
```

另一个布尔数组的场景跟数字数组索引类似，对每个维度，我们提供一个1维的数组来选择我们需要的切片

```python
>>> a = np.arange(12).reshape(3, 4)

>>> b1 = np.array([False, True, True])

>>> b2 = np.array([True, False, True, False])

>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

# 选择行
>>> a[b1, :]
array([[ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

# 同上
>>> a[b1]
array([[ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

# 选择列
>>> a[:, b2]
array([[ 0,  2],
       [ 4,  6],
       [ 8, 10]])

# a weird thing to do
>>> a[b1, b2]
array([ 4, 10])
```

### [ix_()函数](https://docs.scipy.org/doc/numpy-dev/reference/generated/numpy.ix_.html#numpy.ix_) （作用待定）


### 字符串索引

Numpy提供了创建结构化的数组的能力，可以通过列名来操作数据

```python
# dtype分别指定每一个的名字和数据类型
>>> x = np.array([(1, 2., 'Hello'), (2, 3., 'World')], dtype=[('foo', 'i4'), ('bar', 'f4'), ('baz', 'S10')])

>>> x[1]
(2,  3., b'World')

>>> x['foo']
array([1, 2], dtype=int32)

>>> x['bar']
array([ 2.,  3.], dtype=float32)

>>> x['baz']
array([b'Hello', b'World'],
      dtype='|S10')
```

## [更多](https://docs.scipy.org/doc/numpy-dev/user/basics.html)

* Data types
* Array creation
* I/O with NumPy
* Indexing
* Broadcasting
* Byte-swapping
* Structured arrays
* Subclassing ndarray

## 扩展阅读

* [100 numpy exercises](http://www.labri.fr/perso/nrougier/teaching/numpy.100/)
* [试验性的Numpy教程](http://reverland.org/python/2012/08/22/numpy)
* [From Python to Numpy](http://www.labri.fr/perso/nrougier/from-python-to-numpy/)
