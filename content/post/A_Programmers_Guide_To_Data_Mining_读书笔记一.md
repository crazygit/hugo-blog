---
title: 《A Programmer's Guide to Data Mining》读书笔记一
date: 2018-01-22T17:34:52+08:00
tags:
    - 推荐系统
    - 相似度计算
categories:
    - 推荐系统
slug: a-programmers-guid-to-data-mining-note-one
description: >
    推荐系统入门， 物品相似度计算方法汇总
mathjax: true

---
参考:

[A Programmer's Guide to Data Mining](http://guidetodatamining.com/)


推荐系统中，最常用的方法就是协同过滤。顾名思义: 这个方法就是利用别人的喜好来推测你的喜好。
假如你喜欢看电影A和B， 另外一个和你**相似的用户**也喜欢电影A和B，同时他还喜欢看电影C，那么我们就会推测你也喜欢电影C。

[如图所示](https://en.wikipedia.org/?title=Collaborative_filtering)

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-19-Collaborative_filtering.gif)

<!--more-->

## 如何找到相似的用户

相似度计算方法有很多，本文将主要介绍以下几种

距离算法

* 曼哈顿距离(Manhattan Distance)
* 欧几里得距离(Euclidean Distance)
* 切比雪夫距离(Chebyshev Distance)
* 闵可夫斯基距离(Minkowski Distance)

相似度系数算法

* 皮尔逊相关系数(Pearson Correlation Coefficient)
* 向量空间余弦相似度(Cosine Similarity)

对于相同的计算数据，不同的相似度计算方法结果不同，所以需要根据计算样本的特性，选择合适的计算方式。

更多相似度计算方法可以参考:

* [常见的距离算法和相似度(相关系数)计算方法](http://www.cnblogs.com/arachis/p/Similarity.html)
* [机器学习中的相似性度量](http://www.cnblogs.com/heaad/archive/2011/03/08/1977733.html)
* [常用的相似度计算方法原理及实现](http://blog.csdn.net/yixianfeng41/article/details/61917158)


### 曼哈顿距离(Manhattan Distance)

参考[百度百科](https://baike.baidu.com/item/%E6%9B%BC%E5%93%88%E9%A1%BF%E8%B7%9D%E7%A6%BB)

> 曼哈顿距离，又叫出租车距离（Manhattan Distance）是由十九世纪的赫尔曼·闵可夫斯基所创词汇 ，是种使用在几何度量空间的几何学用语，用以标明两个点在标准坐标系上的绝对轴距总和。

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-22-manhattan.jpg)

> 图中红线代表曼哈顿距离，绿色代表欧氏距离，也就是直线距离，而蓝色和黄色代表等价的曼哈顿距离。曼哈顿距离——两点在南北方向上的距离加上在东西方向上的距离，即$d(i, j）= |x_i - x_j|+|y_i - y_j|$。对于一个具有正南正北、正东正西方向规则布局的城镇街道，从一点到达另一点的距离正是在南北方向上旅行的距离加上在东西方向上旅行的距离，因此，曼哈顿距离又称为出租车距离。曼哈顿距离不是距离不变量，当坐标轴变动时，点间的距离就会不同。曼哈顿距离示意图在早期的计算机图形学中，屏幕是由像素构成，是整数，点的坐标也一般是整数，原因是浮点运算很昂贵，很慢而且有误差，如果直接使用AB的欧氏距离(欧几里德距离: 在二维和三维空间中的欧氏距离的就是两点之间的距离），则必须要进行浮点运算，如果使用AC和CB，则只要计算加减法即可，这就大大提高了运算速度，而且不管累计运算多少次，都不会有误差。

* 二维平面两点a(x1,y1)与b(x2,y2)间的曼哈顿距离

$$
d_{ab} = |x_1 - x_2| + |y_1 - y_2|
$$

* 两个n维向量a(x11,x12,…,x1n)与b(x21,x22,…,x2n)间的曼哈顿距离

{% math %}

d_{ab} =\sum\limits_{k=1}^n|x_{1k} - x_{2k}|

{% endmath %}

### 欧几里得距离(Euclidean Distance)

参考[维基百科](在数学中，欧几里得距离或欧几里得度量是欧几里得空间中两点间“普通”（即直线）距离)

> 在数学中，欧几里得距离或欧几里得度量是欧几里得空间中两点间“普通”（即直线）距离

* 二维平面上两点a(x1,y1)与b(x2,y2)间的欧几里得距离:

$$
d_{ab} = \sqrt{(x_1-x_2)^2 + (y_1-y_2)^2}
$$

* 两个n维向量a(x11,x12,…,x1n)与 b(x21,x22,…,x2n)间的欧几里得距离

{% math %}

d_{ab} = \sqrt{\sum\limits_{k=1}^n(x_{1k} - x_{2k})^2}

{% endmath %}

### 切比雪夫距离(Chebyshev Distance)

参考[维基百科](https://zh.wikipedia.org/wiki/%E5%88%87%E6%AF%94%E9%9B%AA%E5%A4%AB%E8%B7%9D%E7%A6%BB)

> 数学上，切比雪夫距离（Chebyshev distance）或是L∞度量是向量空间中的一种度量，二个点之间的距离定义为_其各座标数值差的最大值_。以(x1,y1)和(x2,y2)二点为例，其切比雪夫距离为max(|x2-x1|,|y2-y1|)。切比雪夫距离得名自俄罗斯数学家切比雪夫。

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-22-Chebyshev.png)

> 若将国际象棋棋盘放在二维直角座标系中，格子的边长定义为1，座标的x轴及y轴和棋盘方格平行，原点恰落在某一格的中心点，则王从一个位置走到其他位置需要的步数恰为二个位置的切比雪夫距离，因此切比雪夫距离也称为棋盘距离。例如位置F6和位置E2的切比雪夫距离为4。任何一个不在棋盘边缘的位置，和周围八个位置的切比雪夫距离都是1。

* 二维平面两点a(x1,y1)与b(x2,y2)间的切比雪夫距离

$$
d_{ab} = max(|x_1-x_2|, |y_1 - y_2|)
$$

* 两个n维向量a(x11,x12,…,x1n)与 b(x21,x22,…,x2n)间的切比雪夫距离

{% math %}

d_{ab} = max(|x_{1i}-x_{2i}|)

{% endmath %}

等价于

{% math %}

d_{ab} = \lim\limits_{k \to \infty}(\sum\limits_{i=1}^{n}|x_{1i} - x_{2i}|^k)^{\frac{1}{k}}

{% endmath %}

### 闵可夫斯基距离(Minkowski Distance)
可以将曼哈顿距离，欧几里得距离，切比雪夫距离三种距离的计算公式归纳
为一个计算公式，这就是闵可夫斯基距离(Minkowski Distance)。

$$
d(x, y) = (\sum\limits_{k=1}^n|x_k - y_k|^r)^{\frac{1}{r}}
$$

* $r = 1$, 该公式就是曼哈顿距离
* $r = 2$, 该公式就是曼哈顿距离
* $r = \infty$, 该公式就是切比雪夫距离


### 皮尔逊相关系数(Pearson Correlation Coefficient)

参考:
[皮尔逊相关系数]({{< ref "皮尔逊相关系数.md" >}})

### 余弦相似度(Cosine Similarity)

参考[百度百科](https://baike.baidu.com/item/%E4%BD%99%E5%BC%A6%E7%9B%B8%E4%BC%BC%E5%BA%A6)

> 余弦相似度，又称为余弦相似性，是通过计算两个向量的夹角余弦值来评估他们的相似度。余弦相似度将向量根据坐标值，绘制到向量空间中，如最常见的二维空间。

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-23-cosine-distance.jpg)

* 二维平面上，假设向量a、b的坐标分别为(x1,y1)、(x2,y2) 。则:

$$
cos\theta = \frac{x_1x_2+y_1y_2}{\sqrt{x_1^2 + y_1^2}\sqrt{x_2^2 + y_2^2}}
$$

* 推广到n维空间，计算公式为:

$$
cos\theta = \frac{\sum\limits_1^n(Ai \times Bi)}{\sqrt{\sum\limits_1^nA_i^2}\sqrt{\sum\limits_1^nB_i^2}}
$$

#### 性质

余弦值的范围在[-1,1]之间，值越趋近于1，代表两个向量的方向越接近；越趋近于-1，他们的方向越相反；接近于0，表示两个向量近乎于正交。


### 实战

在介绍了一大堆的相似度计算之后，我们来通过一些例子，说明具体的用法。
假设我们要为一个在线音乐网站的用户推荐乐队。用户可以用1-5颗星来评价他喜欢的乐队，其中包含半颗星(如2.5星)，下表展示了8位用户对8支乐队的评价:

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-23-Recommand-Music-Bannd.png)

图中的`-`表示没有评分。在计算两个用户的距离时，只考虑他们都评价过的乐队。暂时不考虑数据缺失的情况。

数据转换成代码如下:

```python
import json

users_string = '''
{
    "Angelica": {
        "Blues Traveler": 3.5,
        "Broken Bells": 2,
        "Norah Jones": 4.5,
        "Phoenix": 5,
        "Slightly Stoopid": 1.5,
        "The Strokes": 2.5,
        "Vampire Weekend": 2
    },
    "Bill": {
        "Blues Traveler": 2,
        "Broken Bells": 3.5,
        "Deadmau5": 4,
        "Phoenix": 2,
        "Slightly Stoopid": 3.5,
        "Vampire Weekend": 3
    },
    "Chan": {
        "Blues Traveler": 5,
        "Broken Bells": 1,
        "Deadmau5": 1,
        "Norah Jones": 3,
        "Phoenix": 5,
        "Slightly Stoopid": 1
    },
    "Dan": {
        "Blues Traveler": 3,
        "Broken Bells": 4,
        "Deadmau5": 4.5,
        "Phoenix": 3,
        "Slightly Stoopid": 4.5,
        "The Strokes": 4,
        "Vampire Weekend": 2
    },
    "Hailey": {
        "Broken Bells": 4,
        "Deadmau5": 1,
        "Norah Jones": 4,
        "The Strokes": 4,
        "Vampire Weekend": 1
    },
    "Jordyn": {
        "Broken Bells": 4.5,
        "Deadmau5": 4,
        "Norah Jones": 5,
        "Phoenix": 5,
        "Slightly Stoopid": 4.5,
        "The Strokes": 4,
        "Vampire Weekend": 4
    },
    "Sam": {
        "Blues Traveler": 5,
        "Broken Bells": 2,
        "Norah Jones": 3,
        "Phoenix": 5,
        "Slightly Stoopid": 4,
        "The Strokes": 5
    },
    "Veronica": {
        "Blues Traveler": 3,
        "Norah Jones": 5,
        "Phoenix": 4,
        "Slightly Stoopid": 2.5,
        "The Strokes": 3
    }
}
'''

users = json.loads(users_string)
```

计算曼哈顿距离

```python
def manhattan(rating1, rating2):
    """计算曼哈顿距离"""
    distance = 0
    for key in rating1:
        if key in rating2:
            difference = abs(rating1[key] - rating2[key])
            distance += difference
    return distance
```

测试一下效果

```python
>>> manhattan(users['Hailey'], users['Veronica'])
2

>>> manhattan(users['Hailey'], users['Jordyn'])
7.5
```

下面再通过一个函数，实现找出最近的用户距离，该函数会返回一个用户列表，按照距离排序

```python
def computeNearestNeighbor(username, users):
    distances = []
    for user in users:
        if user != username:
            distance = manhattan(users[username], users[user])
            distances.append((distance, user))
    distances.sort()
    return distances
```

测试一下，计算与用户Hailey最相似的用户

```python
>>> computeNearestNeighbor('Hailey', users)
[(2, 'Veronica'),
 (4, 'Chan'),
 (4, 'Sam'),
 (4.5, 'Dan'),
 (5.0, 'Angelica'),
 (5.5, 'Bill'),
 (7.5, 'Jordyn')]
```

最后再来根据距离做推荐

```python
def recommend(username, users):
    # 找到距离最近的用户
    nearest = computeNearestNeighbor(username, users)[0][1]
    recommendations = []
    # 找出这位用户评价过，但自己未评价过的乐队
    neighborRatings = users[nearest]
    userRatings = users[username]
    for artist in neighborRatings:
        if not artist in userRatings:
            recommendations.append((artist, neighborRatings[artist]))
    # 按照评分进行排序
    return sorted(recommendations, key=lambda recommend: recommend[1], reverse=True)
```
为用户Hailey做推荐

```python
>>> recommend('Hailey', users)
[('Phoenix', 4), ('Blues Traveler', 3), ('Slightly Stoopid', 2.5)]
```

闵科夫斯基算法

```python
def minkowski(rating1, rating2, r):
    """闵可夫斯基算法"""
    distance = 0
    for key in rating1:
        if key in rating2:
            distance += pow(abs(rating1[key] - rating2[key]), r)
    return pow(distance, 1.0/r)
```

皮尔逊系数

```python
def pearson(rating1, rating2):
    sum_xy = 0
    sum_x = 0
    sum_y = 0
    sum_x2 = 0
    sum_y2 = 0
    n = 0
    for key in rating1:
        if key in rating2:
            n += 1
            x = rating1[key]
            y = rating2[key]
            sum_xy += x * y
            sum_x += x
            sum_y += y
            sum_x2 += pow(x, 2)
            sum_y2 += pow(y, 2)
    if n == 0:
        return 0
    # now compute denominator
    denominator = (sqrt(sum_x2 - pow(sum_x, 2) / n)
                   * sqrt(sum_y2 - pow(sum_y, 2) / n))
    if denominator == 0:
        return 0
    else:
        return (sum_xy - (sum_x * sum_y) / n) / denominator
```

### 相似度算法选择

根据数据的特性，不同的算法计算的结果会有偏差，但是大体上满足如下规律:

* 如果数据存在"分数膨胀"，就使用皮尔逊相关系数
* 如果数据比较密集，变量之间基本存在公有值，且这些距离数据都是非常重要的，那就使用曼哈顿距离或欧几里得距离
* 如果数据是稀释的，就用余弦定理


### 距离计算库

[`scipy.spatial.distance`](https://docs.scipy.org/doc/scipy/reference/spatial.distance.html)模块包含了各种距离的计算方式。

* scipy.spatial.distance.cityblock 曼哈顿距离
* scipy.spatial.distance.euclidean 欧几里得距离
* scipy.spatial.distance.chebyshev 切比雪夫距离
* scipy.spatial.distance.minkowski 闵可夫斯基距离
* scipy.stats.pearsonr 皮尔逊相关系数

### K最邻近算法

如果只使用最相似的一个用户来做推荐，如果这个用户有特殊的偏好，就会直接反应在推荐内容里。解决方法之一就是寻找多个相似的用户，这里就要用到K最邻近算法了。

完整实现请参考:

<https://github.com/zacharski/pg2dm-python/blob/master/ch2/recommender.py>
