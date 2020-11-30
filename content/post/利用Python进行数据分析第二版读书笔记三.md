---
title: "《利用Python进行数据分析第二版》读书笔记三"
date: 2018-01-04T21:33:14+08:00
tags:
    - pandas
categories:
    - python数据分析
slug: python-for-data-analysis-2nd-edition-note-two
description: >
    本文主要介绍使用Pandas进行数据合并，拼接和变型

---

# 数据合并，拼接和变型

## 分层索引

分层索引是`Pandas`的一个重要特性，能够让我们在一个轴上拥有多层索引。简单来说，它提供了一个在降纬模式下处理多维数据的能力。

如下是一个多层索引的例子

```python
>>> data = pd.Series(np.random.randn(9),
...                  index=[['a', 'a', 'a', 'b', 'b', 'c', 'c', 'd', 'd'], [1,2,3,1,3,1,2,2,3]])

>>> data
a  1    1.007189
   2   -1.296221
   3    0.274992
b  1    0.228913
   3    1.352917
c  1    0.886429
   2   -2.001637
d  2   -0.371843
   3    1.669025
dtype: float64

>>> data.index
MultiIndex(levels=[['a', 'b', 'c', 'd'], [1, 2, 3]],
           labels=[[0, 0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 2, 0, 2, 0, 1, 1, 2]])

# 访问外层索引
>>> data['b']
1    0.228913
3    1.352917
dtype: float64

>>> data['b':'c']
b  1    0.228913
   3    1.352917
c  1    0.886429
   2   -2.001637

# 访问内层索引
>>> data.loc[:, 2]
a   -1.296221
c   -2.001637
d   -0.371843
dtype: float64
```
<!--more-->

多层索引在数据变型和透视表中非常常见，可以使用`unstack`方法，将`Series`转换成`DataFrame`

```python
>>> data.unstack()
          1         2         3
a  1.007189 -1.296221  0.274992
b  0.228913       NaN  1.352917
c  0.886429 -2.001637       NaN
d       NaN -0.371843  1.669025
```

与`unstack`方法相反作用的就是`stack`

```python
>>> data.unstack().stack()
a  1    1.007189
   2   -1.296221
   3    0.274992
b  1    0.228913
   3    1.352917
c  1    0.886429
   2   -2.001637
d  2   -0.371843
   3    1.669025
dtype: float64
```

`DataFrame`的任何一个轴都可以有多个索引

```python
>>> frame = pd.DataFrame(np.arange(12).reshape((4, 3)), index=[['a', 'a', 'b','b'], [1, 2, 1, 2]], columns=[['Ohio', 'Ohio', 'Colorado'],  ['Green', 'Red', 'Green']])

>>> frame
     Ohio     Colorado
    Green Red    Green
a 1     0   1        2
  2     3   4        5
b 1     6   7        8
  2     9  10       11

# 可以给每层索引取一个名字
>>> frame.index.names = ["key1", "key2"]

>>> frame.columns.names = ["state", "color"]

>>> frame
state      Ohio     Colorado
color     Green Red    Green
key1 key2
a    1        0   1        2
     2        3   4        5
b    1        6   7        8
     2        9  10       11

# 通过外层索引选择列
>>> frame['Ohio']
color      Green  Red
key1 key2
a    1         0    1
     2         3    4
b    1         6    7
     2         9   10

# 通过内层索引选择列
>>> frame['Ohio']['Green']
key1  key2
a     1       0
      2       3
b     1       6
      2       9
```

### 分层排序和分类

使用`swaplevel`，我们可以重新对一个轴上的多层索引排序

```python
>>> frame
state      Ohio     Colorado
color     Green Red    Green
key1 key2
a    1        0   1        2
     2        3   4        5
b    1        6   7        8
     2        9  10       11

>>> frame.swaplevel('key1', 'key2')
state      Ohio     Colorado
color     Green Red    Green
key2 key1
1    a        0   1        2
2    a        3   4        5
1    b        6   7        8
2    b        9  10       11
```

我们也可以在一个分层上对数据进行分类

```python
>>> frame.sort_index(level=1)
state      Ohio     Colorado
color     Green Red    Green
key1 key2
a    1        0   1        2
b    1        6   7        8
a    2        3   4        5
b    2        9  10       11

>>> frame.swaplevel(0, 1).sort_index(level=0)
state      Ohio     Colorado
color     Green Red    Green
key2 key1
1    a        0   1        2
     b        6   7        8
2    a        3   4        5
     b        9  10       11
```

