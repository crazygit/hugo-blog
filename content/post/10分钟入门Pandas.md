---
title: 10分钟入门Pandas
date: 2017-12-20T22:21:46+08:00
tags:
    - pandas
categories:
    - python数据分析
slug: 10-minutes-to-pandas
description: >
    Pandas是python数据分析的瑞士军刀, 本文主要根据Pandas官网的10 Minutes to pandas整理而来

---


参考:

[10 Minutes to pandas](http://pandas.pydata.org/pandas-docs/stable/10min.html)

## 安装

支持的python版本: 2.7, 3.5, 3.6

```bash
$ pip install pandas
```

检查本地的pandas运行环境是否完整，可以运行pandas的单元测试用例

```bash
$ pip install pytest
```

```python
>>> import pandas as pd
>>> pd.test()
```
获取当前使用pandas的版本信息

```python
>>> import pandas as pd
>>> pd.__version__
'0.21.1'
```

<!--more-->

## 概览

pandas的基本数据结构:

* `Series`: 一维数据
* `DataFrame`: 二维数据
* `Panel`: 三维数据(从0.20.0版本开始，已经[不再推荐使用](http://pandas.pydata.org/pandas-docs/stable/dsintro.html#dsintro-deprecate-panel))
* `Panel4D`, `PanelND`(不再推荐使用)

`DataFrame`是由`Series`构成的


## 创建Series

创建`Series`最简单的方法

```python
>>> s = pd.Series(data, index=index)
```

`data`可以是不同的类型:

* python字典
* ndarray
* 标量(比如: 5)

### 使用`ndarray`创建(From ndarray)

如果`data`是`ndarray`,那么`index`的长度必须和`data`的长度相同，当没有明确`index`参数时，默认使用`[0, ... len(data) - 1]`作为`index`。

```python
>>> import pandas as pd

>>> import numpy as np

>>> s = pd.Series(np.random.randn(5), index=['a', 'b', 'c', 'd', 'e'])

>>> s
a    0.654385
b    0.055691
c    0.856054
d    0.621810
e    1.802872
dtype: float64

>>> s.index
Index(['a', 'b', 'c', 'd', 'e'], dtype='object')

>>> pd.Series(np.random.randn(5))
0   -0.467183
1   -1.333323
2   -0.493813
3   -0.067705
4   -1.310332
dtype: float64
```

需要注意的是: `pandas`里的索引并不要求唯一性，如果一个操作不支持重复的索引，会自动抛出异常。这么做的原因是很多操作不会用到索引，比如`GroupBy`。

```python
>>> s = pd.Series(np.random.randn(5), index=['a', 'a', 'a', 'a', 'a'])

>>> s
a    0.847331
a   -2.138021
a   -0.364763
a   -0.603172
a    0.363691
dtype: float64
```

### 使用`dict`创建(From dict)

当`data`是`dict`类型时，如果指定了`index`参数，那么就使用`index`参数作为索引。否者，就使用排序后的`data`的`key`作为`index`。

```python
>>> d = {'b': 0., 'a': 1., 'c': 2.}

# 索引的值是排序后的
>>> pd.Series(d)
a    1.0
b    0.0
c    2.0
dtype: float64

# 字典中不存在的key, 直接赋值为NaN(Not a number)
>>> pd.Series(d, index=['b', 'c', 'd', 'a'])
b    0.0
c    2.0
d    NaN
a    1.0
dtype: float64
```

### 使用标量创建(From scalar value)

当`data`是标量时，必须提供`index`, 值会被重复到`index`的长度

```python
>>> pd.Series(5., index=['a', 'b', 'c', 'd', 'e'])
a    5.0
b    5.0
c    5.0
d    5.0
e    5.0
dtype: float64
```

## 创建DataFrame

DataFrame是一个二维的数据结构，可以看做是一个excel表格或一张SQL表，或者值为Series的字典。 跟Series一样，DataFrame也可以通过多种类型的数据结构来创建

* 字典(包含一维ndarray数组，列表，字典或Series)
* 二维的ndarray数组
* 结构化的ndarray
* Series
* 另一个DataFrame

除了`data`之外，还接受`index`和`columns`参数来分布指定行和列的标签


### 从Series字典或嵌套的字典创建(From dict of Series or dicts)

结果的索引是多个Series索引的合集，如果没有指定`columns`，就用排序后的字典的`key`作为列标签。

```python
>>> d = {'one': pd.Series([1,2,3], index=['a', 'b', 'c']),
...      'two': pd.Series([1,2,3,4], index=['a', 'b', 'c', 'd'])}
...

>>> df = pd.DataFrame(d)

>>> df
   one  two
a  1.0    1
b  2.0    2
c  3.0    3
d  NaN    4

>>> pd.DataFrame(d, index=['d', 'b', 'a'])
   one  two
d  NaN    4
b  2.0    2
a  1.0    1

>>> pd.DataFrame(d, index=['d', 'b', 'a'], columns=['two', 'three'])
   two three
d    4   NaN
b    2   NaN
a    1   NaN

>>> df.index
Index(['a', 'b', 'c', 'd'], dtype='object')

>>> df.columns
Index(['one', 'two'], dtype='object')
```

### 从ndarray类型/列表类型的字典(From dict of ndarrays / lists)

```python
>>> d = {'one': [1,2,3,4], 'two': [4,3,2,1]}

>>> pd.DataFrame(d)
   one  two
0    1    4
1    2    3
2    3    2
3    4    1

>>> pd.DataFrame(d, index=['a', 'b', 'c', 'd'])
   one  two
a    1    4
b    2    3
c    3    2
d    4    1
```

### 从结构化ndarray创建(From structured or record array)

```python
>>> data = np.zeros((2, ), dtype=[('A', 'i4'), ('B', 'f4'), ('C', 'a10')])

>>> data
array([(0,  0., b''), (0,  0., b'')],
      dtype=[('A', '<i4'), ('B', '<f4'), ('C', 'S10')])

>>> data[:] = [(1, 2., 'Hello'), (2, 3., 'World')]

>>> pd.DataFrame(data)
   A    B         C
0  1  2.0  b'Hello'
1  2  3.0  b'World'

>>> pd.DataFrame(data, index=['first', 'second'])
    A    B         C
first   1  2.0  b'Hello'
second  2  3.0  b'World'

>>> pd.DataFrame(data, index=['first', 'second'], columns=['C', 'A', 'B'])
               C  A    B
first   b'Hello'  1  2.0
second  b'World'  2  3.0
```

### 从字典列表里创建(a list of dicts)

```python
>>> data2 = [{"a": 1, "b": 2}, {"a": 5, "b": 10, "c": 20}]

>>> pd.DataFrame(data2)
   a   b     c
0  1   2   NaN
1  5  10  20.0

>>> pd.DataFrame(data2, index=["first", "second"])
        a   b     c
first   1   2   NaN
second  5  10  20.0

>>> pd.DataFrame(data2, columns=["a", "b"])
   a   b
0  1   2
1  5  10
```

### 从元祖字典创建（From a dict of tuples）

通过元祖字典，可以创建多索引的DataFrame

```python
>>> pd.DataFrame({('a', 'b'): {('A', 'B'): 1, ('A', 'C'): 2},
...               ('a', 'a'): {('A', 'C'): 3, ('A', 'B'): 4},
...               ('a', 'c'): {('A', 'B'): 5, ('A', 'C'): 6},
...               ('b', 'a'): {('A', 'C'): 7, ('A', 'B'): 8},
...               ('b', 'b'): {('A', 'D'): 9, ('A', 'B'): 10}})
...
       a              b
       a    b    c    a     b
A B  4.0  1.0  5.0  8.0  10.0
  C  3.0  2.0  6.0  7.0   NaN
  D  NaN  NaN  NaN  NaN   9.0
```

### 通过Series创建(From a Series)

```python
>>> pd.DataFrame(pd.Series([1,2,3]))
   0
0  1
1  2
2  3
```

## 查看数据

```python
>>> dates = pd.date_range('20130101', periods=6)

>>> dates
DatetimeIndex(['2013-01-01', '2013-01-02', '2013-01-03', '2013-01-04',
               '2013-01-05', '2013-01-06'],
              dtype='datetime64[ns]', freq='D')

>>> df = pd.DataFrame(np.random.randn(6,4),index=dates,columns=list('ABCD'))

>>> df
                   A         B         C         D
2013-01-01  1.231897 -0.169839  1.333295  0.367142
2013-01-02 -0.127450 -1.716671  0.910350  0.151186
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639
2013-01-04  0.044990 -0.255158 -1.213848  1.076715
2013-01-05  0.418213  0.107400  0.619448  1.494087
2013-01-06 -1.831020  0.813526  0.403101 -1.251946

# 获取前几行(默认前5行)
>>> df.head()
                   A         B         C         D
2013-01-01  1.231897 -0.169839  1.333295  0.367142
2013-01-02 -0.127450 -1.716671  0.910350  0.151186
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639
2013-01-04  0.044990 -0.255158 -1.213848  1.076715
2013-01-05  0.418213  0.107400  0.619448  1.494087

# 获取后3行
>>> df.tail(3)
                   A         B         C         D
2013-01-04  0.044990 -0.255158 -1.213848  1.076715
2013-01-05  0.418213  0.107400  0.619448  1.494087
2013-01-06 -1.831020  0.813526  0.403101 -1.251946

# 获取索引
>>> df.index
DatetimeIndex(['2013-01-01', '2013-01-02', '2013-01-03', '2013-01-04',
               '2013-01-05', '2013-01-06'],
              dtype='datetime64[ns]', freq='D')

# 获取列信息
>>> df.columns
Index(['A', 'B', 'C', 'D'], dtype='object')

# 获取数据信息
>>> df.values
array([[ 1.23189704, -0.16983942,  1.3332949 ,  0.36714191],
       [-0.12744988, -1.71667129,  0.91034961,  0.15118638],
       [-0.24165226, -0.98464711,  0.78865554, -0.20363944],
       [ 0.04498958, -0.25515787, -1.21384804,  1.07671506],
       [ 0.41821265,  0.10740007,  0.61944799,  1.49408712],
       [-1.8310196 ,  0.81352564,  0.40310115, -1.25194611]])

# 获取简单的统计信息
>>> df.describe()
              A         B         C         D
count  6.000000  6.000000  6.000000  6.000000
mean  -0.084170 -0.367565  0.473500  0.272257
std    1.007895  0.880134  0.883494  0.970912
min   -1.831020 -1.716671 -1.213848 -1.251946
25%   -0.213102 -0.802275  0.457188 -0.114933
50%   -0.041230 -0.212499  0.704052  0.259164
75%    0.324907  0.038090  0.879926  0.899322
max    1.231897  0.813526  1.333295  1.494087

# 转置矩阵
>>> df.T
   2013-01-01  2013-01-02  2013-01-03  2013-01-04  2013-01-05  2013-01-06
A    1.231897   -0.127450   -0.241652    0.044990    0.418213   -1.831020
B   -0.169839   -1.716671   -0.984647   -0.255158    0.107400    0.813526
C    1.333295    0.910350    0.788656   -1.213848    0.619448    0.403101
D    0.367142    0.151186   -0.203639    1.076715    1.494087   -1.251946

# 按照列排序
>>> df.sort_values(by='B')
                  A         B         C         D
2013-01-02 -0.127450 -1.716671  0.910350  0.151186
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639
2013-01-04  0.044990 -0.255158 -1.213848  1.076715
2013-01-01  1.231897 -0.169839  1.333295  0.367142
2013-01-05  0.418213  0.107400  0.619448  1.494087
2013-01-06 -1.831020  0.813526  0.403101 -1.251946
```

## 选择数据


### 获取

选择列， 返回的是Series

```python
>>> df['A']
2013-01-01    1.231897
2013-01-02   -0.127450
2013-01-03   -0.241652
2013-01-04    0.044990
2013-01-05    0.418213
2013-01-06   -1.831020
Freq: D, Name: A, dtype: float64

>>> df.A
2013-01-01    1.231897
2013-01-02   -0.127450
2013-01-03   -0.241652
2013-01-04    0.044990
2013-01-05    0.418213
2013-01-06   -1.831020
Freq: D, Name: A, dtype: float64
```

选择行

```python
>>> df[0:3]
                   A         B         C         D
2013-01-01  1.231897 -0.169839  1.333295  0.367142
2013-01-02 -0.127450 -1.716671  0.910350  0.151186
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639

>>> df["20130102":"20130104"]
                   A         B         C         D
2013-01-02 -0.127450 -1.716671  0.910350  0.151186
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639
2013-01-04  0.044990 -0.255158 -1.213848  1.076715
```

### 通过Label选择

```python
# 返回的Series
>>> df.loc[dates[0]]
A    1.231897
B   -0.169839
C    1.333295
D    0.367142
Name: 2013-01-01 00:00:00, dtype: float64

# 返回的DateFrame
>>> df.loc[:, ['A', 'B']]
                  A         B
2013-01-01  1.231897 -0.169839
2013-01-02 -0.127450 -1.716671
2013-01-03 -0.241652 -0.984647
2013-01-04  0.044990 -0.255158
2013-01-05  0.418213  0.107400
2013-01-06 -1.831020  0.813526

>>> df.loc['20130102':'20130104',['A','B']]
                   A         B
2013-01-02 -0.127450 -1.716671
2013-01-03 -0.241652 -0.984647
2013-01-04  0.044990 -0.255158

# 降维返回
>>> df.loc['20130102',['A','B']]
A   -0.127450
B   -1.716671
Name: 2013-01-02 00:00:00, dtype: float64
```

### 通过Position选择

```python
# 返回第4行
>>> df.iloc[3]
A    0.044990
B   -0.255158
C   -1.213848
D    1.076715
Name: 2013-01-04 00:00:00, dtype: float64


>>> df.iloc[3:5,0:2]
                   A         B
2013-01-04  0.044990 -0.255158
2013-01-05  0.418213  0.107400

>>> df.iloc[1:3, :]
                   A         B         C         D
2013-01-02 -0.127450 -1.716671  0.910350  0.151186
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639

# 获得指定位置的元素
>>> df.iloc[1,1]
-1.7166712884342545

>>> df.iat[1,1]
-1.7166712884342545
```

### 布尔索引

```python
>>> df[df.A > 0]
                   A         B         C         D
2013-01-01  1.231897 -0.169839  1.333295  0.367142
2013-01-04  0.044990 -0.255158 -1.213848  1.076715
2013-01-05  0.418213  0.107400  0.619448  1.494087


>>> df[df > 0]
                   A         B         C         D
2013-01-01  1.231897       NaN  1.333295  0.367142
2013-01-02       NaN       NaN  0.910350  0.151186
2013-01-03       NaN       NaN  0.788656       NaN
2013-01-04  0.044990       NaN       NaN  1.076715
2013-01-05  0.418213  0.107400  0.619448  1.494087
2013-01-06       NaN  0.813526  0.403101       NaN


>>> df2=df.copy()

>>> df2['E'] = ['one','one','two','three','four','three']

>>> df2
                   A         B         C         D      E
2013-01-01  1.231897 -0.169839  1.333295  0.367142    one
2013-01-02 -0.127450 -1.716671  0.910350  0.151186    one
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639    two
2013-01-04  0.044990 -0.255158 -1.213848  1.076715  three
2013-01-05  0.418213  0.107400  0.619448  1.494087   four
2013-01-06 -1.831020  0.813526  0.403101 -1.251946  three

# 使用isin()来过滤
>>> df2[df2['E'].isin(['two', 'four'])]
                   A         B         C         D     E
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639   two
2013-01-05  0.418213  0.107400  0.619448  1.494087  four
```

### 赋值

根据日期新增加一列

```python
>>> s1
2013-01-02    1
2013-01-03    2
2013-01-04    3
2013-01-05    4
2013-01-06    5
2013-01-07    6
Freq: D, dtype: int64

>>> df['F'] = s1

>>> df
                   A         B         C         D    F
2013-01-01  1.231897 -0.169839  1.333295  0.367142  NaN
2013-01-02 -0.127450 -1.716671  0.910350  0.151186  1.0
2013-01-03 -0.241652 -0.984647  0.788656 -0.203639  2.0
2013-01-04  0.044990 -0.255158 -1.213848  1.076715  3.0
2013-01-05  0.418213  0.107400  0.619448  1.494087  4.0
2013-01-06 -1.831020  0.813526  0.403101 -1.251946  5.0

# 通过label赋值
>>> df.at[dates[0], 'A'] = 0

# 通过position赋值
>>> df.iat[0,1] = 0

# 通过ndarray赋值
>>> df.loc[:, 'D'] = np.array([5] * len(df))

>>> df
                   A         B         C  D    F
2013-01-01  0.000000  0.000000  1.333295  5  NaN
2013-01-02 -0.127450 -1.716671  0.910350  5  1.0
2013-01-03 -0.241652 -0.984647  0.788656  5  2.0
2013-01-04  0.044990 -0.255158 -1.213848  5  3.0
2013-01-05  0.418213  0.107400  0.619448  5  4.0
2013-01-06 -1.831020  0.813526  0.403101  5  5.0

# 通过where操作
>>> df = pd.DataFrame(np.random.randn(6,4),index=dates,columns=list('ABCD'))

>>> df
                   A         B         C         D
2013-01-01 -1.231777 -0.068987 -0.105402  1.512076
2013-01-02 -1.120426 -0.240417  0.223964 -0.559793
2013-01-03  0.697097  0.758780 -1.191408 -0.793882
2013-01-04  0.332519  0.784564  0.805932 -1.169186
2013-01-05  0.010235  0.156115  0.419567 -2.279214
2013-01-06  0.294819 -0.691370  0.294119 -0.208475

>>> df2 = df.copy()

>>> df2[df > 0] = -df2

>>> df2
                   A         B         C         D
2013-01-01 -1.231777 -0.068987 -0.105402 -1.512076
2013-01-02 -1.120426 -0.240417 -0.223964 -0.559793
2013-01-03 -0.697097 -0.758780 -1.191408 -0.793882
2013-01-04 -0.332519 -0.784564 -0.805932 -1.169186
2013-01-05 -0.010235 -0.156115 -0.419567 -2.279214
2013-01-06 -0.294819 -0.691370 -0.294119 -0.208475
```

## 数据缺失

pandas使用`np.nan`来表示缺失的数据，它默认不参与任何运算

```python
>>> df1 = df.reindex(index=dates[0:4], columns=list(df.columns) + ['E'])

>>> df1
                   A         B         C  D    F   E
2013-01-01  0.000000  0.000000  1.333295  5  NaN NaN
2013-01-02 -0.127450 -1.716671  0.910350  5  1.0 NaN
2013-01-03 -0.241652 -0.984647  0.788656  5  2.0 NaN
2013-01-04  0.044990 -0.255158 -1.213848  5  3.0 NaN

>>> df1.loc[dates[0]:dates[1], 'E'] = 1

>>> df1
                   A         B         C  D    F    E
2013-01-01  0.000000  0.000000  1.333295  5  NaN  1.0
2013-01-02 -0.127450 -1.716671  0.910350  5  1.0  1.0
2013-01-03 -0.241652 -0.984647  0.788656  5  2.0  NaN
2013-01-04  0.044990 -0.255158 -1.213848  5  3.0  NaN

# 丢弃所有包含NaN的行
>>> df1.dropna(how='any')
                  A         B        C  D    F    E
2013-01-02 -0.12745 -1.716671  0.91035  5  1.0  1.0

# 填充所有包含NaN的元素
>>> df1.fillna(value=5)
                   A         B         C  D    F    E
2013-01-01  0.000000  0.000000  1.333295  5  5.0  1.0
2013-01-02 -0.127450 -1.716671  0.910350  5  1.0  1.0
2013-01-03 -0.241652 -0.984647  0.788656  5  2.0  5.0
2013-01-04  0.044990 -0.255158 -1.213848  5  3.0  5.0

# 获取元素值为nan的布尔掩码
>>> pd.isna(df1)
                A      B      C      D      F      E
2013-01-01  False  False  False  False   True  False
2013-01-02  False  False  False  False  False  False
2013-01-03  False  False  False  False  False   True
2013-01-04  False  False  False  False  False   True
```

## 运算操作

### Stats统计

运算操作都会排除`NaN`元素

```python
>>> dates = pd.date_range('20130101', periods=6)

>>> df = pd.DataFrame(np.arange(24).reshape(6,4),index=dates,columns=list('ABCD'))

>>> df
             A   B   C   D
2013-01-01   0   1   2   3
2013-01-02   4   5   6   7
2013-01-03   8   9  10  11
2013-01-04  12  13  14  15
2013-01-05  16  17  18  19
2013-01-06  20  21  22  23

# 计算列的平均值
>>> df.mean()
A    10.0
B    11.0
C    12.0
D    13.0
dtype: float64

计算行的平均值
>>> df.mean(1)
2013-01-01     1.5
2013-01-02     5.5
2013-01-03     9.5
2013-01-04    13.5
2013-01-05    17.5
2013-01-06    21.5
Freq: D, dtype: float64

# shift(n),按照列的方向，从上往下移动n个位置
>>> s = pd.Series([1,3,5,np.nan,6,8], index=dates).shift(2)

>>> s
2013-01-01    NaN
2013-01-02    NaN
2013-01-03    1.0
2013-01-04    3.0
2013-01-05    5.0
2013-01-06    NaN
Freq: D, dtype: float64

# sub函数,DataFrame相减操作, 等于 df-s
>>> df.sub(s, axis='index')
               A     B     C     D
2013-01-01   NaN   NaN   NaN   NaN
2013-01-02   NaN   NaN   NaN   NaN
2013-01-03   7.0   8.0   9.0  10.0
2013-01-04   9.0  10.0  11.0  12.0
2013-01-05  11.0  12.0  13.0  14.0
2013-01-06   NaN   NaN   NaN   NaN
```

### Apply

```python
>>> df
             A   B   C   D
2013-01-01   0   1   2   3
2013-01-02   4   5   6   7
2013-01-03   8   9  10  11
2013-01-04  12  13  14  15
2013-01-05  16  17  18  19
2013-01-06  20  21  22  23

# 在列方向累加
>>> df.apply(np.cumsum)
             A   B   C   D
2013-01-01   0   1   2   3
2013-01-02   4   6   8  10
2013-01-03  12  15  18  21
2013-01-04  24  28  32  36
2013-01-05  40  45  50  55
2013-01-06  60  66  72  78

# 列方向的最大值-最小值， 得到的是一个Series
>>> df.apply(lambda x: x.max() - x.min())
A    20
B    20
C    20
D    20
dtype: int64
```

### 直方图 Histogramming

```python
>>> s = pd.Series(np.random.randint(0, 7, size=10))

>>> s
0    6
1    5
2    0
3    2
4    5
5    1
6    3
7    3
8    3
9    1
dtype: int64

# 索引是出现的数字，值是次数
>>> s.value_counts()
3    3
5    2
1    2
6    1
2    1
0    1
dtype: int64
```

### 字符串方法

```python
>>> s = pd.Series(['A', 'B', 'C', 'Aaba', 'Baca', np.nan, 'CABA', 'dog', 'cat'])

>>> s.str.lower()
0       a
1       b
2       c
3    aaba
4    baca
5     NaN
6    caba
7     dog
8     cat
dtype: object
```

## 合并

### [Concat](http://pandas.pydata.org/pandas-docs/stable/merging.html#merging)

```python
>>> df = pd.DataFrame(np.random.randn(10, 4))

>>> df
          0         1         2         3
0 -1.710767 -2.107488  1.441790  0.959924
1  0.509422  0.099733  0.845039  0.232462
2 -0.609247  0.533162 -0.387640  0.668803
3  0.946219 -0.326805  1.245303  1.336090
4 -1.069114  0.755313 -1.003991 -0.327009
5  1.169418 -1.225637 -2.137500  1.766341
6 -1.751095  0.279439  0.018053  1.800435
7 -0.328828 -1.513893  1.879333  0.945217
8  2.440123 -0.260918 -0.232951 -1.337775
9 -0.876878 -1.153583 -1.487573 -1.509871

# 分成小块
>>> pieces = [df[:3], df[3:7], df[7:]]

# 合并
>>> pd.concat(pieces)
          0         1         2         3
0 -1.710767 -2.107488  1.441790  0.959924
1  0.509422  0.099733  0.845039  0.232462
2 -0.609247  0.533162 -0.387640  0.668803
3  0.946219 -0.326805  1.245303  1.336090
4 -1.069114  0.755313 -1.003991 -0.327009
5  1.169418 -1.225637 -2.137500  1.766341
6 -1.751095  0.279439  0.018053  1.800435
7 -0.328828 -1.513893  1.879333  0.945217
8  2.440123 -0.260918 -0.232951 -1.337775
9 -0.876878 -1.153583 -1.487573 -1.509871
```

### [Join](http://pandas.pydata.org/pandas-docs/stable/merging.html#merging-join)

跟数据库的Join操作一样

```python
>>> left = pd.DataFrame({'key': ['foo', 'foo'], 'lval': [1, 2]})

>>> left
   key  lval
0  foo     1
1  foo     2

>>> right = pd.DataFrame({'key': ['foo', 'foo'], 'rval': [4, 5]})

>>> right
   key  rval
0  foo     4
1  foo     5

>>> pd.merge(left, right, on='key')
   key  lval  rval
0  foo     1     4
1  foo     1     5
2  foo     2     4
3  foo     2     5
```

另一个例子

```python
>>> left = pd.DataFrame({'key': ['foo', 'bar'], 'lval': [1, 2]})

>>> left
   key  lval
0  foo     1
1  bar     2

>>> right = pd.DataFrame({'key': ['foo', 'bar'], 'rval': [4, 5]})

>>> right
   key  rval
0  foo     4
1  bar     5

>>> pd.merge(left, right, on='key')
   key  lval  rval
0  foo     1     4
1  bar     2     5
```

### [Append](http://pandas.pydata.org/pandas-docs/stable/merging.html#merging-concatenation)

```python
>>> df = pd.DataFrame(np.random.randn(8, 4), columns=['A','B','C','D'])

>>> df
          A         B         C         D
0 -1.521762 -0.850721  1.322354 -0.226562
1 -2.773304 -0.663303  0.895075 -0.171524
2  0.322975 -0.796484  0.379920  0.028333
3 -0.350795  1.839747 -0.359241 -0.027921
4 -0.945340  1.062598 -2.208670  0.769027
5 -0.329458 -0.145658  1.580258 -1.414820
6 -0.261757 -1.435025 -0.512306 -0.222287
7 -0.994207 -1.219057  0.781283 -1.795741

>>> s = df.iloc[3]

>>> df.append(s, ignore_index=True)
          A         B         C         D
0 -1.521762 -0.850721  1.322354 -0.226562
1 -2.773304 -0.663303  0.895075 -0.171524
2  0.322975 -0.796484  0.379920  0.028333
3 -0.350795  1.839747 -0.359241 -0.027921
4 -0.945340  1.062598 -2.208670  0.769027
5 -0.329458 -0.145658  1.580258 -1.414820
6 -0.261757 -1.435025 -0.512306 -0.222287
7 -0.994207 -1.219057  0.781283 -1.795741
8 -0.350795  1.839747 -0.359241 -0.027921
```


## [Grouping](http://pandas.pydata.org/pandas-docs/stable/groupby.html#groupby)

`group by`的操作需要经过以下1个或多个步骤

* 根据条件分组数据(**Spliting**)
* 在各个分组上执行函数(**Applying**)
* 合并结果(**Combining**)

```python
>>> df = pd.DataFrame({'A' : ['foo', 'bar', 'foo', 'bar',
...                           'foo', 'bar', 'foo', 'foo'],
...                    'B' : ['one', 'one', 'two', 'three',
...                           'two', 'two', 'one', 'three'],
...                    'C' : np.arange(1, 9),
 ...                    'D' : np.arange(2, 10)})
...
...

>>> df
     A      B  C  D
0  foo    one  1  2
1  bar    one  2  3
2  foo    two  3  4
3  bar  three  4  5
4  foo    two  5  6
5  bar    two  6  7
6  foo    one  7  8
7  foo  three  8  9

# 分组求和
>>> df.groupby('A').sum()
      C   D
A
bar  12  15
foo  24  29

# 多列分组
>>> df.groupby(['A','B']).sum()
           C   D
A   B
bar one    2   3
    three  4   5
    two    6   7
foo one    8  10
    three  8   9
    two    8  10

>>> b = df.groupby(['A','B']).sum()

# 多索引
>>> b.index
MultiIndex(levels=[['bar', 'foo'], ['one', 'three', 'two']],
           labels=[[0, 0, 0, 1, 1, 1], [0, 1, 2, 0, 1, 2]],
           names=['A', 'B'])

>>> b.columns
Index(['C', 'D'], dtype='object')
```

## [Reshaping](http://pandas.pydata.org/pandas-docs/stable/reshaping.html#reshaping-stacking)

### Stack


```python
>>> tuples = list(zip(*[['bar', 'bar', 'baz', 'baz',
...                      'foo', 'foo', 'qux', 'qux'],
...                     ['one', 'two', 'one', 'two',
...                      'one', 'two', 'one', 'two']]))
...

>>> tuples
[('bar', 'one'),
 ('bar', 'two'),
 ('baz', 'one'),
 ('baz', 'two'),
 ('foo', 'one'),
 ('foo', 'two'),
 ('qux', 'one'),
 ('qux', 'two')]

>>> index = pd.MultiIndex.from_tuples(tuples, names=['first', 'second'])

>>> df = pd.DataFrame(np.random.randn(8, 2), index=index, columns=['A', 'B'])

>>> df
                     A         B
first second
bar   one     0.096893  0.479194
      two    -0.771606  0.331693
baz   one    -0.022540  0.531284
      two    -0.039843  1.876942
foo   one     0.250473  1.163931
      two    -1.127163  1.447566
qux   one    -0.410361 -0.734333
      two    -0.461247  0.018531

>>> df2 = df[:4]

>>> df2
                     A         B
first second
bar   one     0.096893  0.479194
      two    -0.771606  0.331693
baz   one    -0.022540  0.531284
      two    -0.039843  1.876942

>>> stacked = df2.stack()

>>> stacked
first  second
bar    one     A    0.096893
               B    0.479194
       two     A   -0.771606
               B    0.331693
baz    one     A   -0.022540
               B    0.531284
       two     A   -0.039843
               B    1.876942
dtype: float64

>>> type(stacked)
pandas.core.series.Series

>>> stacked.index
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two'], ['A', 'B']],
           labels=[[0, 0, 0, 0, 1, 1, 1, 1], [0, 0, 1, 1, 0, 0, 1, 1], [0, 1, 0, 1, 0, 1, 0, 1]],
           names=['first', 'second', None])

>>> stacked.values
array([ 0.09689327,  0.47919417, -0.77160574,  0.3316934 , -0.02253955,
        0.53128436, -0.03984337,  1.8769416 ])


>>> stacked.unstack()
                     A         B
first second
bar   one     0.096893  0.479194
      two    -0.771606  0.331693
baz   one    -0.022540  0.531284
      two    -0.039843  1.876942

>>> stacked.unstack(1)
second        one       two
first
bar   A  0.096893 -0.771606
      B  0.479194  0.331693
baz   A -0.022540 -0.039843
      B  0.531284  1.876942

>>> stacked.unstack(0)
first          bar       baz
second
one    A  0.096893 -0.022540
       B  0.479194  0.531284
two    A -0.771606 -0.039843
       B  0.331693  1.876942
```


### [数据透视表(Pivot Tables)](http://pandas.pydata.org/pandas-docs/stable/reshaping.html#reshaping-pivot)

## [时间序列](http://pandas.pydata.org/pandas-docs/stable/timeseries.html#timeseries)

pandas在时间序列上，提供了很方便的按照频率重新采样的功能，在财务分析上非常有用

```python
# 把每秒的数据按5分钟聚合
>>> rng = pd.date_range('1/1/2012', periods=100, freq='S')
>>> ts = pd.Series(np.random.randint(0, 500, len(rng)), index=rng)
>>> ts.resample('5Min').sum()
2012-01-01    22073
Freq: 5T, dtype: int64
```

加上时区信息

```python
>>> rng = pd.date_range('3/6/2012 00:00', periods=5, freq='D')

>>> ts = pd.Series(np.random.randn(len(rng)), rng)

>>> ts
2012-03-06   -0.386974
2012-03-07    0.657785
2012-03-08    1.390234
2012-03-09    0.412904
2012-03-10   -1.189340
Freq: D, dtype: float64

>>> ts_utc = ts.tz_localize('UTC')

>>> ts_utc
2012-03-06 00:00:00+00:00   -0.386974
2012-03-07 00:00:00+00:00    0.657785
2012-03-08 00:00:00+00:00    1.390234
2012-03-09 00:00:00+00:00    0.412904
2012-03-10 00:00:00+00:00   -1.189340
Freq: D, dtype: float64
```

转换成另一个时区

```python
>>> ts_utc.tz_convert('Asia/Shanghai')
2012-03-06 08:00:00+08:00   -0.386974
2012-03-07 08:00:00+08:00    0.657785
2012-03-08 08:00:00+08:00    1.390234
2012-03-09 08:00:00+08:00    0.412904
2012-03-10 08:00:00+08:00   -1.189340
Freq: D, dtype: float64
```

时间跨度转换

```python
>>> rng = pd.date_range('1/1/2012', periods=5, freq='M')
>>> ts = pd.Series(np.random.randn(len(rng)), index=rng)

>>> ts
2012-01-31    0.825174
2012-02-29   -2.190258
2012-03-31   -0.073171
2012-04-30   -0.404208
2012-05-31    0.245025
Freq: M, dtype: float64

>>> ps = ts.to_period()

>>> ps
2012-01    0.825174
2012-02   -2.190258
2012-03   -0.073171
2012-04   -0.404208
2012-05    0.245025
Freq: M, dtype: float64

>>> ps.to_timestamp()
2012-01-01    0.825174
2012-02-01   -2.190258
2012-03-01   -0.073171
2012-04-01   -0.404208
2012-05-01    0.245025
Freq: MS, dtype: float64
```

转换季度时间

```python
>>> prng = pd.period_range('1990Q1', '2000Q4', freq='Q-NOV')

>>> ts = pd.Series(np.random.randn(len(prng)), prng)

>>> ts.head()
1990Q1   -0.590040
1990Q2   -0.750392
1990Q3   -0.385517
1990Q4   -0.380806
1991Q1   -1.252727
Freq: Q-NOV, dtype: float64

>>> ts.index = (prng.asfreq('M', 'e') + 1).asfreq('H', 's') + 9

>>> ts.head()
1990-03-01 09:00   -0.590040
1990-06-01 09:00   -0.750392
1990-09-01 09:00   -0.385517
1990-12-01 09:00   -0.380806
1991-03-01 09:00   -1.252727
Freq: H, dtype: float64
```

## [Categoricals分类](http://pandas.pydata.org/pandas-docs/stable/categorical.html#categorical)

```python
>>> df = pd.DataFrame({"id":[1,2,3,4,5,6], "raw_grade":['a', 'b', 'b', 'a', 'a', 'e']})

>>> df
   id raw_grade
0   1         a
1   2         b
2   3         b
3   4         a
4   5         a
5   6         e
```

转换原始类别为分类数据类型

```python
>>> df["grade"] = df["raw_grade"].astype("category")

>>> df
   id raw_grade grade
0   1         a     a
1   2         b     b
2   3         b     b
3   4         a     a
4   5         a     a
5   6         e     e

>>> df["grade"]
0    a
1    b
2    b
3    a
4    a
5    e
Name: grade, dtype: category
Categories (3, object): [a, b, e]
```

重命名分类为更有意义的名称

```python
>>> df["grade"].cat.categories = ["very good", "good", "very bad"]

>>> df
   id raw_grade      grade
0   1         a  very good
1   2         b       good
2   3         b       good
3   4         a  very good
4   5         a  very good
5   6         e   very bad
```

重新安排顺分类,同时添加缺少的分类(序列 .cat方法下返回新默认序列)

```python
>>> df["grade"] = df["grade"].cat.set_categories(["very bad", "bad", "medium", "good", "very good"])

>>> df
   id raw_grade      grade
0   1         a  very good
1   2         b       good
2   3         b       good
3   4         a  very good
4   5         a  very good
5   6         e   very bad

>>> df["grade"]
0    very good
1         good
2         good
3    very good
4    very good
5     very bad
Name: grade, dtype: category
Categories (5, object): [very bad, bad, medium, good, very good]
```

按照分类排序

```python
>>> df.sort_values(by="grade")
   id raw_grade      grade
5   6         e   very bad
1   2         b       good
2   3         b       good
0   1         a  very good
3   4         a  very good
4   5         a  very good
```

按照分类分组，同时也会显示空的分类

```python
>>> df.groupby("grade").size()
grade
very bad     1
bad          0
medium       0
good         2
very good    3
dtype: int64
```

## [Plotting](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization)

```python
>>> import matplotlib.pyplot as plt
>>> ts = pd.Series(np.random.randn(1000), index=pd.date_range('1/1/2000', periods=1000))
>>> ts = ts.cumsum()

>>> ts.plot()
<matplotlib.axes._subplots.AxesSubplot at 0x108594668>

>>> plt.show()
```

![曲线图](http://images.wiseturtles.com/2017-12-21-plot.png)

画图带图例的图

```python
>>> df = pd.DataFrame(np.random.randn(1000, 4), index=ts.index, columns=['A','B'
... ,'C', 'D'])

>>> df.cumsum()

>>> plt.figure();df.plot();plt.legend(loc='best')
<matplotlib.legend.Legend at 0x111793f98>

>>> plt.show()
```

![带图例的曲线图](http://images.wiseturtles.com/2017-12-21-plot-lines.png)



## 数据In/Out

### [CSV](http://pandas.pydata.org/pandas-docs/stable/io.html#csv-text-files)

保存到csv文件

```python
>>> df.to_csv('foo.csv')
```

从csv文件读取数据

```python
>>> pd.read_csv('foo.csv')
```

### [HDF5](http://pandas.pydata.org/pandas-docs/stable/io.html#hdf5-pytables)

保存到HDF5仓库

```python
>>> df.to_hdf('foo.h5','df')
```

从仓库读取

```python
>>> pd.read_hdf('foo.h5','df')
```

### [Excel](http://pandas.pydata.org/pandas-docs/stable/io.html#io-excel)

保存到excel

```python
>>> df.to_excel('foo.xlsx', sheet_name='Sheet1')
```

从excel文件读取

```python
 >>> pd.read_excel('foo.xlsx', 'Sheet1', index_col=None, na_values=['NA'])
```

## 扩展阅读

* [pandas指南](http://pandas.pydata.org/pandas-docs/stable/tutorials.html)
