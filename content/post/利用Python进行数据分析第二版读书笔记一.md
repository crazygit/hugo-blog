---
title: 《利用Python进行数据分析第二版》读书笔记一
date: 2017-12-26T18:21:46+08:00
tags:
    - numpy
    - pandas
    - matplotlib
categories:
    - python数据分析
slug: python-for-data-analysis-2nd-edition-note-one
description: >
    本文是利用Python进行数据分析的读书笔记一，主要记录了Numpy, Pandas的一些基础操作和数据读取和保存方法

---

## Numpy基础

更多关于NumPy的使用, 可以参考[NumPy快速入门指南]({{< ref "NumPy快速入门指南.md" >}})

### Numpy数据类型

![Numpy DataTypes](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2017-12-26-Numpy_data_types.png)

### np.where

`np.where`是一个向量版本的三元表达式`x if c else y`。假如我们有三个数组`xarr`, `yarr`, `cond`。
当`cond`为真时，取`xarr`的值，否则取`yarr`的值。我们可以这样做:

```python
>>> xarr = np.array([1.1, 1.2, 1.3, 1.4, 1.5])

>>> yarr = np.array([2.1, 2.2, 2.3, 2.4, 2.5])

>>> cond = np.array([True, False, True, True, False])

>>> result = [(x if c else y) for x, y, c in zip(xarr, yarr, cond)]

>>> result
[1.1000000000000001, 2.2000000000000002, 1.3, 1.3999999999999999, 2.5]
```
<!--more-->

但是上面这种做法是纯python的做法，效率很慢，而且无法用于多维数组。用`np.where`就很简单了。

```python
>>> result = np.where(cond, xarr, yarr)

>>> result
array([ 1.1,  2.2,  1.3,  1.4,  2.5])
```

`np.where`的第二个和第三个参数，除了数组类型之外，也可以是标量

```python
>>> arr = np.random.randn(4, 4)

>>> arr
array([[-0.16323877, -0.65162139, -1.53392306, -0.84839303],
       [ 0.67860968,  1.02998872, -1.66306679, -1.23551981],
       [ 0.66004673, -1.13882878, -0.90901002, -1.11414436],
       [ 0.24704485, -0.87881313, -1.06539659, -0.82992508]])

>>> arr > 0
array([[False, False, False, False],
       [ True,  True, False, False],
       [ True, False, False, False],
       [ True, False, False, False]], dtype=bool)

# 将数组中>0的全部替换为2，小于0的全部用-2替换
>>> np.where(arr > 0, 2, -2)
array([[-2, -2, -2, -2],
       [ 2,  2, -2, -2],
       [ 2, -2, -2, -2],
       [ 2, -2, -2, -2]])

# 将数组中>0的全部替换为2，小于0的不变
>>> np.where(arr > 0, 2, arr)
array([[-0.16323877, -0.65162139, -1.53392306, -0.84839303],
       [ 2.        ,  2.        , -1.66306679, -1.23551981],
       [ 2.        , -1.13882878, -0.90901002, -1.11414436],
       [ 2.        , -0.87881313, -1.06539659, -0.82992508]])
```

### 布尔数组

当为`True`时，赋值为1，为`False`时，赋值为0。利用这个特性，很容易计算出一个数组里满足特定条件的元素个数。

```python
>>> arr = np.random.randn(100)

# 统计大于0的元素个数
>>> (arr > 0).sum()
48

# 统计小于0的元素个数
>>> (arr < 0).sum()
52
```

`any`函数用于检查数组中有一个元素或多个元素的值为`True`。`all`函数用于检查数组中所有的元素是否为`True`。

```python
>>> bools = np.array([False, False, True, False])

>>> bools.any()
True

>>> bools.all()
False
```

### 数组排序

就跟python内置的`sort`函数一样，Numpy提供了排序功能

```python
>>> arr = np.random.randn(6)

>>> arr
array([ 0.45462395,  1.89881592,  0.23095329, -0.15011573, -0.50788654,
       -0.19426878])

>>> arr.sort()

>>> arr
array([-0.50788654, -0.19426878, -0.15011573,  0.23095329,  0.45462395,
        1.89881592])
```

也可以在指定轴排序