### 分层统计

许多关于`Series`和`DataFrame`的统计操作，都接受一个`level`参数来对层上进行统计

```python
>>> frame
state      Ohio     Colorado
color     Green Red    Green
key1 key2
a    1        0   1        2
     2        3   4        5
b    1        6   7        8
     2        9  10       11

>>> frame.sum(level='key2')
state  Ohio     Colorado
color Green Red    Green
key2
1         6   8       10
2        12  14       16

>>> frame.sum(level='color', axis=1)
color      Green  Red
key1 key2
a    1         2    1
     2         8    4
b    1        14    7
     2        20   10
```

### 利用DataFrame的列作为索引

使用`set_index`方法，可以将`DataFrame`的列作为索引

```python
>>> frame = pd.DataFrame({'a': range(7), 'b': range(7, 0, -1),
...                        'c': ['one', 'one', 'one', 'two', 'two',
...                                     'two', 'two'],
...                       'd':[0,1,2,0,1,2,3]})
...

>>> frame
   a  b    c  d
0  0  7  one  0
1  1  6  one  1
2  2  5  one  2
3  3  4  two  0
4  4  3  two  1
5  5  2  two  2
6  6  1  two  3

>>> frame2 = frame.set_index(['c', 'd'])

>>> frame2
       a  b
c   d
one 0  0  7
    1  1  6
    2  2  5
two 0  3  4
    1  4  3
    2  5  2
    3  6  1

>>> frame2.index
MultiIndex(levels=[['one', 'two'], [0, 1, 2, 3]],
           labels=[[0, 0, 0, 1, 1, 1, 1], [0, 1, 2, 0, 1, 2, 3]],
           names=['c', 'd'])

# 默认情况下，作为索引的列会自动从DataFrame里移除，可以通过drop参数保留它们
>>> frame.set_index(['c', 'd'], drop=False)
       a  b    c  d
c   d
one 0  0  7  one  0
    1  1  6  one  1
    2  2  5  one  2
two 0  3  4  two  0
    1  4  3  two  1
    2  5  2  two  2
    3  6  1  two  3
```

`reset_index`函数的与`set_index`函数相反，多层索引会被放到列里面去

```python
>>> frame2.reset_index()
     c  d  a  b
0  one  0  0  7
1  one  1  1  6
2  one  2  2  5
3  two  0  3  4
4  two  1  4  3
5  two  2  5  2
6  two  3  6  1
```

## 合并数据集

数据合并操作主要有以下几个方法:

* `pandas.merge`: 基于key来合并row，类似关系型数据库的`JOIN`操作
* `pandas.concat`: 在轴上合并数据，类似`stack`操作
* `combine_first`: 合并重复的数据和填充确实的数据

### 类似数据库的DataFrame JOIN操作

如下一个"多对一"的例子，在`df1`里有多行值是a和b, 而在`df2`里面各自只有一行。

```python
>>> df1 = pd.DataFrame({'key':['b', 'b', 'a', 'c', 'a', 'a', 'b'],
...                    'data1': range(7)})
...

>>> df2 = pd.DataFrame({'key': ['a', 'b', 'd'],
...                     'data2': range(3)})
...

>>> df1
   data1 key
0      0   b
1      1   b
2      2   a
3      3   c
4      4   a
5      5   a
6      6   b

>>> df2
   data2 key
0      0   a
1      1   b
2      2   d

# 默认是inner Join
>>> pd.merge(df1, df2)
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0
```

上面的例子里，我们并没有指明以哪一列作为join on的条件。merge函数默认使用列名相同的列作为key。最好的使用建议是通过`on`参数明确指明key信息

```python
>>> pd.merge(df1, df2, on='key')
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0
```

如果两个DataFrame的列名都不相同，我们也可以单独指定它们

```python
>>> df3 = pd.DataFrame({'lkey': ['b', 'b', 'a', 'c', 'a', 'a', 'b'],
...                     'data1': range(7)})
...

>>> df4 = pd.DataFrame({'rkey': ['a', 'b', 'd'],
...                      'data2': range(3)})
...

>>> pd.merge(df3, df4, left_on='lkey', right_on='rkey')
   data1 lkey  data2 rkey
0      0    b      1    b
1      1    b      1    b
2      6    b      1    b
3      2    a      0    a
4      4    a      0    a
5      5    a      0    a
```

