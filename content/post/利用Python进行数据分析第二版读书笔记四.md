---
title: 《利用Python进行数据分析第二版》读书笔记四
date: 2018-01-09T09:50:48+08:00
tags:
    - matplotlib
    - pandas
    - seaborn
categories:
    - python数据分析
slug: python-for-data-analysis-2nd-edition-note-four
description: >
    使用matplotlib, pandas, seaborn进行数据可视化展示

---

# 绘图和可视化

绘图时，在`ipython`里运行

```python
>>> %matplotlib
Using matplotlib backend: TkAgg
```

可以直接绘制图形，不然每次绘制图形最后必须调用`plt.show()`才能看到图形。

*本文里使用的CSV测试数据可以从这里[下载](https://github.com/wesm/pydata-book)*

## matplotlib简单入门

关于`matplotlib`的使用入门，可以参考:

[matplotlib教程]({{< ref "matplotlib教程.md" >}})

<!--more-->
## 使用pandas和seaborn绘图

数据可视化方面, `matplotlib`算是比较底层的工具了。 `pandas`也简单内置了一些常用的画图方法。`seaborn`则是另一个专门用于数据统计的可视化库。

**注意**

在代码中引入`searborn`会修改`matplotlib`默认的画图颜色和样式，即便不使用`searborn`画图，也可以引入它来提高画图的美化效果。

### 线性图

`Series`和`DataFrame`都有`plot`方法来绘制一些基本图形，默认调用`plot`方法会绘制线性图

```python
>>> s = pd.Series(np.random.randn(10).cumsum(), index=np.arange(0, 100, 10))

>>> s
0     0.471154
10    0.037093
20   -0.130165
30   -0.907087
40   -0.495938
50    0.525166
60    0.822416
70    0.667376
80    1.488170
90    2.041607
dtype: float64

>>> s.plot()
```

![SeriesPlot](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-series_plot.png)

默认使用`Series`的`index`作为`x`轴。也可以通过参数`use_index=False`来设置不用`Series`的`index`作为`x`轴。其它可以定制的参数如下:

![Series Plot Method](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-series_plot_method_args.png)

`DataFrame`的`plot`方法会把每一列绘制成一条线，并且自动绘制图例

```python
>>> df = pd.DataFrame(np.random.randn(10, 4).cumsum(0),
...                   columns=['A', 'B', 'C', 'D'],
...                    index=np.arange(0, 100, 10))
...

>>> df.plot()
```

![DataFrame plot](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-DataFrame_plot.png)

`DataFrame`的`plot`方法常用的参数如下：
![DataFrame plot method](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-DataFrame_plot_method_args.png)


### 条形图(Bar Plots)

`plot.bar()`和`plot.barh()`可以用来绘垂直和水平的制柱状图。

大多数pandas的`plot`方法都可以接受`ax`参数（一个matplotlib subplot对象），让我们更方便在表格样式中绘制子图。

```python
>>> fig, axes = plt.subplots(2, 1)

>>> data = pd.Series(np.random.rand(16), index=list('abcdefghijklmnop'))

>>> data.plot.bar(ax=axes[0], color='k', alpha=0.7)
<matplotlib.axes._subplots.AxesSubplot at 0x1076809e8>

>>> data.plot.barh(ax=axes[1], color='k', alpha=0.7)
<matplotlib.axes._subplots.AxesSubplot at 0x10be498d0>

```
![Series plot bar](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-Series_plot_bar.png)

`DataFrame`在做柱状图时会自动按照列分组

```python
>>> df = pd.DataFrame(np.random.rand(6, 4),
...                   index=['one', 'two', 'three', 'four', 'five', 'six'],
...                   columns=pd.Index(['A', 'B', 'C', 'D'], name='Genus'))
...

>>> df
Genus         A         B         C         D
one    0.682972  0.387897  0.052612  0.249418
two    0.628619  0.415778  0.412352  0.455799
three  0.330864  0.472638  0.780744  0.606024
four   0.665816  0.172358  0.294163  0.038945
five   0.390196  0.354316  0.566566  0.592186
six    0.556651  0.721578  0.857398  0.939181

>>> df.plot.bar()
```
![DataFrame plot bar](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-DataFrame_plot_bar_vertical.png)

我们可以可以通过参数`stacked=True`来创建堆叠的柱状图

```python
>>> df.plot.barh(stacked=True, alpha=0.5)
```

![DataFrame plot barh](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-DataFrame_plot_barh.png)


**小贴士**:

一个比较常见的关于`Series`绘制柱状图就是使用`value_counts`, 可以很直观的看到统计的数据

```
>>> s = pd.Series(np.random.randint(0, 10, 20))

>>> s
0     4
1     9
2     0
3     4
4     9
5     0
6     2
7     8
8     0
9     6
10    3
11    6
12    9
13    4
14    7
15    1
16    1
17    3
18    7
19    6

>>> s.value_counts().plot.bar()
```

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-Series_plot_bar_value_counts.png)


