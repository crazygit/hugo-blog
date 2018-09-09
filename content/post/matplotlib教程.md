---
title: matplotlib教程
date: 2017-12-25T11:14:51+08:00
tags:
    - matplotlib
categories:
    - python数据分析
slug: matplotlib-quickstart
description: >
    matplotlib是数据可视化的一个库，功能十分强大，使用它可以很方便的画图各种图形。

---

本文参考:

[Matplotlib 画图教程系列 | 莫烦Python](https://morvanzhou.github.io/tutorials/data-manipulation/plt/)

## 安装

```bash
$ pip install matplotlib
```

## 基本用法

### 画一个基本的线条

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-1, 1, 50)
y = 2*x + 1
# x,y 的值是一一对应的
plt.plot(x, y)
# 让图片显示
plt.show()
```
![效果图](http://images.wiseturtles.com/2017-12-25-line.png)

<!--more-->

### 一次画多个图像(Figure)

默认情况下，一次只会画一张图片。通过`figure`，我们可以一次画多个图像。

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y1 = 2*x + 1
y2 = x**2

# 画第一张图
plt.figure()
plt.plot(x, y1)

# 画第二张图
# num: 设置图片的序号, figsize: 设置图片的大小
plt.figure(num=3, figsize=(8,5))
plt.plot(x, y2)
# 画第二张图的第二根图线
# color: 设置图线的颜色, linewidth: 图线的宽度, linestyle: 图线的样式
plt.plot(x, y1, color='red', linewidth=1.0, linestyle='--')

plt.show()
```