`merge`的默认方式是`inner join`，我们还可以指定join方式为`left`, `right`, `outer`

```python
>>> df1
   data1 key
0      0   b
1      1   b
2      2   a
3      3   c
4      4   a
5      5   a
6      6   b

>>> df2
   data2 key
0      0   a
1      1   b
2      2   d

>>> pd.merge(df1, df2, how='inner')
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0

>>> pd.merge(df1, df2, how='outer')
   data1 key  data2
0    0.0   b    1.0
1    1.0   b    1.0
2    6.0   b    1.0
3    2.0   a    0.0
4    4.0   a    0.0
5    5.0   a    0.0
6    3.0   c    NaN
7    NaN   d    2.0

>>> pd.merge(df1, df2, how='left')
   data1 key  data2
0      0   b    1.0
1      1   b    1.0
2      2   a    0.0
3      3   c    NaN
4      4   a    0.0
5      5   a    0.0
6      6   b    1.0

>>> pd.merge(df1, df2, how='right')
   data1 key  data2
0    0.0   b      1
1    1.0   b      1
2    6.0   b      1
3    2.0   a      0
4    4.0   a      0
5    5.0   a      0
6    NaN   d      2
```

不同的Join方式表现行为如下图:

![不同的Join方式](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-03-join_way.png)

“多对多”的方式, 默认情况下，多对多的结果是“笛卡尔乘积”。
如下: `df1`的`key`列有3个`b`, `df2`的`key`列有2个`b`。因此结果里会有`3x2=6`个b。 不同的join方式只是会影响结果里key的出现个数，不会影响“笛卡尔乘积”的行为

```python
>>> df1 = pd.DataFrame({'key': ['b', 'b', 'a', 'c', 'a', 'b'],
...  'data1': range(6)})
...

>>> df2 = pd.DataFrame({'key': ['a', 'b', 'a', 'b', 'd'],
...                    'data2': range(5)})
...

>>> df1
   data1 key
0      0   b
1      1   b
2      2   a
3      3   c
4      4   a
5      5   b

>>> df2
   data2 key
0      0   a
1      1   b
2      2   a
3      3   b
4      4   d

>>> pd.merge(df1, df2, on='key', how='left')
    data1 key  data2
0       0   b    1.0
1       0   b    3.0
2       1   b    1.0
3       1   b    3.0
4       2   a    0.0
5       2   a    2.0
6       3   c    NaN
7       4   a    0.0
8       4   a    2.0
9       5   b    1.0
10      5   b    3.0

# 不同的join方式，结果里的key不一样，单都是笛卡尔乘积结果
>>> pd.merge(df1, df2, on='key', how='inner')
   data1 key  data2
0      0   b      1
1      0   b      3
2      1   b      1
3      1   b      3
4      5   b      1
5      5   b      3
6      2   a      0
7      2   a      2
8      4   a      0
9      4   a      2
```


也可以根据多个key来进行合并操作

```python
>>> left = pd.DataFrame({'key1': ['foo', 'foo', 'bar'],
...                      'key2': ['one', 'two', 'one'],
...                      'lval': [1, 2, 3]})
...

>>> right = pd.DataFrame({'key1': ['foo', 'foo', 'bar', 'bar'],
...                       'key2': ['one', 'one', 'one', 'two'],
...                        'rval': [4, 5, 6, 7]})
...

>>> left
  key1 key2  lval
0  foo  one     1
1  foo  two     2
2  bar  one     3

>>> right
  key1 key2  rval
0  foo  one     4
1  foo  one     5
2  bar  one     6
3  bar  two     7

>>> pd.merge(left, right, on=['key1', 'key2'], how='outer')
  key1 key2  lval  rval
0  foo  one   1.0   4.0
1  foo  one   1.0   5.0
2  foo  two   2.0   NaN
3  bar  one   3.0   6.0
4  bar  two   NaN   7.0
```

哪些key会出现在结果里是根据join方式来决定的，可以把多个key的组合当做是一个元组来作为单个key的join方式考虑(尽管实现方式并不是这样)

**注意**:

当列和列合并时，DataFrame的索引对象会被丢弃


最后一个关于合并的问题是处理结果中的重复列名。可以通过`suffixes`参数指定