让我们来看一个统计聚会人数多少和星期几的关系,

```python
>>> tips = pd.read_csv('examples/tips.csv')

# size: 聚会人数
# day: 星期几
>>> tips.head()
   total_bill   tip smoker  day    time  size
0       16.99  1.01     No  Sun  Dinner     2
1       10.34  1.66     No  Sun  Dinner     3
2       21.01  3.50     No  Sun  Dinner     3
3       23.68  3.31     No  Sun  Dinner     2
4       24.59  3.61     No  Sun  Dinner     4

# 可以得知聚会人数介于1-6之间
>>> tips['size'].unique()
array([2, 3, 4, 1, 6, 5])

>>> tips[tips['size'] == 1]
     total_bill   tip smoker   day    time  size
67         3.07  1.00    Yes   Sat  Dinner     1
82        10.07  1.83     No  Thur   Lunch     1
111        7.25  1.00     No   Sat  Dinner     1
222        8.58  1.92    Yes   Fri   Lunch     1

# 以星期为index, size为列统计数据
>>> party_counts = pd.crosstab(tips['day'], tips['size'])

>>> party_counts
size  1   2   3   4  5  6
day
Fri   1  16   1   1  0  0
Sat   2  53  18  13  1  0
Sun   0  39  15  18  3  1
Thur  1  48   4   5  1  3

# 排除1个人的情况
>>> party_counts = party_counts.loc[:, 2:5]

>>> party_counts
size   2   3   4  5
day
Fri   16   1   1  0
Sat   53  18  13  1
Sun   39  15  18  3
Thur  48   4   5  1

# 计算星期的总人数
>>> party_counts.sum(1)
day
Fri     18
Sat     85
Sun     75
Thur    58
dtype: int64

# 计算size占星期的百分比
>>> party_pcts = party_counts.div(party_counts.sum(1), axis=0)

>>> party_pcts
size         2         3         4         5
day
Fri   0.888889  0.055556  0.055556  0.000000
Sat   0.623529  0.211765  0.152941  0.011765
Sun   0.520000  0.200000  0.240000  0.040000
Thur  0.827586  0.068966  0.086207  0.017241

party_pcts.plot.bar()
```

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-tips_figure.png)

从上图可以看出，周末的时候，聚会人数(size)多的是在周末。


当数据需要聚合和汇总时，使用`seaborn`更方便一点。让我们来计算每天的小费情况

```python
>>> import seaborn as sns

>>> tips['tip_pct'] = tips['tip'] / (tips['total_bill'] - tips['tip'])

>>> tips.head()
   total_bill   tip smoker  day    time  size   tip_pct
0       16.99  1.01     No  Sun  Dinner     2  0.063204
1       10.34  1.66     No  Sun  Dinner     3  0.191244
2       21.01  3.50     No  Sun  Dinner     3  0.199886
3       23.68  3.31     No  Sun  Dinner     2  0.162494
4       24.59  3.61     No  Sun  Dinner     4  0.172069

>>> sns.barplot(x='tip_pct', y='day', data=tips, orient='h')
```
![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_barplot.png)

我们还可以通过`hue`参数来让它来添加额外的分类

```python
>>> sns.barplot(x='tip_pct', y='day', hue='time', data=tips, orient='h')
```

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_bar_plot_hue.png)

我们还可以改变`searborn`的绘图样式

```python
>>> sns.set(style="whitegrid")
```

### 柱状图和密度图(Histograms and Density Plots)