```python
>>> arr = np.random.randn(5, 3)

>>> arr
array([[ 0.85243833, -0.20601835,  0.42260075],
       [ 0.0022698 ,  0.69309103, -0.6865517 ],
       [ 0.17471696,  0.77361444, -0.25720617],
       [ 0.83807922,  0.43469269, -0.18505689],
       [-0.19003894,  0.29031477,  1.68124349]])

# 在列方向排序
>>> arr.sort(axis=0)

>>> arr
array([[-0.19003894, -0.20601835, -0.6865517 ],
       [ 0.0022698 ,  0.29031477, -0.25720617],
       [ 0.17471696,  0.43469269, -0.18505689],
       [ 0.83807922,  0.69309103,  0.42260075],
       [ 0.85243833,  0.77361444,  1.68124349]])

# 在行方向排序
>>> arr.sort(axis=1)

>>> arr
array([[-0.6865517 , -0.20601835, -0.19003894],
       [-0.25720617,  0.0022698 ,  0.29031477],
       [-0.18505689,  0.17471696,  0.43469269],
       [ 0.42260075,  0.69309103,  0.83807922],
       [ 0.77361444,  0.85243833,  1.68124349]])
```

### 唯一性和in操作

Numpy专门为一维数组提供了一些函数

**唯一性（unique)**

```python
>>> names = np.array(['Bob', 'Joe', 'Will', 'Bob', 'Will', 'Joe', 'Joe'])

>>> np.unique(names)
array(['Bob', 'Joe', 'Will'],
      dtype='<U4')

>>> ints = np.array([3, 3, 3, 2, 2, 1, 1, 4, 4])

>>> np.unique(ints)
array([1, 2, 3, 4])
```

**np.in1d**

```python
>>> values = np.array([6, 0, 0, 3, 2, 5, 6])

# 检查一个数组的值是否在另一个数组里，返回一个布尔数组
>>> np.in1d(values, [2, 3, 6])
array([ True, False, False,  True,  True, False,  True], dtype=bool)
```

### 文件输入输出

numpy可以使用`save`和`load`方法来输出和加载。默认情况下，是以未压缩的二进制格式保存，并会自动加上后缀扩展`.npy`

```python
>>> import numpy as np

>>> arr = np.arange(10)

>>> arr
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

>>> np.save('some_array', arr)

>>> np.load('some_array.npy')
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
```

使用`savez`可以一次以未压缩的二进制格式保存多个数组

```python
>>> xarr = np.arange(10)

>>> barr = np.arange(5)

# 使用a,b参数分别保存xarr和barr
>>> np.savez('array_archive.npz', a=xarr, b=barr)

>>> arch = np.load('array_archive.npz')

>>> arch['a']
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

>>> arch['b']
array([0, 1, 2, 3, 4])
```

以压缩的格式保存数组

```python
>>> np.savez_compressed('arrays_compressed.npz', a=arr, b=arr)
```

### 矩阵乘法的另一种表示方法

在python3.5中, 可以使用`@`来表示矩阵乘法, 跟`dot`函数效果一样

```python
>>> x = np.array([[1., 2., 3.], [4., 5., 6.]])

>>> np.dot(x, np.ones(3))
array([  6.,  15.])

>>> x @ np.ones(3)
array([  6.,  15.])
```

### 线性代数(Linear Algebra)

`numpy.linalg` 模块包含类线性代数常用的操作


## Pandas基础

更多关于Pandas的使用, 可以参考[10分钟入门Pandas]({{< ref "10分钟入门Pandas.md" >}})

### `Series`对象和它的`index`都有一个`name`属性

```python
>>> s = pd.Series(np.arange(10))

>>> s
0    0
1    1
2    2
3    3
4    4
5    5
6    6
7    7
8    8
9    9
dtype: int64

>>> s.name = 'IntSeries'

>>> s.index.name = 'IntSeriesIndex'

>>> s
IntSeriesIndex
0    0
1    1
2    2
3    3
4    4
5    5
6    6
7    7
8    8
9    9
Name: IntSeries, dtype: int64
```

### 列操作

```python
>>> data = pd.DataFrame(np.arange(12).reshape(3,4), columns=['a', 'b', 'c', 'd'])

>>> data
   a  b   c   d
0  0  1   2   3
1  4  5   6   7
2  8  9  10  11

# 添加列
>>> data['e']= 5

>>> data
   a  b   c   d  e
0  0  1   2   3  5
1  4  5   6   7  5
2  8  9  10  11  5

# 删除列
>>> del data['a']

>>> data
   b   c   d  e
0  1   2   3  5
1  5   6   7  5
2  9  10  11  5
```