```python

# 默认对重复的列名是自动添加了"_x", "_y"
>>> pd.merge(left, right, on='key1')
  key1 key2_x  lval key2_y  rval
0  foo    one     1    one     4
1  foo    one     1    one     5
2  foo    two     2    one     4
3  foo    two     2    one     5
4  bar    one     3    one     6
5  bar    one     3    two     7

>>> pd.merge(left, right, on='key1', suffixes=('_left', '_right'))
  key1 key2_left  lval key2_right  rval
0  foo       one     1        one     4
1  foo       one     1        one     5
2  foo       two     2        one     4
3  foo       two     2        one     5
4  bar       one     3        one     6
5  bar       one     3        two     7
```

### 基于索引做合并(Merging on Index)

通过设置`left_index=True`或`right_index=True`（或者两个同时设置）可以实现基于索引做合并

```python
>>> left1 = pd.DataFrame({'key': ['a', 'b', 'a', 'a', 'b', 'c'],
...                      'value': range(6)})
...

>>> right1 = pd.DataFrame({'group_val': [3.5, 7]}, index=['a', 'b'])

>>> left1
  key  value
0   a      0
1   b      1
2   a      2
3   a      3
4   b      4
5   c      5

>>> right1
   group_val
a        3.5
b        7.0

>>> pd.merge(left1, right1, left_on='key', right_index=True)
  key  value  group_val
0   a      0        3.5
2   a      2        3.5
3   a      3        3.5
1   b      1        7.0
4   b      4        7.0

>>> pd.merge(left1, right1, left_on='key', right_index=True, how='outer')
  key  value  group_val
0   a      0        3.5
2   a      2        3.5
3   a      3        3.5
1   b      1        7.0
4   b      4        7.0
5   c      5        NaN
```

多层索引的合并会更复杂一些

```python
>>> lefth = pd.DataFrame({'key1': ['Ohio', 'Ohio', 'Ohio', 'Nevada', 'Nevada'],
...                       'key2': [2000, 2001, 2002, 2001, 2002],
...                       'data': np.arange(5.)})
...

>>> righth = pd.DataFrame(np.arange(12).reshape((6, 2)), index=[['Nevada', 'Nevada', 'Ohio', 'Ohio', 'Ohio', 'Ohio'], [2001, 2000, 2000, 2000, 2001, 2002]],
...                       columns=['event1', 'event2'])
...

>>> lefth
   data    key1  key2
0   0.0    Ohio  2000
1   1.0    Ohio  2001
2   2.0    Ohio  2002
3   3.0  Nevada  2001
4   4.0  Nevada  2002

>>> righth
             event1  event2
Nevada 2001       0       1
       2000       2       3
Ohio   2000       4       5
       2000       6       7
       2001       8       9
       2002      10      11

>>> pd.merge(lefth, righth, left_on=['key1', 'key2'], right_index=True)
   data    key1  key2  event1  event2
0   0.0    Ohio  2000       4       5
0   0.0    Ohio  2000       6       7
1   1.0    Ohio  2001       8       9
2   2.0    Ohio  2002      10      11
3   3.0  Nevada  2001       0       1

>>> pd.merge(lefth, righth, left_on=['key1', 'key2'], right_index=True, how='outer')
   data    key1  key2  event1  event2
0   0.0    Ohio  2000     4.0     5.0
0   0.0    Ohio  2000     6.0     7.0
1   1.0    Ohio  2001     8.0     9.0
2   2.0    Ohio  2002    10.0    11.0
3   3.0  Nevada  2001     0.0     1.0
4   4.0  Nevada  2002     NaN     NaN
4   NaN  Nevada  2000     2.0     3.0
```

同时基于两边的索引合并

```python
>>> left2 = pd.DataFrame([[1., 2.], [3., 4.], [5., 6.]], index=['a', 'c', 'e'], columns=['Ohio', 'Nevada'])

>>> right2 = pd.DataFrame([[7., 8.], [9., 10.], [11., 12.], [13, 14]], index=['b', 'c', 'd', 'e'], columns=['Missouri', 'Alabama'])

>>> left2
   Ohio  Nevada
a   1.0     2.0
c   3.0     4.0
e   5.0     6.0

>>> right2
   Missouri  Alabama
b       7.0      8.0
c       9.0     10.0
d      11.0     12.0
e      13.0     14.0

>>> pd.merge(left2, right2, how='outer', left_index=True, right_index=True)
   Ohio  Nevada  Missouri  Alabama
a   1.0     2.0       NaN      NaN
b   NaN     NaN       7.0      8.0
c   3.0     4.0       9.0     10.0
d   NaN     NaN      11.0     12.0
e   5.0     6.0      13.0     14.0
```

