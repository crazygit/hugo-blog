---
title: 《Fast Data Processing with Spark 2（Third Edition)》读书笔记一
date: 2018-01-11T18:48:27+08:00
tags:
    - spark
categories:
    - spark
slug: fast-data-processing-with-spark2-note-one
description: >
    快速了解Spark的安装，Spark Shell使用和SparkSession对象
---

本书其它笔记[Fast Data Processing with Spark 2（Third Edition)》读书笔记目]({{< ref "FastDataProcessingwithSpark2读书笔记目录.md" >}})

## Spark概览

> Apache Spark is a fast and general-purpose cluster computing system

本文基于目前最新的spark版本`2.2.1`整理，以`Python3.6`为例，在Mac单机情况下使用。

## 安装

### 部署环境需求

* Window, Linux, Mac均可
* Java8+(需要设置`JAVA_HOME`环境变量)

下面三个根据自己习惯使用的语言选择安装

* Python2.7+/3.4+
* R 3.1+
* 为了使用Scala API, 需要安装完整的Scala 2.11.x版本

<!--more-->

### 解压安装

从[Spark官网](https://spark.apache.org/downloads.html)选择合适的版本下载。

```bash
$ wget https://www.apache.org/dyn/closer.lua/spark/spark-2.2.1/spark-2.2.1-bin-hadoop2.7.tgz

$ tar -zxvf spark-2.2.1-bin-hadoop2.7.tgz
```
解压后即可使用, 不过在使用之前，让我们先配置几个常用的环境变量:

```bash
# SPARK_HOME根据本机实际情况修改路径
export SPARK_HOME="/path/to/spark-2.2.1-bin-hadoop2.7"

# 当用pyspark时默认使用ipyton
export PYSPARK_DRIVER_PYTHON="ipython"

# 如果习惯使用Jupyter, 可以添加下面的配置，那么在用pyspark时，默认会调用Jupyter notebook，否则为默认的ipython console
export PYSPARK_DRIVER_PYTHON_OPTS="notebook"

# 添加到PATH，方便随时调用pyspark命令
export PATH="$SPARK_HOME/bin:$PATH"
```

关于Spark集群的搭建，练习阶段暂时不会使用到，等以后需要再补充。

## 快速开始

### 运行例子和交互式shell

Spark内置了一些Scala, Java列子，同时也有一些Python和R语言的例子在`example/src/main`目录里。

为了运行Scala, Java 例子，在`$SPARK_HOME`目录下, 用`bin/run-example <class> [params]`形式运行。如

```bash
./bin/run-example SparkPi 10
```

也可以启动一个Scala语言的交互式的Shell

```bash
./bin/spark-shell --master local[4]
```

`--master`选项设置在本地以两个线程运行。 `local[N]`中的`N`一般设置为本机的CPU核数。更多关于`--master`的选项可以参考[Master URLs](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)

启动Python语言的交互式Shell

```bash
./bin/pyspark --master local[4]
```
运行Python例子

```bash
./bin/spark-submit examples/src/main/python/pi.py 10
```

启动R语言的交互式Shell

```
./bin/sparkR --master local[4]
```

R语言的例子

```
./bin/spark-submit examples/src/main/r/dataframe.R
```


### 使用Spark shell做交互式分析

进入到`$SPARK_HOME`目录下, 启动`pyspark`

```bash
$ ./bin/pyspark
```

```python
# 加载README.md文件
>>> textFile = spark.read.text("README.md")

# 统计文件行数
>>> textFile.count()
103

# 获取第一行
>>> textFile.first()
Row(value='# Apache Spark')

# 过滤出包含"Spark"的行
>>> linesWithSpark = textFile.filter(textFile.value.contains("Spark"))

# 统计包含"Spark"的行的行数
>>> textFile.filter(textFile.value.contains("Spark")).count()

>>> from pyspark.sql.functions import *

# 获取包含单词最多行的单词个数
>>> textFile.select(size(split(textFile.value, "\s+")).name("numWords")).agg(max
... (col("numWords"))).collect()
[Row(max(numWords)=22)]

# 统计单词出现的频次
>>> wordCounts = textFile.select(explode(split(textFile.value, "\s+")).name("wor
... d")).groupBy("word").count()

>>> wordCounts.collect()
[Row(word='online', count=1),
 Row(word='graphs', count=1),
 ...
]

# 缓存
>>> linesWithSpark.cache()
DataFrame[value: string]

>>> linesWithSpark.count()
20
```

在使用Spark Shell时，也会开启一个Spark monitor UI，默认在本地4040端口运行。

![Spark Monitor UI](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-11-SparkMonitorUI.png)

### 创建一个简单的应用

一个简单的应用统计文件中包含字符"a"和"b"的行数的脚本`SimpleApp.py`

```bash
$ cat SimpleApp.py
"""SimpleApp.py"""
from pyspark.sql import SparkSession

appName='Simple Application'
logFile = "README.md"  # Should be some file on your system
spark = SparkSession.builder.appName(appName).getOrCreate()
logData = spark.read.text(logFile).cache()

numAs = logData.filter(logData.value.contains('a')).count()
numBs = logData.filter(logData.value.contains('b')).count()

print("Lines with a: %i, lines with b: %i" % (numAs, numBs))

spark.stop()
```
运行应用

```bash
$ ./bin/spark-submit --master "local[4]" SimpleApp.py
```


## 创建`SparkSession`对象

一个`SparkSession`对象表示与本地(或远程)的Spark集群连接，是与Spark交互的主要入口。

为了建立到Spark的连接，`SparkSession`需要配置如下信息:

* **Master URL**: spark服务器的连接地址, 具体可参考[Master URLs](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)
* **Application Name**: 便于识别的应用名
* **Spark Home**: spark的安装路径
* **JARs**: 任务依赖的JAR包路径

### `SparkSession` vs `SparkContext`

`SparkSession`和`SparkContext`两者之间有什么关系呢？
让我们回顾一下spark的发展历史，看能不能找到一点线索:


在Spark 2.0.0 之前。主要有三个连接对象:

* `SparkContext`: 主要连接Spark执行环境和完成进行创建`RDDs`等其它功能
* `SqlContext`: 和SparkSQL一起在`SparkContext`背后工作
* `HiveContext`: 和`Hive`仓库交互

Saprk 2.0.0之后，引入了`Datasets/DataFrames`作为主要的分数式数据抽象接口。`SparkSession`对象成为了Spark操作的主要入口。有几点需要注意:

1. 在Scala和Java中`Datasets`是主要的数据抽象类型。而在R和Python语言中，`DataFrame`是主要的抽象类型。它们在API操作上基本没有区别(可以认为是一个东西，只是在不同语言里叫法不一样)

2. 尽管在2.0.0之后，`Datasets/DataFrames`作为新的接口形式，但是`RDDs`依然在被使用，所以当操作`RDDs`，主要使用`SparkContext`。

3. `SparkSession`对象包含了`SparkContext`对象。在Spark 2.0.0版本之后，`SparkContext`仍然是作为连接Spark集群的管道，因此`SparkContext`主要执行环境操作，比如: 累加器(accumulators)，`addFile`, `addJars`等。而`SparkSession`则用于读取和创建`Datasets/DataFrames`等操作。

### 创建一个`SparkSession`对象

我们可以使用下面的语句创建一个`SparkSession`对象

```python
>>> from pyspark.sql import SparkSession

>>> sparkSession = SparkSession.builder.appName('app name').\
                        master('local').getOrCreate()
```

其实当我们运行`pyspark`时，其实就已经自动创建了一个`SparkSession`并且分配给了`spark`变量。

在`pyspark`脚本里面，我们可以看到下面一句话。

```bash
export PYTHONSTARTUP="${SPARK_HOME}/python/pyspark/shell.py"
```
它设置了环境变量`PYTHONSTARTUP`, 那么在我们运行`pyspark`时，会先自动运行脚本

```bash
${SPARK_HOME}/python/pyspark/shell.py
```

我们再来看看这个脚本里有些什么，下图是脚本的部分截图:

![](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-11-spark_shell.png)

从红色框里的代码，我们可以看到创建了`SparkSession`对象并赋值给`spark`，`SparkSession`包含了`SparkContext`(`spark.sparkContext`), 并且使用`sc`指向`SparkContext`。

简要的说，我们一般采用如下规则:

* 创建一个`SparkSession`对象
* 使用`SparkSession`读取，创建SQL语句的视图，创建`Datasets/DataFrames`
* 从`SparkSession`得到`sparkContext`, 用于完成累加器(accumulators)，分发缓存文件和`RDDs`的相关操作。


### `sparkContext`元数据

`sparkContext`包含了一些实用的元数据信息。

获取spark版本信息

```python
>>> spark.version
u'2.2.1'

>>> sc.version
u'2.2.1'
```

获取部署的应用名

```python
>>> sc.appName
u'PySparkShell'
```

获取内存信息(**Python中暂时没有实现**)

```python
>>> sc.getExecutorMemoryStatus
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-4-755a4391f6a7> in <module>()
----> 1 sc.getExecutorMemoryStatus

AttributeError: 'SparkContext' object has no attribute 'getExecutorMemoryStatus'
```

但是Scala语言中有

```scala
scala> sc.getExecutorMemoryStatus
res0: scala.collection.Map[String,(Long, Long)] = Map(169.254.93.254:55267 -> (384093388,384093388))
```

`169.254.93.254:55267`代办主机信息。
`(384093388,384093388)`分别代表当前分配的最大内存和剩余内存

获取主机信息

```python
>>> sc.master
u'local[*]'
```

获取配置信息

```python
>>> sc.getConf().toDebugString().split('\n')
[u'spark.app.id=local-1515662568196',
 u'spark.app.name=PySparkShell',
 u'spark.driver.host=169.254.93.254',
 u'spark.driver.port=55105',
 u'spark.executor.id=driver',
 u'spark.master=local[*]',
 u'spark.rdd.compress=True',
 u'spark.serializer.objectStreamReset=100',
 u'spark.sql.catalogImplementation=hive',
 u'spark.submit.deployMode=client']
```
