---
title: 《利用Python进行数据分析第二版》读书笔记二
date: 2017-12-28T15:57:31+08:00
tags:
    - numpy
    - pandas
    - matplotlib
categories:
    - python数据分析
slug: python-for-data-analysis-2nd-edition-note-two
description: >
    本文主要介绍使用Pandas进行数据清洗和预处理相关操作

---

## 数据清洗和预处理

### 过滤出缺失的数据

Series

```python
>>> from numpy import nan as NA

>>> data = pd.Series([1, NA, 3.5, NA, 7])

>>> data.dropna()
0    1.0
2    3.5
4    7.0
dtype: float64

# 等价于上面的dropna
>>> data[data.notnull()]
0    1.0
2    3.5
4    7.0
dtype: float64
```

<!--more-->

DataFrame

```python
>>> data = pd.DataFrame([[1., 6.5, 3.], [1., NA, NA], [NA, NA, NA], [NA, 6.5, 3.]])

>>> cleaned = data.dropna()

>>> data
     0    1    2
0  1.0  6.5  3.0
1  1.0  NaN  NaN
2  NaN  NaN  NaN
3  NaN  6.5  3.0

>>> cleaned
     0    1    2
0  1.0  6.5  3.0
```

过滤至少包含几个值的数据

```python
>>> df = pd.DataFrame(np.random.randn(7, 3))

>>> df.iloc[:4, 1] = NA

>>> df.iloc[:2, 2] = NA

>>> df
          0         1         2
0  0.006690       NaN       NaN
1  0.850493       NaN       NaN
2 -0.601578       NaN -0.453566
3 -0.819006       NaN -0.176161
4 -0.440714 -0.700860 -1.266824
5 -1.347181  1.561944 -0.203258
6  3.195225  1.604114 -1.243471

>>> df.dropna()
          0         1         2
4 -0.440714 -0.700860 -1.266824
5 -1.347181  1.561944 -0.203258
6  3.195225  1.604114 -1.243471

# 保留至少有2个为非NaN值的的行
>>> df.dropna(thresh=2)
          0         1         2
2 -0.601578       NaN -0.453566
3 -0.819006       NaN -0.176161
4 -0.440714 -0.700860 -1.266824
5 -1.347181  1.561944 -0.203258
6  3.195225  1.604114 -1.243471
```

### 填充缺失的数据

补充缺失的值为0

```python
>>> df
          0         1         2
0  0.006690       NaN       NaN
1  0.850493       NaN       NaN
2 -0.601578       NaN -0.453566
3 -0.819006       NaN -0.176161
4 -0.440714 -0.700860 -1.266824
5 -1.347181  1.561944 -0.203258
6  3.195225  1.604114 -1.243471

>>> df.fillna(0)
          0         1         2
0  0.006690  0.000000  0.000000
1  0.850493  0.000000  0.000000
2 -0.601578  0.000000 -0.453566
3 -0.819006  0.000000 -0.176161
4 -0.440714 -0.700860 -1.266824
5 -1.347181  1.561944 -0.203258
6  3.195225  1.604114 -1.243471
```

按照指定的列填充缺失的值。第1列为NaN的填充为0.5，第2列为NaN的填充为0

```python
>>> df.fillna({1: 0.5, 2: 0})
          0         1         2
0  0.006690  0.500000  0.000000
1  0.850493  0.500000  0.000000
2 -0.601578  0.500000 -0.453566
3 -0.819006  0.500000 -0.176161
4 -0.440714 -0.700860 -1.266824
5 -1.347181  1.561944 -0.203258
6  3.195225  1.604114 -1.243471
```

默认情况下，fillna都是返回一个新的对象，使用`inplace`参数可以直接修改原来的DataFame

```python
>>> _ = df.fillna(0, inplace=True)

>>> df
          0         1         2
0  0.006690  0.000000  0.000000
1  0.850493  0.000000  0.000000
2 -0.601578  0.000000 -0.453566
3 -0.819006  0.000000 -0.176161
4 -0.440714 -0.700860 -1.266824
5 -1.347181  1.561944 -0.203258
6  3.195225  1.604114 -1.243471
```

### 去掉重复的行