另外对于基于索引合并，`DataFrame`还提供了一种简单的方法

```python
>>> left2.join(right2, how='outer')
   Ohio  Nevada  Missouri  Alabama
a   1.0     2.0       NaN      NaN
b   NaN     NaN       7.0      8.0
c   3.0     4.0       9.0     10.0
d   NaN     NaN      11.0     12.0
e   5.0     6.0      13.0     14.0
```

一次join多个`DataFrame`

```python
>>> another = pd.DataFrame([[7., 8.], [9., 10.], [11., 12.], [16., 17.]], index=['a', 'c', 'e', 'f'], columns=['New York', 'Oregon'])

>>> another
   New York  Oregon
a       7.0     8.0
c       9.0    10.0
e      11.0    12.0
f      16.0    17.0

>>> left2.join([right2, another])
   Ohio  Nevada  Missouri  Alabama  New York  Oregon
a   1.0     2.0       NaN      NaN       7.0     8.0
c   3.0     4.0       9.0     10.0       9.0    10.0
e   5.0     6.0      13.0     14.0      11.0    12.0
```

### 基于轴连接

`Numpy`里有`concatenate`函数来连接数组

```python
>>> arr= np.arange(12).reshape((3, 4))

>>> arr
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

>>> np.concatenate([arr, arr])
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11],
       [ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])

>>> np.concatenate([arr, arr], axis=1)
array([[ 0,  1,  2,  3,  0,  1,  2,  3],
       [ 4,  5,  6,  7,  4,  5,  6,  7],
       [ 8,  9, 10, 11,  8,  9, 10, 11]])
```

`Pandas`里面, 由于轴上多了索引信息，合并也更复杂一些。首先我们来看`Series`的合并

```python
>>> s1 = pd.Series([0, 1], index=['a', 'b'])

>>> s2 = pd.Series([2, 3, 4], index=['c', 'd', 'e'])

>>> s3 = pd.Series([5, 6], index=['f', 'g'])

>>> pd.concat([s1, s2, s3])
a    0
b    1
c    2
d    3
e    4
f    5
g    6
dtype: int64

>>> pd.concat([s1, s2, s3], axis=1)
     0    1    2
a  0.0  NaN  NaN
b  1.0  NaN  NaN
c  NaN  2.0  NaN
d  NaN  3.0  NaN
e  NaN  4.0  NaN
f  NaN  NaN  5.0
g  NaN  NaN  6.0
```

默认的合并方式是"outer"join, 我们也可以通过`join`参数指定`inner`join

```python
>>> s4 = pd.concat([s1, s3])

>>> s4
a    0
b    1
f    5
g    6
dtype: int64

>>> pd.concat([s1, s3], axis=1)
     0    1
a  0.0  NaN
b  1.0  NaN
f  NaN  5.0
g  NaN  6.0

>>> pd.concat([s1, s4], axis=1, join='inner')
   0  1
a  0  0
b  1  1
```

通过参数`join_axes`, 我们可以指定join的列

```python
>>> pd.concat([s1, s1, s3], axis=1, keys=['one', 'two', 'three'])
   one  two  three
a  0.0  0.0    NaN
b  1.0  1.0    NaN
f  NaN  NaN    5.0
g  NaN  NaN    6.0
```

通过参数`keys`, 可以创建多层索引

```python
# keys参数的值和要合并的列表一一对应
>>> result = pd.concat([s1, s1, s3], keys=['one', 'two', 'three'])

>>> result
one    a    0
       b    1
two    a    0
       b    1
three  f    5
       g    6
dtype: int64

>>> result.unstack()
         a    b    f    g
one    0.0  1.0  NaN  NaN
two    0.0  1.0  NaN  NaN
three  NaN  NaN  5.0  6.0
```

通过设置`axis=1`， 也可以创建列索引

```python

>>> pd.concat([s1, s1, s3], axis=1)
     0    1    2
a  0.0  0.0  NaN
b  1.0  1.0  NaN
f  NaN  NaN  5.0
g  NaN  NaN  6.0

>>> pd.concat([s1, s1, s3], axis=1, keys=['one', 'two', 'three'])
   one  two  three
a  0.0  0.0    NaN
b  1.0  1.0    NaN
f  NaN  NaN    5.0
g  NaN  NaN    6.0
```

同样的逻辑，可以扩展到`DataFrame`