使用`Series`和`DataFrame`的`plot`函数

首先我们看一下小费百分比的分布区间

```python
>>> pd.cut(tips['tip_pct'], 50)
0      (0.0345, 0.0853]
1         (0.182, 0.23]
2         (0.182, 0.23]
3        (0.134, 0.182]
4        (0.134, 0.182]
5         (0.182, 0.23]
6        (0.278, 0.327]
7       (0.0853, 0.134]
8        (0.134, 0.182]
9        (0.278, 0.327]
10        (0.182, 0.23]
11       (0.134, 0.182]
12      (0.0853, 0.134]
13        (0.182, 0.23]
14        (0.23, 0.278]
15        (0.182, 0.23]
16        (0.182, 0.23]
17       (0.278, 0.327]
18        (0.23, 0.278]
19        (0.182, 0.23]
20       (0.278, 0.327]
21       (0.134, 0.182]
22       (0.134, 0.182]
23        (0.23, 0.278]
24        (0.182, 0.23]
25       (0.134, 0.182]
26       (0.134, 0.182]
27        (0.182, 0.23]
28        (0.23, 0.278]
29       (0.134, 0.182]
             ...
214      (0.278, 0.327]
215     (0.0853, 0.134]
216     (0.0853, 0.134]
217      (0.134, 0.182]
218       (0.182, 0.23]
219     (0.0853, 0.134]
220       (0.182, 0.23]
221      (0.327, 0.375]
222      (0.278, 0.327]
223       (0.23, 0.278]
224     (0.0853, 0.134]
225      (0.134, 0.182]
226       (0.23, 0.278]
227      (0.134, 0.182]
228       (0.23, 0.278]
229      (0.134, 0.182]
230     (0.0853, 0.134]
231       (0.23, 0.278]
232      (0.375, 0.423]
233      (0.134, 0.182]
234       (0.23, 0.278]
235      (0.134, 0.182]
236     (0.0853, 0.134]
237    (0.0345, 0.0853]
238      (0.134, 0.182]
239       (0.23, 0.278]
240    (0.0345, 0.0853]
241     (0.0853, 0.134]
242     (0.0853, 0.134]
243       (0.182, 0.23]
Name: tip_pct, Length: 244, dtype: category
Categories (50, interval[float64]): [(0.0345, 0.0853] < (0.0853, 0.134] < (0.134, 0.182] <
                                     (0.182, 0.23] ... (2.259, 2.307] < (2.307, 2.356] <
                                     (2.356, 2.404] < (2.404, 2.452]]

>>> pd.value_counts(bins)
(0.134, 0.182]      76
(0.182, 0.23]       57
(0.23, 0.278]       41
(0.0853, 0.134]     33
(0.278, 0.327]      15
(0.0345, 0.0853]    12
(0.327, 0.375]       4
(0.375, 0.423]       3
(0.713, 0.762]       1
(0.472, 0.52]        1
(2.404, 2.452]       1
(1.873, 1.921]       0
(1.969, 2.018]       0
(0.81, 0.858]        0
(0.762, 0.81]        0
(1.921, 1.969]       0
(0.665, 0.713]       0
(0.617, 0.665]       0
(0.568, 0.617]       0
(0.52, 0.568]        0
(0.423, 0.472]       0
(0.907, 0.955]       0
(2.018, 2.066]       0
(2.066, 2.114]       0
(2.114, 2.163]       0
(2.163, 2.211]       0
(2.211, 2.259]       0
(2.259, 2.307]       0
(2.307, 2.356]       0
(0.858, 0.907]       0
(0.955, 1.003]       0
(1.824, 1.873]       0
(1.438, 1.486]       0
(1.776, 1.824]       0
(1.728, 1.776]       0
(1.679, 1.728]       0
(1.631, 1.679]       0
(1.583, 1.631]       0
(1.535, 1.583]       0
(1.486, 1.535]       0
(1.39, 1.438]        0
(1.003, 1.051]       0
(1.341, 1.39]        0
(1.293, 1.341]       0
(1.245, 1.293]       0
(2.356, 2.404]       0
(1.148, 1.196]       0
(1.1, 1.148]         0
(1.051, 1.1]         0
(1.196, 1.245]       0
Name: tip_pct, dtype: int64
```
从上面可以看出，小费百分比在(0.134, 0.182]里的最多，下面的图形也能反应这一点