### DataFrame的index和column也可以设置name属性

```python
>>> data = pd.DataFrame(np.arange(12).reshape(3,4), columns=['a', 'b', 'c', 'd'])

>>> data
   a  b   c   d
0  0  1   2   3
1  4  5   6   7
2  8  9  10  11

>>> data.index.name = 'index_name'

>>> data.columns.name = 'columns_name'

>>> data
columns_name  a  b   c   d
index_name
0             0  1   2   3
1             4  5   6   7
2             8  9  10  11
```

### Reindexing

`Series`和`DataFrame`对象可以使用`reindex`方法来修改行索引和列索引

```python
# Series reindex
>>> obj = pd.Series(range(3), index=['a', 'b', 'c'])

>>> obj
a    0
b    1
c    2
dtype: int64

>>> obj2 = obj.reindex(['a', 'b', 'c', 'd', 'e'])

>>> obj2
a    0.0
b    1.0
c    2.0
d    NaN
e    NaN
dtype: float64

# DataFrame reindex
>>> frame = pd.DataFrame(np.arange(9).reshape(3,3), index=['a', 'c', 'd'], columns=['Ohio', 'Texas', 'California'])

>>> frame
   Ohio  Texas  California
a     0      1           2
c     3      4           5
d     6      7           8

>>> frame2 = frame.reindex(['a', 'b', 'c', 'd'])

>>> frame2
   Ohio  Texas  California
a   0.0    1.0         2.0
b   NaN    NaN         NaN
c   3.0    4.0         5.0
d   6.0    7.0         8.0

>>> states = ['Texas', 'Utah', 'California']

>>> frame.reindex(columns=states)
   Texas  Utah  California
a      1   NaN           2
c      4   NaN           5
d      7   NaN           8
```

`reindex`后的值还可以使用`method`参数来补充新添加的值

```python
>>> obj = pd.Series(['blue', 'purple', 'yellow'], index=[0, 2, 4])

>>> obj
0      blue
2    purple
4    yellow
dtype: object

# method的可选值
# * default: don't fill gaps
# * pad / ffill: propagate last valid observation forward to next
#   valid
# * backfill / bfill: use next valid observation to fill gap
# * nearest: use nearest valid observations to fill gap
>>> obj.reindex(range(6), method='ffill')
0      blue
1      blue
2    purple
3    purple
4    yellow
5    yellow
dtype: object
```

### Drop行或列

```python
>>> obj = pd.Series(np.arange(5.), index=['a', 'b', 'c', 'd', 'e'])

>>> obj
a    0.0
b    1.0
c    2.0
d    3.0
e    4.0
dtype: float64

# drop掉一个索引
>>> new_obj = obj.drop('c')

>>> new_obj
a    0.0
b    1.0
d    3.0
e    4.0
dtype: float64

>>> obj.drop(['d', 'c'])
a    0.0
b    1.0
e    4.0
dtype: float64

>>> data = pd.DataFrame(np.arange(16).reshape((4, 4)),
...                     index=['Ohio', 'Colorado', 'Utah', 'New York'],
...                      columns=['one', 'two', 'three', 'four'])
...

>>> data
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15

# drop行
>>> data.drop(['Colorado', 'Ohio'])
          one  two  three  four
Utah        8    9     10    11
New York   12   13     14    15

# drop列
>>> data.drop('two', axis=1)
          one  three  four
Ohio        0      2     3
Colorado    4      6     7
Utah        8     10    11
New York   12     14    15

# 默认情况下，drop方法会返回一个新的对象，但是不会修改原来的值
# 使用inplace参数可以直接修改原来的值
>>> data.drop('Utah', inplace=True)

>>> data
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
New York   12   13     14    15
```

### 函数应用和映射