```python
>>> df1 = pd.DataFrame(np.arange(6).reshape(3, 2), index=['a', 'b', 'c'], columns=['one', 'two'])

>>> df2 = pd.DataFrame(5 + np.arange(4).reshape(2, 2), index=['a', 'c'],columns=['three', 'four'])

>>> df1
   one  two
a    0    1
b    2    3
c    4    5

>>> df2
   three  four
a      5     6
c      7     8

>>> pd.concat([df1, df2], axis=1, keys=['level1', 'level2'])
  level1     level2
     one two  three four
a      0   1    5.0  6.0
b      2   3    NaN  NaN
c      4   5    7.0  8.0
```

也可以通过传递字典的方式, 字典的keys将作为`keys`参数。

```python
>>> pd.concat({'level1': df1, 'level2': df2}, axis=1)
  level1     level2
     one two  three four
a      0   1    5.0  6.0
b      2   3    NaN  NaN
c      4   5    7.0  8.0
```

还可以通过`names`参数指定所以的名字

```python
>>> pd.concat([df1, df2], axis=1, keys=['level1', 'level2'],names=['upper', 'lower'])
upper level1     level2
lower    one two  three four
a          0   1    5.0  6.0
b          2   3    NaN  NaN
c          4   5    7.0  8.0
```

最后一种情况是如果两个需要合并的DataFrame行索引没有任何相关的数据，可以通过` ignore_index=True`来合并。

```python
>>> df1 = pd.DataFrame(np.random.randn(3, 4), columns=['a', 'b', 'c', 'd'])

>>> df2 = pd.DataFrame(np.random.randn(2, 3), columns=['b', 'd', 'a'])

>>> df1
          a         b         c         d
0  0.440681  0.961317  1.087380  0.906632
1  1.614141 -0.511330 -0.384905  0.691080
2 -0.526509  0.084097 -0.068141  1.002294

>>> df2
          b         d         a
0  0.264169 -1.022511 -1.322645
1 -0.698436 -0.148560  1.582804

>>> pd.concat([df1, df2], ignore_index=True)
          a         b         c         d
0  0.440681  0.961317  1.087380  0.906632
1  1.614141 -0.511330 -0.384905  0.691080
2 -0.526509  0.084097 -0.068141  1.002294
3 -1.322645  0.264169       NaN -1.022511
4  1.582804 -0.698436       NaN -0.148560
```

### 合并重复的数据

有一种数据结合方法既不属于`merge`，也不属于`concatenation`。比如两个数据集，index可能完全覆盖，或覆盖一部分。这里举个例子，考虑下numpy的where函数，可以在数组上进行类似于if-else表达式般的判断：

```python
>>> a = pd.Series([np.nan, 2.5, np.nan, 3.5, 4.5, np.nan], index=['f', 'e', 'd', 'c', 'b', 'a'])

>>> b = pd.Series(np.arange(len(a), dtype=np.float64), index=['f', 'e', 'd', 'c', 'b', 'a'])

>>> b[-1] = np.nan

>>> a
f    NaN
e    2.5
d    NaN
c    3.5
b    4.5
a    NaN
dtype: float64

>>> b
f    0.0
e    1.0
d    2.0
c    3.0
b    4.0
a    NaN
dtype: float64

>>> np.where(pd.isnull(a), b, a)
array([ 0. ,  2.5,  2. ,  3.5,  4.5,  nan])
```

`Series`的`combine_first`函数可以达到类似的效果

```python
>>> b[:-2]
f    0.0
e    1.0
d    2.0
c    3.0
dtype: float64

>>> a[2:]
d    NaN
c    3.5
b    4.5
a    NaN
dtype: float64

# b中有值时就用b中的，否则就用a的，index是两个Series的 outer join
>>> b[:-2].combine_first(a[2:])
a    NaN
b    4.5
c    3.0
d    2.0
e    1.0
f    0.0
dtype: float64
```

`DataFrame`的`combine_first`函数也有同样的功能，在DataFrame调用`combine_first`时，可以认为把传递给`combine_first`函数的DataFrame作为调用该函数的Dataframe缺失数据的补充。