```python
>>> data = pd.DataFrame({'k1': ['one', 'two'] * 3 + ['two'],
...                      'k2': [1, 1, 2, 3, 3, 4, 4]})
...

>>> data
    k1  k2
0  one   1
1  two   1
2  one   2
3  two   3
4  one   3
5  two   4
6  two   4

# 重复的行返回为True
>>> data.duplicated()
0    False
1    False
2    False
3    False
4    False
5    False
6     True
dtype: bool

# 去掉重复的行(即data.duplicated()为True的行)
>>> data.drop_duplicates()
    k1  k2
0  one   1
1  two   1
2  one   2
3  two   3
4  one   3
5  two   4

>>> data['v1'] = range(7)

>>> data
    k1  k2  v1
0  one   1   0
1  two   1   1
2  one   2   2
3  two   3   3
4  one   3   4
5  two   4   5
6  two   4   6

# 去掉k1列重复的行
>>> data.drop_duplicates(['k1'])
    k1  k2  v1
0  one   1   0
1  two   1   1

# 默认情况下，遇到重复的值时，只会保留第一个出现的，可以通过keep参数保留最后一次出现的行
>>> data.drop_duplicates(['k1', 'k2'], keep='last')
    k1  k2  v1
0  one   1   0
1  two   1   1
2  one   2   2
3  two   3   3
4  one   3   4
6  two   4   6
```

### 替换值(Replace)

```python
>>> data = pd.Series([1., -999., 2., -999., -1000., 3.])

>>> data
0       1.0
1    -999.0
2       2.0
3    -999.0
4   -1000.0
5       3.0
dtype: float64

# 将-999替换成nan
>>> data.replace(-999, np.nan)
0       1.0
1       NaN
2       2.0
3       NaN
4   -1000.0
5       3.0
dtype: float64

# 将多个值-999和-1000替换成nan
>>> data.replace([-999, -1000], np.nan)
0    1.0
1    NaN
2    2.0
3    NaN
4    NaN
5    3.0
dtype: float64

#  将-999替换成nan， -1000替换成0
>>> data.replace([-999, -1000], [np.nan, 0])
0    1.0
1    NaN
2    2.0
3    NaN
4    0.0
5    3.0
dtype: float64

# 效果同上
>>> data.replace({-999: np.nan, -1000: 0})
0    1.0
1    NaN
2    2.0
3    NaN
4    0.0
5    3.0
dtype: float64
```

### 重命名轴的索引(Renaming Axis Indexes)


```python
>>> data = pd.DataFrame(np.arange(12).reshape((3, 4)),
...                     index=['Ohio', 'Colorado', 'New York'],
...                     columns=['one', 'two', 'three', 'four'])
...

>>> transform = lambda x: x[:4].upper()

# map方法返回一个新的Index类型，不会修改原来的值
>>> data.index.map(transform)
Index(['OHIO', 'COLO', 'NEW '], dtype='object')

>>> data
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
New York    8    9     10    11

# 可以重新赋值index达到replace的效果
>>> data.index = data.index.map(transform)

>>> data
      one  two  three  four
OHIO    0    1      2     3
COLO    4    5      6     7
NEW     8    9     10    11

# rename方法也可以接受函数
>>> data.rename(index=str.title, columns=str.upper)
      ONE  TWO  THREE  FOUR
Ohio    0    1      2     3
Colo    4    5      6     7
New     8    9     10    11

# 同样rename方法可以接受字典
>>> data.rename(index={'OHIO': 'INDIANA'},columns={'three': 'peekaboo'})
         one  two  peekaboo  four
INDIANA    0    1         2     3
COLO       4    5         6     7
NEW        8    9        10    11
```


### 离散和装箱

考虑这样一个场景，我们有一些关于人的年龄的数据，现在需要根据年龄大小来分组，如:
18-25岁，26岁到35岁， 36岁到60岁，61岁以上来分组，可以这样实现

```python
>>> ages = [20, 22, 25, 27, 21, 23, 37, 31, 61, 45, 41, 32]

>>> bins = [18, 25, 35, 60, 100]

# 使用cut方法装箱
>>> cats = pd.cut(ages, bins)

>>> cats
[(18, 25], (18, 25], (18, 25], (25, 35], (18, 25], ..., (25, 35], (60, 100], (35, 60], (35, 60], (25, 35]]
Length: 12
Categories (4, interval[int64]): [(18, 25] < (25, 35] < (35, 60] < (60, 100]]


>>> cats.codes
array([0, 0, 0, 1, 0, 0, 2, 1, 3, 2, 2, 1], dtype=int8)

>>> cats.categories
IntervalIndex([(18, 25], (25, 35], (35, 60], (60, 100]]
              closed='right',
              dtype='interval[int64]')

# 统计每个年龄段的人数
>>> pd.value_counts(cats)
(18, 25]     5
(35, 60]     3
(25, 35]     3
(60, 100]    1
dtype: int64
```

默认情况下括号(18, 25]左开右闭，表示>18 并且 <=25，可以指定括号的开闭情况