```python
>>> frame = pd.DataFrame(np.random.randn(4, 3), columns=list('bde'), index=['Utah', 'Ohio', 'Texas', 'Oregon'])

>>> frame
               b         d         e
Utah    0.282987 -0.941163 -1.212623
Ohio   -1.072346 -0.245222 -0.731497
Texas   2.264081  0.169154 -0.512539
Oregon -0.795480  0.630302  1.127120

>>> np.abs(frame)
               b         d         e
Utah    0.282987  0.941163  1.212623
Ohio    1.072346  0.245222  0.731497
Texas   2.264081  0.169154  0.512539
Oregon  0.795480  0.630302  1.127120

>>> f = lambda x: x.max() - x.min()

# 默认作用于每列
>>> frame.apply(f)
b    3.336427
d    1.571465
e    2.339742
dtype: float64

# 指定作用于每行
>>> frame.apply(f, axis='columns')
Utah      1.495609
Ohio      0.827124
Texas     2.776620
Oregon    1.922600
dtype: float64

# applymap作用于每个元素
>>> format = lambda x: '%.2f' % x

>>> frame.applymap(format)
            b      d      e
Utah     0.28  -0.94  -1.21
Ohio    -1.07  -0.25  -0.73
Texas    2.26   0.17  -0.51
Oregon  -0.80   0.63   1.13

>>> frame['e'].map(format)
Utah      -1.21
Ohio      -0.73
Texas     -0.51
Oregon     1.13
Name: e, dtype: object
```

### 排序(Sort)

按索引排序

```python
>>> obj = pd.Series(range(4), index=['d', 'a', 'b', 'c'])

>>> obj
d    0
a    1
b    2
c    3
dtype: int64

# 按索引排序
>>> obj.sort_index()
a    1
b    2
c    3
d    0
dtype: int64

>>> frame = pd.DataFrame(np.arange(8).reshape((2, 4)), index=['three', 'one'], columns=['d', 'a', 'b', 'c'])

>>> frame
       d  a  b  c
three  0  1  2  3
one    4  5  6  7

# 按索引排序
>>> frame.sort_index()
       d  a  b  c
one    4  5  6  7
three  0  1  2  3

# 按列排序
>>> frame.sort_index(axis=1)
       a  b  c  d
three  1  2  3  0
one    5  6  7  4

# 按照降序排序
>>> frame.sort_index(axis=1, ascending=False)
       d  c  b  a
three  0  3  2  1
one    4  7  6  5
```

按值排序

```python
>>> obj = pd.Series([4, 7, -3, 2])

>>> obj
0    4
1    7
2   -3
3    2
dtype: int64

>>> obj.sort_values()
2   -3
3    2
0    4
1    7
dtype: int64


>>> obj = pd.Series([4, np.nan, 7, np.nan, -3, 2])

# 排序时NaN会自动排在最后
>>> obj.sort_values()
4   -3.0
5    2.0
0    4.0
2    7.0
1    NaN
3    NaN
dtype: float64

>>> frame = pd.DataFrame({'b': [4, 7, -3, 2], 'a': [0, 1, 0, 1]})

>>> frame
   a  b
0  0  4
1  1  7
2  0 -3
3  1  2

# 按照指定的列排序
>>> frame.sort_values(by='b')
   a  b
2  0 -3
3  1  2
0  0  4
1  1  7

# 按照多个列排序
>>> frame.sort_values(by=['a', 'b'])
   a  b
2  0 -3
0  0  4
3  1  2
1  1  7
```

### 排名(rank)

`ranking`（排名）是根据数字的排序，分配一个数字。rank方法能用于series和DataFrame，rank方法默认会给每个group一个mean rank（平均排名）。

rank表示在这个数在原来的Series中排第几名，有相同的数，取其排名平均（默认）作为值。

```python
>>> obj = pd.Series([7, -5, 7, 4, 2, 0, 4])

>>> obj
0    7
1   -5
2    7
3    4
4    2
5    0
6    4
dtype: int64

# 索引1 排第一名。rank值是1.0
# 索引3和6分别排4，5名，所以rank是平均是4.5
>>> obj.rank()
0    6.5
1    1.0
2    6.5
3    4.5
4    3.0
5    2.0
6    4.5
dtype: float64

# 可以看出rank分配的值跟sort_values的排序是一致的
>>> obj.sort_values()
1   -5
5    0
4    2
3    4
6    4
0    7
2    7
```

排序也可以根据数据出现的顺序来排序，而不是使用平均值