```python
>>> tips['tip_pct'].plot.hist(bins=50)
<matplotlib.axes._subplots.AxesSubplot at 0x12607ce10>
```
![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_plot_hist.png)

```python
>>> tips['tip_pct'].plot.density()
<matplotlib.axes._subplots.AxesSubplot at 0x12607ce10>
```

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_plot_density.png)

使用`searborn`的`distplot`方法更简便，可以同时在一个图里绘制柱状图和密度曲线

```python
>>> comp1 = np.random.normal(0, 1, size=200)

>>> comp2 = np.random.normal(10, 2, size=200)

>>> values = pd.Series(np.concatenate([comp1, comp2]))

>>> sns.distplot(values, bins=100, color='k')
<matplotlib.axes._subplots.AxesSubplot at 0x120e1c588>
```
![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-searborn_distplot.png)


### 散点图或点图(Scatter or Point Plots)

使用`seaborn`的`regplot`函数，不仅可以画散点图，还可以绘制一个线性回归线

```python
>>> macro = pd.read_csv('examples/macrodata.csv')
>>> data = macro[['cpi', 'm1', 'tbilrate', 'unemp']]

>>> trans_data = np.log(data).diff().dropna()

>>> trans_data[-5:]
          cpi        m1  tbilrate     unemp
198 -0.007904  0.045361 -0.396881  0.105361
199 -0.021979  0.066753 -2.277267  0.139762
200  0.002340  0.010286  0.606136  0.160343
201  0.008419  0.037461 -0.200671  0.127339
202  0.008894  0.012202 -0.405465  0.042560

# 以m1为x轴，unemp为y轴
>>> sns.regplot('m1', 'unemp', data=trans_data)
<matplotlib.axes._subplots.AxesSubplot at 0x116c026d8>

>>> plt.title('Changes in log %s versus log %s' % ('m1', 'unemp'))
Text(0.5,1,'Changes in log m1 versus log unemp')
```
![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_regplot.png)

使用`pairplot`函数可以绘制散点图矩阵

```python
>>> sns.pairplot(trans_data, diag_kind='kde', plot_kws={'alpha': 0.2})
<seaborn.axisgrid.PairGrid at 0x12760e0b8>
```
![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_pairplot.png)


### 多面网格和类别数据(Facet Grids and Categorical Data)


如果遇到数据集需要额外的分组维度，可以利用多面网格。seaborn有一个有用的内建函数`factorplot`

从`day/time/smoker`来展示小费的百分比

```python
>>> tips = pd.read_csv('examples/tips.csv')

>>> tips['tip_pct'] = tips['tip'] / (tips['total_bill'] - tips['tip'])

>>> sns.factorplot(x='day', y='tip_pct', hue='time', col='smoker', kind='bar', data=tips[tips.tip_pct < 1])
<seaborn.axisgrid.FacetGrid at 0x12724b390>
```

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_factorplot.png)

除了按照`time`作为bar分组之外，我们还可以通过添加参数`row=time`添加额外的面

```python
>>> sns.factorplot(x='day', y='tip_pct', row='time', col='smoker', kind='bar', data=tips[tips.tip_pct < 1])
<seaborn.axisgrid.FacetGrid at 0x1261bc6d8>
```
![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_factorplot_rows.png)

`factorplot`还支持一些其它的绘图参数, 如绘制box图

```python
>>> sns.factorplot(x='tip_pct', y='day', kind='box', data=tips[tips.tip_pct < 0.5])
<seaborn.axisgrid.FacetGrid at 0x127caeac8>
```
![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-09-seaborn_factorplot_kind.png)


## 其它的python可视化工具

除了上面提到的之外，还有许多开源的可视化工具值得我们去探索

* [Bokeh](https://bokeh.pydata.org/en/latest/)
* [Plotly](https://plot.ly/python/)
* [Graph](https://gephi.org/)

对于制作用于打印或网页的静态图形，作者推荐使用matplotlib以及pandas和seaborn这样的工具。对于其他的数据可视化需求，去学一个有用的工具可能会有帮助。