```python
>>> pd.cut(ages, bins, right=False)
[[18, 25), [18, 25), [25, 35), [25, 35), [18, 25), ..., [25, 35), [60, 100), [35, 60), [35, 60), [25, 35)]
Length: 12
Categories (4, interval[int64]): [[18, 25) < [25, 35) < [35, 60) < [60, 100)]
```

我们也可以指定每个箱子的名字

```python
>>> group_names = ['Youth', 'YoungAdult', 'MiddleAged', 'Senior']

>>> pd.cut(ages, bins, labels=group_names)
[Youth, Youth, Youth, YoungAdult, Youth, ..., YoungAdult, Senior, MiddleAged, MiddleAged, YoungAdult]
Length: 12
Categories (4, object): [Youth < YoungAdult < MiddleAged < Senior]
```

在调用cut函数，指定`bins`参数时，如果传递是一个数字n而不是一个区间，那么默认会将Searies按照从小到到的顺序自动分成n个区间。

```python
>>> data = np.random.rand(20)

>>> data
array([ 0.18918431,  0.84853486,  0.03490019,  0.99371555,  0.72304736,
        0.3356213 ,  0.40800301,  0.71508546,  0.45061796,  0.28337607,
        0.31318716,  0.86330303,  0.88466783,  0.22120597,  0.25290735,
        0.82730126,  0.14255095,  0.36400834,  0.75004135,  0.1090306 ])

#precision表示小数展示的位数
>>> pd.cut(data, 4, precision=2)
[(0.034, 0.27], (0.75, 0.99], (0.034, 0.27], (0.75, 0.99], (0.51, 0.75], ..., (0.75, 0.99], (0.034, 0.27], (0.27, 0.51], (0.51, 0.75], (0.034, 0.27]]
Length: 20
Categories (4, interval[float64]): [(0.034, 0.27] < (0.27, 0.51] < (0.51, 0.75] < (0.75, 0.99]]
```

一个跟`cut`类似的办法是`qcut`,使用cut方法，通常无法保证每个装箱的元素个数是相同的，使用`qcut`可以按照百分比来分配元素

```python
>>> data = np.random.randn(1000)

>>> cats = pd.qcut(data, 4)

>>> cats
[(0.72, 3.328], (-0.632, 0.0365], (0.72, 3.328], (-3.237, -0.632], (-0.632, 0.0365], ..., (0.0365, 0.72], (0.0365, 0.72], (-0.632, 0.0365], (-0.632, 0.0365], (0.0365, 0.72]]
Length: 1000
Categories (4, interval[float64]): [(-3.237, -0.632] < (-0.632, 0.0365] < (0.0365, 0.72] <
                                    (0.72, 3.328]]

# 可以看到每个装箱的元素个数是一致的
>>> pd.value_counts(cats)
(0.72, 3.328]       250
(0.0365, 0.72]      250
(-0.632, 0.0365]    250
(-3.237, -0.632]    250
dtype: int64
```

同样，我们可以指定每个装箱的百分比

```python
# 累进的百分比10%, 40%, 40%, 10%
>>> cats=pd.qcut(data, [0, 0.1, 0.5, 0.9, 1.])

>>> cats
[(0.0365, 1.271], (-1.215, 0.0365], (0.0365, 1.271], (-1.215, 0.0365], (-1.215, 0.0365], ..., (0.0365, 1.271], (0.0365, 1.271], (-1.215, 0.0365], (-1.215, 0.0365], (0.0365, 1.271]]
Length: 1000
Categories (4, interval[float64]): [(-3.237, -1.215] < (-1.215, 0.0365] < (0.0365, 1.271] <
                                    (1.271, 3.328]]

>>> pd.value_counts(cats)
(0.0365, 1.271]     400
(-1.215, 0.0365]    400
(1.271, 3.328]      100
(-3.237, -1.215]    100
dtype: int64
```

### 检查和过滤异常值