```python
>>> df1 = pd.DataFrame({'a': [1., np.nan, 5., np.nan],
...                      'b': [np.nan, 2., np.nan, 6.],
...                      'c': range(2, 18, 4)})
...

>>> df2 = pd.DataFrame({'a': [5., 4., np.nan, 3., 7.],
...                    'b': [np.nan, 3., 4., 6., 8.]})
...

>>> df1
     a    b   c
0  1.0  NaN   2
1  NaN  2.0   6
2  5.0  NaN  10
3  NaN  6.0  14

>>> df2
     a    b
0  5.0  NaN
1  4.0  3.0
2  NaN  4.0
3  3.0  6.0
4  7.0  8.0

# df1中缺失的数据，使用df2中的补上
>>> df1.combine_first(df2)
     a    b     c
0  1.0  NaN   2.0
1  4.0  2.0   6.0
2  5.0  4.0  10.0
3  3.0  6.0  14.0
4  7.0  8.0   NaN
```

## 变型和旋转(Reshaping and Pivoting)


### 多层索引变型

在DataFrame中，主要有两个方法用于多层索引变型

* `stack`: 把列转换为行
* `unstack`: 把行转换为列

```python
>>> data = pd.DataFrame(np.arange(6).reshape((2, 3)),
...                     index=pd.Index(['Ohio', 'Colorado'], name='state'),
...                     columns=pd.Index(['one', 'two', 'three'], name='number'))
...

>>> data
number    one  two  three
state
Ohio        0    1      2
Colorado    3    4      5

# 把列转换为行
>>> result = data.stack()

>>> result
state     number
Ohio      one       0
          two       1
          three     2
Colorado  one       3
          two       4
          three     5
dtype: int64

# 把行转换为列
>>> result.unstack()
number    one  two  three
state
Ohio        0    1      2
Colorado    3    4      5
```

默认情况下`unstack`是从最里层的列被转换为行(`stack`函数也类似)。也可以通过层的索引和名字来。

```python
>>> result.index
MultiIndex(levels=[['Ohio', 'Colorado'], ['one', 'two', 'three']],
           labels=[[0, 0, 0, 1, 1, 1], [0, 1, 2, 0, 1, 2]],
           names=['state', 'number'])

# 0对应Index levels的索引
>>> result.unstack(0)
state   Ohio  Colorado
number
one        0         3
two        1         4
three      2         5

# state对应Index name
>>> result.unstack('state')
state   Ohio  Colorado
number
one        0         3
two        1         4
three      2         5
```

`unstack`的过程中，也可能会引入一些缺失值

```python
>>> s1 = pd.Series([0, 1, 2, 3], index=['a', 'b', 'c', 'd'])

>>> s2 = pd.Series([4, 5, 6], index=['c', 'd', 'e'])

>>> data2 = pd.concat([s1, s2], keys=['one', 'two'])

>>> data2
one  a    0
     b    1
     c    2
     d    3
two  c    4
     d    5
     e    6
dtype: int64

>>> data2.unstack()
       a    b    c    d    e
one  0.0  1.0  2.0  3.0  NaN
two  NaN  NaN  4.0  5.0  6.0
```

`stack`函数默认会过滤掉NaN数据

```python
>>> data2.unstack().stack()
one  a    0.0
     b    1.0
     c    2.0
     d    3.0
two  c    4.0
     d    5.0
     e    6.0
dtype: float64

>>> data2.unstack().stack(dropna=False)
one  a    0.0
     b    1.0
     c    2.0
     d    3.0
     e    NaN
two  a    NaN
     b    NaN
     c    4.0
     d    5.0
     e    6.0
dtype: float64
```


当`unstack`作用于DataFrame时，则是从最低层开始。

```python
>>> df = pd.DataFrame({'left': result, 'right': result + 5},
...                    columns=pd.Index(['left', 'right'], name='side'))
...

>>> df
side             left  right
state    number
Ohio     one        0      5
         two        1      6
         three      2      7
Colorado one        3      8
         two        4      9
         three      5     10

>>> df.unstack()
side     left           right
number    one two three   one two three
state
Ohio        0   1     2     5   6     7
Colorado    3   4     5     8   9    10

# 也可以通过索引名来指定要操作的列
>>> df.unstack('state')
side   left          right
state  Ohio Colorado  Ohio Colorado
number
one       0        3     5        8
two       1        4     6        9
three     2        5     7       10

>>> df.unstack('state').stack('side')
state         Colorado  Ohio
number side
one    left          3     0
       right         8     5
two    left          4     1
       right         9     6
three  left          5     2
       right        10     7
```

### 把“长”格式转换为"宽"格式(Pivoting “Long” to “Wide” Format)