![Figure 1](http://images.wiseturtles.com/2017-12-25-figure_1.png)
![Figure 3](http://images.wiseturtles.com/2017-12-25-figure_3.png)

### 设置x,y轴的坐标和标签

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y1 = 2*x + 1
y2 = x**2

plt.figure(figsize=(9, 5))
plt.plot(x, y2)
plt.plot(x, y1, color='red', linewidth=1.0, linestyle='--')

# 设置x轴的坐标范围在(-1, 2)
plt.xlim(-1, 2)
# 设置y轴的坐标范围在(-2, 3)
plt.ylim(-2, 3)
# 设置x轴的标签
plt.xlabel("I am x")
# 设置y轴的标签
plt.ylabel("I am y")

new_ticks = np.linspace(-1, 2, 5)
# 设置x轴的新坐标
plt.xticks(new_ticks)

# 用文字代替y轴的坐标值, 首尾的$符号表示改变文字的字体
plt.yticks([-2, -1.8, -1, 1.22, 3],
            [r'$really\ bad$', r'$bad$', r'$normal$', r'$good$', r'$really\ good$'])

plt.show()
```
![Figure with custom ticks](http://images.wiseturtles.com/2017-12-25-figsize_ticks.png)

### 设置x,y轴的坐标原点位置和x,y轴的刻度位置

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y1 = 2*x + 1
y2 = x**2

plt.figure(figsize=(9, 5))
plt.plot(x, y2)
plt.plot(x, y1, color='red', linewidth=1.0, linestyle='--')

plt.xlim(-1, 2)
plt.ylim(-2, 3)
plt.xlabel('I am x')
plt.ylabel('I am y')

new_ticks = np.linspace(-1, 2, 5)
plt.xticks(new_ticks)

plt.yticks([-2, -1.8, -1, 1.22, 3],
            [r'$really\ bad$', r'$bad$', r'$normal$', r'$good$', r'$really\ good$'])

# gca  = 'get current axis'
# 获取图片的4个外边框线
ax = plt.gca()
# 去掉右边的边框
ax.spines['right'].set_color('none')
# 去掉上边的边框
ax.spines['top'].set_color('none')
# 设置x轴的坐标刻度在下面
ax.xaxis.set_ticks_position('bottom')
# 设置y轴的坐标刻度在左边
ax.yaxis.set_ticks_position('left')

# 下面两个设置坐标原点的位置(0, 0)
# 设置y轴的值在0刻度
ax.spines['bottom'].set_position(('data', 0))
# 设置x轴的值在0刻度
ax.spines['left'].set_position(('data', 0))

plt.show()
```

![change figure current axis](http://images.wiseturtles.com/2017-12-25-figure_gca.png)


### 添加图例(legend)

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y1 = 2*x + 1
y2 = x**2

plt.figure(figsize=(9, 5))

plt.xlim(-1, 2)
plt.ylim(-2, 3)
plt.xlabel('I am x')
plt.ylabel('I am y')

new_ticks = np.linspace(-1, 2, 5)
plt.xticks(new_ticks)

plt.yticks([-2, -1.8, -1, 1.22, 3],
            [r'$really\ bad$', r'$bad$', r'$normal$', r'$good$', r'$really\ good$'])

# 通过label参数设置线的名称
l1, = plt.plot(x, y2, label='up')
l2, = plt.plot(x, y1, color='red', linewidth=1.0, linestyle='--', label='down')
# 调用legend方法显示图例，如果这里使用了labels参数，会覆盖上面plot函数的label参数，否则图例就是用plot时指定的label。loc参数指定图例的位置
plt.legend(handles=[l1, l2], labels=['aaa', 'bbb'], loc='best')
```

![figure with legend](http://images.wiseturtles.com/2017-12-25-figure_legend.png)

### 添加标注(Annotation)

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y = 2 * x + 1
plt.figure(num=1, figsize=(9, 5))
plt.plot(x, y)

ax = plt.gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.yaxis.set_ticks_position('left')
ax.spines['bottom'].set_position(('data', 0))
ax.spines['left'].set_position(('data', 0))

x0 = 1
y0 = 2 * x0 + 1
plt.scatter(x0, y0, s=50, color='b')
plt.plot([x0, x0], [y0, 0], 'k--', lw=2.5)

# method 1: 添加标注 annotate
plt.annotate(
    r'$2x+1=%s$' % y0,  # 标注的文本
    xy=(x0, y0),        # 标注的位置
    xycoords='data',
    xytext=(+30, -30),  # 标注文本相对于标注位置的偏移
    textcoords="offset points",
    fontsize=16,        # 标注的文字字体大小
    arrowprops=dict(    # 设置标注的箭头样式
        arrowstyle='->',
        connectionstyle='arc3, rad=.2'
    )
)

# method 2: 添加标注 text
plt.text(
    -3.7, 3,  # 标注的位置
    r'$This\ is\ the\ some\ text.\mu\ \sigma_i\ \alpha_t$',  # 标注的文本
    fontdict={'size': 16, 'color': 'r'}   # 标注的字体设置
)

plt.show()
```
![annotate](http://images.wiseturtles.com/2017-12-25-Figure_annotate.png)

### 能见度(tick)

当图片中的内容较多，相互遮盖时，我们可以通过设置相关内容的透明度来使图片更易于观察，即是通过bbox(backgroud box)参数设置来调节图像信息.

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y = 0.1 * x
plt.figure()
#  参数zorder指定plot在z轴方向排序
plt.plot(x, y, linewidth=10, zorder=1)

plt.ylim(-2, 2)
ax = plt.gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.spines['right'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.yaxis.set_ticks_position('left')
ax.spines['bottom'].set_position(('data', 0))
ax.spines['left'].set_position(('data', 0))

# 获取所有x轴和y轴的tick label
for label in ax.get_xticklabels() + ax.get_yticklabels():
    label.set_fontsize(12)  # 设置label的文字大小
    label.set_bbox(dict(
        facecolor='white',  # 设置前景色
        edgecolor='None',   # 设置边框为无
        alpha=0.7,   # 设置透明度
        zorder=2
    ))

plt.show()
```
![figure tick](http://images.wiseturtles.com/2017-12-25-figure_tick.png)


## 画图的种类

### Scatter 散点图

```python
import matplotlib.pyplot as plt
import numpy as np

n = 1024

X = np.random.normal(0, 1, n)
Y = np.random.normal(0, 1, n)
T = np.arctan2(Y, X)  # 用于设置颜色

plt.scatter(X, Y, s=75, c=T, alpha=0.5)
plt.xlim(-1.5, 1.5)
plt.ylim(-1.5, 1.5)

# 去掉x,y轴的坐标刻度
plt.xticks(())
plt.yticks(())

plt.show()
```
![figure scatter](http://images.wiseturtles.com/2017-12-25-Figure_scatter.png)


### Bar柱状图

```python
import matplotlib.pyplot as plt
import numpy as np

n = 12

X = np.arange(n)

Y1 = (1 - X / float(n)) * np.random.uniform(0.5, 1.0, n)
Y2 = (1 - X / float(n)) * np.random.uniform(0.5, 1.0, n)

plt.bar(X, +Y1, facecolor='#9999ff', edgecolor='white')
plt.bar(X, -Y2, facecolor='#ff9999', edgecolor='white')

for x, y in zip(X, Y1):
    # ha: horizontal alignment
    # va: vertical alignment
    plt.text(x + 0.05, y + 0.02, '%.2f' % y, ha='center', va='bottom')

for x, y in zip(X, Y2):
    # ha: horizontal alignment
    # va: vertical alignment
    plt.text(x + 0.05, -y - 0.02, '-%.2f' % y, ha='center', va='top')

plt.xlim(-0.5, n)
plt.xticks(())
plt.ylim(-1.25, +1, 25)
plt.yticks(())

plt.show()
```

![figure bar](http://images.wiseturtles.com/2017-12-25-Figure_bar.png)

### Contours 等高线图

```python
import matplotlib.pyplot as plt
import numpy as np

def f(x, y):
    # the height function
    return (1 - x / 2 + x ** 5 + y ** 3) * np.exp(-x ** 2 - y ** 2)

n = 256
x = np.linspace(-3, 3, n)
y = np.linspace(-3, 3, n)

#X, Y为shape为(256, 256)的表
X, Y = np.meshgrid(x, y)

# 接下来进行颜色填充。使用函数plt.contourf把颜色加进去。
# 位置参数分别为: X, Y, f(X,Y)。透明度0.75，并将 f(X,Y) 的值对应到color map的暖色组中寻找对应颜色。
# use plt.contourf to filling contours
# X, Y and value for (X,Y) point
# 其中，8代表等高线的密集程度，这里被分为10个部分。如果是0，则图像被一分为二
plt.contourf(X, Y, f(X, Y), 8, alpha=.75, cmap=plt.cm.hot)

# 接下来进行等高线绘制。使用plt.contour函数划线。位置参数为：X, Y, f(X,Y)。颜色选黑色，线条宽度选0.5
# use plt.contour to add contour lines
C = plt.contour(X, Y, f(X, Y), 8, colors='black', linewidth=.5)

# 最后加入Label，inline控制是否将Label画在线里面，字体大小为10。并将坐标轴隐藏：
plt.clabel(C, inline=True, fontsize=10)
plt.xticks(())
plt.yticks(())

plt.show()
```

![figure contours](http://images.wiseturtles.com/2017-12-25-Figure_Contours.png)

### Image图片
```python
import matplotlib.pyplot as plt
import numpy as np

# 三行三列的格子，a代表每一个值，图像右边有一个注释，白色代表值最大的地方，颜色越深值越小。
a = np.array([0.313660827978, 0.365348418405, 0.423733120134,
              0.365348418405, 0.439599930621, 0.525083754405,
              0.423733120134, 0.525083754405, 0.651536351379]).reshape(3,3)

# 之前选cmap的参数时用的是：cmap=plt.cmap.bone，而现在，我们可以直接用单引号传入参数。
# origin='lower'代表的就是选择的原点的位置。
# interpolation参数设置出图的方式, 具体可以参考:
# https://matplotlib.org/examples/images_contours_and_fields/interpolation_methods.html
plt.imshow(a, interpolation='nearest', cmap='bone', origin='lower')

# 添加右边显示的colorbar ，其中我们添加一个shrink参数，使colorbar的长度变短为原来的92%：
plt.colorbar(shrink=.92)

plt.xticks(())
plt.yticks(())
plt.show()
```
![figure image](http://images.wiseturtles.com/2017-12-25-Figure_image.png)


### 3D图像

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure()
ax = Axes3D(fig)
ax.scatter(1, 1, 1)

# X, Y value
X = np.arange(-4, 4, 0.25)
Y = np.arange(-4, 4, 0.25)
X, Y = np.meshgrid(X, Y)    # x-y 平面的网格
R = np.sqrt(X ** 2 + Y ** 2)
# height value
Z = np.sin(R)

# rstride 和 cstride 分别代表 row 和 column 的跨度。
ax.plot_surface(X, Y, Z, rstride=1, cstride=1, cmap=plt.get_cmap('rainbow'))
# 添加 XY 平面的等高线,如果zdir 选择了x，那么效果将会是对于 XZ 平面的投影
ax.contourf(X, Y, Z, zdir='z', offset=-2, cmap=plt.get_cmap('rainbow'))
plt.show()
```
![figure 3d](http://images.wiseturtles.com/2017-12-25-Figure_3d.png)


## 多图合并显示（subplot）

### 多合一显示
#### 均匀分割

```python
import matplotlib.pyplot as plt

plt.figure()

# 分成两行两列，画在第一个图的位置
plt.subplot(2, 2, 1)
plt.plot([0, 1], [0, 1])

plt.subplot(2, 2, 2)
plt.plot([0, 1], [0, 2])

# `223` 简单的写法，分成两行两列，画在第三个图的位置
plt.subplot(223)
plt.plot([0, 1], [0, 3])

plt.subplot(224)
plt.plot([0, 1], [0, 4])

plt.show()
```
![figure subplot 1](http://images.wiseturtles.com/2017-12-26-Figure_subplot_1.png)

#### 不均匀分割

```python
import matplotlib.pyplot as plt

plt.figure()

# 分成两行一列，画在第一个位置
plt.subplot(2, 1, 1)
plt.plot([0, 1], [0, 1])

# 分成两行三列，注意是从位置4开始
plt.subplot(2, 3, 4)
plt.plot([0, 1], [0, 2])

plt.subplot(235)
plt.plot([0, 1], [0, 3])

plt.subplot(236)
plt.plot([0, 1], [0, 4])

plt.show()
```
![figure subplot 2](http://images.wiseturtles.com/2017-12-26-Figure_subplot_2.png)

### 分格显示

#### 方法1: subplot2grid

```python
import matplotlib.pyplot as plt

plt.figure()

# 将figure分成3行3列，从(0,0)开始， 横跨3列，1行
ax1 = plt.subplot2grid((3, 3), (0, 0), colspan=3, rowspan=1)
ax2 = plt.subplot2grid((3, 3), (1, 0), colspan=2)
ax3 = plt.subplot2grid((3, 3), (1, 2), rowspan=2)
ax4 = plt.subplot2grid((3, 3), (2, 0))
ax5 = plt.subplot2grid((3, 3), (2, 1))

ax1.plot([1, 2], [1, 2])
ax1.set_title("ax1 title")
ax2.set_title("ax2 title")
ax3.set_title("ax3 title")
ax4.set_title("ax4 title")
ax5.set_title("ax5 title")

plt.show()
```
![figure subplot2grid](http://images.wiseturtles.com/2017-12-26-Figure_subplot2grid.png)

#### 方法2: gridspec
```python
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec

plt.figure()

# 将figure分成三行三列
gs = gridspec.GridSpec(3, 3)
# ax1, 第一行，所有列
ax1 = plt.subplot(gs[0, :])
# ax2, 第二行，前两列
ax2 = plt.subplot(gs[1, :2])
# ax2, 第二行到最后一行，第三列
ax3 = plt.subplot(gs[1:, 2])
# ax2, 最后一行，第一列
ax4 = plt.subplot(gs[-1, 0])
ax5 = plt.subplot(gs[-1, -2])

ax1.plot([1, 2], [1, 2])
ax1.set_title('ax1 title')
ax2.set_title("ax2 title")
ax3.set_title("ax3 title")
ax4.set_title("ax4 title")
ax5.set_title("ax5 title")

plt.show()
```

![figure gridspec](http://images.wiseturtles.com/2017-12-26-figure_gridspec.png)

#### 方法3: subplots

```python
import matplotlib.pyplot as plt

plt.figure()

# 使用plt.subplots建立一个2行2列的图像窗口
# sharex=True表示共享x轴坐标, sharey=True表示共享y轴坐标.
# f代表figure对象
# ((ax11, ax12), (ax13, ax14))表示第1行从左至右依次放ax11和ax12, 第2行从左至右依次放ax13和ax14.
f, ((ax11, ax12), (ax13, ax14)) = plt.subplots(2, 2, sharex=True, sharey=True)

ax11.scatter([1, 2], [1, 2])

# 紧凑显示图像
plt.tight_layout()

plt.show()
```

![figure subplots](http://images.wiseturtles.com/2017-12-26-Figure_subplots.png)

### 图中图

```python
import matplotlib.pyplot as plt

fig = plt.figure()

# 创建数据
x = [1, 2, 3, 4, 5, 6, 7]
y = [1, 3, 4, 2, 5, 8, 6]

# 4个值都是占整个figure坐标系的百分比。
# 在这里，假设figure的大小是10x10，那么大图就被包含在由(1, 1)开始，宽8，高8的坐标系内。
left, bottom, width, height = 0.1, 0.1, 0.8, 0.8

# 将大图坐标系添加到figure中，颜色为r(red)，取名为title

ax1 = fig.add_axes([left, bottom, width, height])
ax1.plot(x, y, 'r')
ax1.set_xlabel('x')
ax1.set_ylabel('y')
ax1.set_title('title')

# 绘制左上角的小图，步骤和绘制大图一样，注意坐标系位置和大小的改变
left, bottom, width, height = 0.2, 0.6, 0.25, 0.25
ax2 = fig.add_axes([left, bottom, width, height])
ax2.plot(y, x, 'b')
ax2.set_xlabel('x')
ax2.set_ylabel('y')
ax2.set_title('title inside 1')

# 绘制右下角的小图。这里我们采用一种更简单方法，即直接往plt里添加新的坐标系：

plt.axes([0.6, 0.2, 0.25, 0.25])
plt.plot(y[::-1], x, 'g') # 注意对y进行了逆序处理
plt.xlabel('x')
plt.ylabel('y')
plt.title('title inside 2')

plt.show()
```
![figure insiede](http://images.wiseturtles.com/2017-12-26-Figure_inside.png)

### 次坐标轴

有时候我们会用到次坐标轴，即在同个图上有第2个y轴存在

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(0, 10, 0.1)
y1 = 0.05 * x ** 2
y2 = -1 * y1

# 获取figure默认的坐标系 ax1
fig, ax1 = plt.subplots()
# 对ax1调用twinx()方法，生成如同镜面效果后的ax2
ax2 = ax1.twinx()

ax1.plot(x, y1, 'g-')  # green, solid line
ax1.set_xlabel('X data')
ax1.set_ylabel('Y1 data', color='g')

ax2.plot(x, y2, 'b-')  # blue
ax2.set_ylabel('Y2 data', color='b')

plt.show()
```
![figure twinx](http://images.wiseturtles.com/2017-12-26-Figure_twinx.png)

## 动画(Animation)

```python
from matplotlib import pyplot as plt
from matplotlib import animation
import numpy as np

fig, ax = plt.subplots()

# 数据是一个0~2π内的正弦曲线
x = np.arange(0, 2 * np.pi, 0.01)
line, = ax.plot(x, np.sin(x))


# 构造自定义动画函数animate，用来更新每一帧上各个x对应的y坐标值，参数表示第i帧
def animate(i):
    line.set_ydata(np.sin(x + i / 10.0))
    return line,


# 构造开始帧函数init
def init():
    line.set_ydata(np.sin(x))
    return line,


# 参数说明:
# fig 进行动画绘制的figure
# func 自定义动画函数，即传入刚定义的函数animate
# frames 动画长度，一次循环包含的帧数
# init_func 自定义开始帧，即传入刚定义的函数init
# interval 更新频率，以ms计
# blit 选择更新所有点，还是仅更新产生变化的点。应选择True，但mac用户请选择False，否则无法显示动画
ani = animation.FuncAnimation(fig=fig,
                              func=animate,
                              frames=100,
                              init_func=init,
                              interval=20,
                              blit=True)

# 保存为MP4格式
# ani.save('basic_animation.mp4', fps=30, extra_args=['-vcodec', 'libx264'])

plt.show()
```

{%
iframe http://images.wiseturtles.com/basic_animation.mp4 100% 100%
%}

## 扩展阅读

* [Matplotlib gallery](https://matplotlib.org/gallery.html) 可以查看各种图的效果和代码
* [Matplotlib 教程](https://liam0205.me/2014/09/11/matplotlib-tutorial-zh-cn/)
* [十分钟入门Matplotlib](http://codingpy.com/article/a-quick-intro-to-matplotlib/)