```python
>>> data = pd.DataFrame(np.random.randn(1000, 4))

>>> data.describe()
                 0            1            2            3
count  1000.000000  1000.000000  1000.000000  1000.000000
mean      0.034294    -0.023318     0.009570     0.067062
std       1.001493     0.971333     0.996241     0.959285
min      -2.937180    -3.697901    -3.195441    -3.012063
25%      -0.673629    -0.654237    -0.641570    -0.567363
50%       0.030598    -0.004002     0.017635     0.093394
75%       0.691715     0.636873     0.659011     0.697847
max       3.736384     3.272939     3.114762     3.311736

# 找出第3列绝对值大于3的行
>>> col = data[2]

>>> col[np.abs(col) > 3]
219   -3.195441
980    3.114762
Name: 2, dtype: float64

# 找出所有包含绝对值大于3的行
>>> data[(np.abs(data) > 3).any(1)]
            0         1         2         3
21   1.082695  1.455599  0.740997  3.053146
205  3.736384 -0.109024  0.252715  1.640695
219 -1.513811  1.117404 -3.195441  0.975343

# sign函数根据元素的值返回
# 小于0 返回 -1
# 等于0 返回 0
# 大于0 返回1
# NaN 返回 NaN
>>> np.sign(data).head()
     0    1    2    3
0  1.0 -1.0  1.0  1.0
1  1.0  1.0  1.0  1.0
2  1.0  1.0 -1.0  1.0
3  1.0  1.0  1.0  1.0
4  1.0 -1.0  1.0 -1.0
```

### 随机排序和随机取样

使用`numpy.random.permutation`函数，很容易实现对Series和DataFrame的随机排序

```python
# 初始化一个DataFrame
>>> df = pd.DataFrame(np.arange(5 * 4).reshape((5, 4)))

# df的索引时0至4按顺序排列的
>>> df
    0   1   2   3
0   0   1   2   3
1   4   5   6   7
2   8   9  10  11
3  12  13  14  15
4  16  17  18  19

# 生成一个随机的索引序列
>>> sampler = np.random.permutation(5)

>>> sampler
array([2, 4, 3, 0, 1])

# 使用新的索引序列
>>> df.take(sampler)
    0   1   2   3
2   8   9  10  11
4  16  17  18  19
3  12  13  14  15
0   0   1   2   3
1   4   5   6   7
```

随机返回一些行的数据

```python
# 随机返回3行数据
>>> df.sample(3)
    0   1   2   3
0   0   1   2   3
2   8   9  10  11
4  16  17  18  19
```

使用`replace`参数可以允许重复取样

```python
>>> choices = pd.Series([5, 7, -1, 6, 4])

# 随机从Series中取10个元素，允许重复使用
>>> draws = choices.sample(n=10, replace=True)

>>> draws
4    4
4    4
0    5
2   -1
0    5
4    4
1    7
4    4
1    7
0    5
dtype: int64
```

### 计算指标/虚拟变量(Computing Indicator/Dummy Variables)

定义参考:

<http://wiki.mbalib.com/wiki/%E8%99%9A%E6%8B%9F%E5%8F%98%E9%87%8F>

一种常用于统计建模或机器学习的转换方式是: 将分类变量转换为“哑变量矩阵”或“指标矩阵”。如果 DataFrame 的某一列中含有 k 个
不同的值，则可以派生出一个 k 列矩阵或 DataFrame（其值全为1和0）。pandas 有一个 get_dummies 函数可以实现该功能。

```python
>>> df = pd.DataFrame({'key': ['b', 'b', 'a', 'c', 'a', 'b'], 'data1': range(6)}
... )

>>> df
   data1 key
0      0   b
1      1   b
2      2   a
3      3   c
4      4   a
5      5   b

# 指定列元素值根据是否出现，分别赋值为0和1
>>> pd.get_dummies(df['key'])
   a  b  c
0  0  1  0
1  0  1  0
2  1  0  0
3  0  0  1
4  1  0  0
5  0  1  0
```

也可以指定一个前缀

```python
>>> dummies = pd.get_dummies(df['key'], prefix='key')

>>> dummies
   key_a  key_b  key_c
0      0      1      0
1      0      1      0
2      1      0      0
3      0      0      1
4      1      0      0
5      0      1      0
```

另外一种常用的方法是和离散函数`cut`结合

```python
>>> np.random.seed(12345)

>>> values = np.random.rand(10)

>>> values
array([ 0.92961609,  0.31637555,  0.18391881,  0.20456028,  0.56772503,
        0.5955447 ,  0.96451452,  0.6531771 ,  0.74890664,  0.65356987])

>>> bins = [0, 0.2, 0.4, 0.6, 0.8, 1]

# 统计了每个索引值在哪个区间
# 索引为0的0.92961609，出现在区间(0.8, 1.0]
>>> pd.get_dummies(pd.cut(values, bins))
   (0.0, 0.2]  (0.2, 0.4]  (0.4, 0.6]  (0.6, 0.8]  (0.8, 1.0]
0           0           0           0           0           1
1           0           1           0           0           0
2           1           0           0           0           0
3           0           1           0           0           0
4           0           0           1           0           0
5           0           0           1           0           0
6           0           0           0           0           1
7           0           0           0           1           0
8           0           0           0           1           0
9           0           0           0           1           0
```