一种常见的把多个时间序列保存在数据库或csv文件的方式叫作**long or stacked format**。 让我们来看一些例子(例子里使用的文件可以从这里[下载](https://github.com/wesm/pydata-book))


```python
>>> data = pd.read_csv('examples/macrodata.csv')

>>> data.head()
     year  quarter   realgdp  realcons  realinv  realgovt  realdpi    cpi  \
0  1959.0      1.0  2710.349    1707.4  286.898   470.045   1886.9  28.98
1  1959.0      2.0  2778.801    1733.7  310.859   481.301   1919.7  29.15
2  1959.0      3.0  2775.488    1751.8  289.226   491.260   1916.4  29.35
3  1959.0      4.0  2785.204    1753.7  299.356   484.052   1931.3  29.37
4  1960.0      1.0  2847.699    1770.5  331.722   462.199   1955.5  29.54

      m1  tbilrate  unemp      pop  infl  realint
0  139.7      2.82    5.8  177.146  0.00     0.00
1  141.7      3.08    5.1  177.830  2.34     0.74
2  140.5      3.82    5.3  178.657  2.74     1.09
3  140.0      4.33    5.6  179.386  0.27     4.06
4  139.6      3.50    5.2  180.007  2.31     1.19

>>> periods = pd.PeriodIndex(year=data.year, quarter=data.quarter,  name='date')

>>> columns = pd.Index(['realgdp', 'infl', 'unemp'], name='item')

>>> data = data.reindex(columns=columns)

>>> data.head()
item   realgdp  infl  unemp
0     2710.349  0.00    5.8
1     2778.801  2.34    5.1
2     2775.488  2.74    5.3
3     2785.204  0.27    5.6
4     2847.699  2.31    5.2

>>> data.index = periods.to_timestamp('D', 'end')

>>> data.head()
item         realgdp  infl  unemp
date
1959-03-31  2710.349  0.00    5.8
1959-06-30  2778.801  2.34    5.1
1959-09-30  2775.488  2.74    5.3
1959-12-31  2785.204  0.27    5.6
1960-03-31  2847.699  2.31    5.2

>>> data.stack().head()
date        item
1959-03-31  realgdp    2710.349
            infl          0.000
            unemp         5.800
1959-06-30  realgdp    2778.801
            infl          2.340
dtype: float64

>>> data.stack().head().reset_index()
        date     item         0
0 1959-03-31  realgdp  2710.349
1 1959-03-31     infl     0.000
2 1959-03-31    unemp     5.800
3 1959-06-30  realgdp  2778.801
4 1959-06-30     infl     2.340

>>> data.stack().head().reset_index().rename(columns={0: 'value'})
        date     item     value
0 1959-03-31  realgdp  2710.349
1 1959-03-31     infl     0.000
2 1959-03-31    unemp     5.800
3 1959-06-30  realgdp  2778.801
4 1959-06-30     infl     2.340

>>> ldata[:10]
        date     item     value
0 1959-03-31  realgdp  2710.349
1 1959-03-31     infl     0.000
2 1959-03-31    unemp     5.800
3 1959-06-30  realgdp  2778.801
4 1959-06-30     infl     2.340
5 1959-06-30    unemp     5.100
6 1959-09-30  realgdp  2775.488
7 1959-09-30     infl     2.740
8 1959-09-30    unemp     5.300
9 1959-12-31  realgdp  2785.204
```

上面这种`ldata`的格式就被称作`long format for multiple time series`

```python
>>> pivoted = ldata.pivot('date', 'item', 'value')

>>> pivoted.head()
item        infl   realgdp  unemp
date
1959-03-31  0.00  2710.349    5.8
1959-06-30  2.34  2778.801    5.1
1959-09-30  2.74  2775.488    5.3
1959-12-31  0.27  2785.204    5.6
1960-03-31  2.31  2847.699    5.2
```

###  把“宽”格式转换为"长"格式(Pivoting “Long” to “Wide” Format)

与`pivot`相反的操作就是`pandas.melt`

```python
>>> df = pd.DataFrame({'key': ['foo', 'bar', 'baz'],
...                    'A': [1, 2, 3],
...                    'B': [4, 5, 6],
...                    'C': [7, 8, 9]})
...

>>> df
   A  B  C  key
0  1  4  7  foo
1  2  5  8  bar
2  3  6  9  baz

>>> melted = pd.melt(df, ['key'])

>>> melted
   key variable  value
0  foo        A      1
1  bar        A      2
2  baz        A      3
3  foo        B      4
4  bar        B      5
5  baz        B      6
6  foo        C      7
7  bar        C      8
8  baz        C      9
```