```python
>>> obj
0    7
1   -5
2    7
3    4
4    2
5    0
6    4
dtype: int64

# 下面的排序中，虽然索引3和6对应的值是一样的，但是这次rank值并没有取平均值4.5，
# 而是根据它们出现的顺序，rank值依次为4和5
>>> obj.rank(method='first')
0    6.0
1    1.0
2    7.0
3    4.0
4    3.0
5    2.0
6    5.0
dtype: float64

# 排序时，rank值取大的排名
# 索引3和6对应的值是一样的。rank值同时排第5
>>> obj.rank(method='max')
0    7.0
1    1.0
2    7.0
3    5.0
4    3.0
5    2.0
6    5.0
dtype: float64

# 排序时，rank值取小的排名
# 索引3和6对应的值是一样的。rank值同时排第4
>>> obj.rank(method='min')
0    6.0
1    1.0
2    6.0
3    4.0
4    3.0
5    2.0
6    4.0
dtype: float64

# 按照倒序排名
>>> obj.rank(method='min', ascending=False)
0    1.0
1    7.0
2    1.0
3    3.0
4    5.0
5    6.0
6    3.0
dtype: float64


>>> frame = pd.DataFrame({'b': [4.3, 7, -3, 2], 'a': [0, 1, 0, 1], 'c': [-2, 5, 8, -2.5]})

>>> frame
   a    b    c
0  0  4.3 -2.0
1  1  7.0  5.0
2  0 -3.0  8.0
3  1  2.0 -2.5

# 按照行返回rank值
>>> frame.rank(axis='columns')
     a    b    c
0  2.0  3.0  1.0
1  1.0  3.0  2.0
2  2.0  1.0  3.0
3  2.0  3.0  1.0
```


### 重复的索引

```python
>>> obj = pd.Series(range(5), index=['a', 'a', 'b', 'b', 'c'])

>>> obj
a    0
a    1
b    2
b    3
c    4
dtype: int64

# is_unique 属性可以确定index是否含有重复的值
>>> obj.index.is_unique
False
```

## Pandas数据加载，存储

**备注**: 下面例子使用的csv文件可以从这里[下载](https://github.com/wesm/pydata-book)。

### 读取CSV文件

使用`read_csv`加载文件

```python
>>> import pandas as pd

>>> ! cat examples/ex1.csv
a,b,c,d,message
1,2,3,4,hello
5,6,7,8,world
9,10,11,12,foo

>>> df = pd.read_csv('examples/ex1.csv')

>>> df
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

使用`read_table`加载文件

```python
>>> pd.read_table('examples/ex1.csv', sep=',')
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo

>>> !cat examples/ex2.csv
1,2,3,4,hello
5,6,7,8,world
9,10,11,12,foo
```

针对没有header的情况

```python
>>> pd.read_csv('examples/ex2.csv', header=None)
   0   1   2   3      4
0  1   2   3   4  hello
1  5   6   7   8  world
2  9  10  11  12    foo
```

指定columns索引

```python
>>> pd.read_csv('examples/ex2.csv', names=['a', 'b', 'c', 'd', 'message'])
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo

>>> names = ['a', 'b', 'c', 'd', 'message']
```

使用csv中的message列作为index

```python
>>> pd.read_csv('examples/ex2.csv', names=names, index_col='message')
         a   b   c   d
message
hello    1   2   3   4
world    5   6   7   8
foo      9  10  11  12
```

构建hierarchical index

```python
>>> !cat examples/csv_mindex.csv
key1,key2,value1,value2
one,a,1,2
one,b,3,4
one,c,5,6
one,d,7,8
two,a,9,10
two,b,11,12
two,c,13,14
two,d,15,16

>>> parsed = pd.read_csv('examples/csv_mindex.csv', index_col=['key1', 'key2'])

>>> parsed
           value1  value2
key1 key2
one  a          1       2
     b          3       4
     c          5       6
     d          7       8
two  a          9      10
     b         11      12
     c         13      14
     d         15      16

>>> parsed.index
MultiIndex(levels=[['one', 'two'], ['a', 'b', 'c', 'd']],
           labels=[[0, 0, 0, 0, 1, 1, 1, 1], [0, 1, 2, 3, 0, 1, 2, 3]],
           names=['key1', 'key2'])
```

使用其他的分隔符号(如下使用空格作为列分隔符)

```python
>>> list(open('examples/ex3.txt'))
['            A         B         C\n',
 'aaa -0.264438 -1.026059 -0.619500\n',
 'bbb  0.927272  0.302904 -0.032399\n',
 'ccc -0.264273 -0.386314 -0.217601\n',
 'ddd -0.871858 -0.348382  1.100491\n']

>>> result = pd.read_table('examples/ex3.txt', sep='\s+')

>>> result
            A         B         C
aaa -0.264438 -1.026059 -0.619500
bbb  0.927272  0.302904 -0.032399
ccc -0.264273 -0.386314 -0.217601
ddd -0.871858 -0.348382  1.100491
```

过滤掉指定的行

```python
>>> !cat examples/ex4.csv
# hey!
a,b,c,d,message
# just wanted to make things more difficult for you
# who reads CSV files with computers, anyway?
1,2,3,4,hello
5,6,7,8,world
9,10,11,12,foo
>>> pd.read_csv('examples/ex4.csv', skiprows=[0, 2, 3])
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

默认情况下，pandas认为`NULL`和`NA`为缺失的值

```python
>>> !cat examples/ex5.csv
something,a,b,c,d,message
one,1,2,3,4,NA
two,5,6,,8,world
three,9,10,11,12,foo
>>> result = pd.read_csv('examples/ex5.csv')

>>> result
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     foo

>>> pd.isnull(result)
   something      a      b      c      d  message
0      False  False  False  False  False     True
1      False  False  False   True  False    False
2      False  False  False  False  False    False
```

也可以通过`na_values`来指定缺失的值

```python
# 认为NULL为NaN
>>> result = pd.read_csv('examples/ex5.csv', na_values=['NULL'])

>>> result
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     foo

>>> sentinels = {'message': ['foo', 'NA'], 'something': ['two']}

# 通过字典，可以为每一列设置NaN值
>>> pd.read_csv('examples/ex5.csv', na_values=sentinels)
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       NaN  5   6   NaN   8   world
2     three  9  10  11.0  12     NaN
```

读取文件的一部分

```python
# 设置只显示10行
>>> pd.options.display.max_rows = 10

>>> result = pd.read_csv('examples/ex6.csv')

>>> result
           one       two     three      four key
0     0.467976 -0.038649 -0.295344 -1.824726   L
1    -0.358893  1.404453  0.704965 -0.200638   B
2    -0.501840  0.659254 -0.421691 -0.057688   G
3     0.204886  1.074134  1.388361 -0.982404   R
4     0.354628 -0.133116  0.283763 -0.837063   Q
...        ...       ...       ...       ...  ..
9995  2.311896 -0.417070 -1.409599 -0.515821   L
9996 -0.479893 -0.650419  0.745152 -0.646038   E
9997  0.523331  0.787112  0.486066  1.093156   K
9998 -0.362559  0.598894 -1.843201  0.887292   G
9999 -0.096376 -1.012999 -0.657431 -0.573315   0

[10000 rows x 5 columns]
```

分块读取文件

```python
>>> chunker = pd.read_csv('examples/ex6.csv', chunksize=1000)

>>> chunker
<pandas.io.parsers.TextFileReader at 0x1136a62b0>

# 通过遍历chunker，可以读取整个文件，1次读取chunksize行
>>> [type(piece) for piece in chunker]
[pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame,
 pandas.core.frame.DataFrame]
```

### 保存数据

保存到CSV文件

```python
>>> data = pd.read_csv('examples/ex5.csv')

>>> data
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     foo

>>> data.to_csv('examples/out.csv')

>>> !cat examples/out.csv
,something,a,b,c,d,message
0,one,1,2,3.0,4,
1,two,5,6,,8,world
2,three,9,10,11.0,12,foo
```

指定输出到标准输出和列分隔符

```python
>>> import sys

>>> data.to_csv(sys.stdout, sep='|')
|something|a|b|c|d|message
0|one|1|2|3.0|4|
1|two|5|6||8|world
2|three|9|10|11.0|12|foo
```

默认情况下，确实的数据是以空字符串输出的，我们也可以指定缺失的数据的输出内容

```python
>>> data.to_csv(sys.stdout, na_rep='NULL')
,something,a,b,c,d,message
0,one,1,2,3.0,4,NULL
1,two,5,6,NULL,8,world
2,three,9,10,11.0,12,foo
```

默认情况下，列索引和行索引都会写出到文件，也可以禁用这个行为

```python
# columns的索引用的是header参数
>>> data.to_csv(sys.stdout, na_rep='NULL', index=False, header=False)
one,1,2,3.0,4,NULL
two,5,6,NULL,8,world
three,9,10,11.0,12,foo
```

保存部分数据

```python
# 只保存a,b,c列
>>> data.to_csv(sys.stdout, index=False, columns=['a', 'b', 'c'])
a,b,c
1,2,3.0
5,6,
9,10,11.0
```
