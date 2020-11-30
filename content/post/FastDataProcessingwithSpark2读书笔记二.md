---
title: 《Fast Data Processing with Spark 2（Third Edition)》读书笔记二
date: 2018-01-15T17:45:35+08:00
tags:
    - spark
categories:
    - spark
slug: fast-data-processing-with-spark2-note-two
description: >
    本文主要介绍在Spark中加载和保存数据, 以及Spark的一些概念和Spark SQL查询
---

本书其它笔记[Fast Data Processing with Spark 2（Third Edition)》读书笔记目]({{< ref "FastDataProcessingwithSpark2读书笔记目录.md" >}})

## 在Spark中加载和保存数据

在我们开始操作数据之前，让我们先看一些Spark的概念以及了解一下不同的数据形态

### Spark抽象概念

Saprk的主要特点就是分布式的数据描述(representation)和计算，因此拥有大规模的数据操作。*Spark主要的数据描述单元就是`RDD`*(原句是: Spark's primary unit for representation of data is RDD)， 可以很方便的允许并行的数据计算。在Saprk 2.0.0版本之前，都是基于`RDDs`工作的。然而，它们都是低级别的原始结构，在执行和扩展上有很大的优化空间。因此才有了`Datasets/DataFrames`。`Datasets/DataFrames`是API级别的抽象，也是编程的主要接口，它提供了大量操作`RDD`, 但是通过优化查询计划在`RDDs`上封装了一层。因此，底层仍然是`RDD`, 只是通过`Datasets/DataFrames`的API来访问。

> RDDs can be viewed as arrays of arrays with primitive data types, such as integers, floats, and strings.
>
> Datasets/DataFrames, on the other hand, are similar to a table or a spreadsheet with column headings-such as name, title, order number, order date, and movie rating-and the associated data types.

`RDDs`可以看做是一系列原始数据数组的集合，比如: 整型，浮点型和字符串。`Datasets/DataFrames`在另一方面来说，有点类似一张表单或表格，有许多列标题（比如姓名，标题，订单号，订单日期，电影评分）以及关联的数据类型。

<!--more-->

无论在什么情况下。当使用`Datasets/DataFrames`,通过用`SparkSession`操作。然而，当需要进行一些低级别的操作来实复杂的计算或累加时，就是用`RDDs`,通过`SparkContext`来操作。

使用`dataset.rdd()`和`SparkSession.createDataset(rdd)/SparkSession.createDataFr ame(rdd)`方法，可以将`RDDs`转变为`Datasets/DataFrames`。


### RDDs

我们可以使用任何Hadoop支持的数据源来创建`RDDs`。Scala, Java, Python语言的原始数据结构都可以作为`RDD`的基础。使用原始的数据集合创建`RDDs`在测试时非常有用。

由于`RDD`是懒加载，它只有在它被需要时才会去计算。也就是说：当你尝试从`RDD`获取数据时，可能会失败。`RDD`里的数据只有在被缓存引用或输出时才会通过计算创建，这也意味着你可以构造大量的计算而不用担心阻塞计算线程。甚至，只有当你在具体化`RDD`（materialize the RDD）时，代码才会去加载原始数据。

**Tips**:

每次你具体化`RDD`时，它都会计算一次。因此，当频繁的使用时，可以借助缓存机制来提高效率。


### 数据形态

从数据形态的角度来说，所有的数据可以分成三类

* **结构化的数据（Structured data**），通常存储在数据库里，如Oracle, HBase, Cassandra等，关系型数据库是最常见的结构化数据的存储方式。结构化的数据样式，数据类型和数据大小都是已知的。

* **半结构化的数据（Semi-structured data）**，
正如它的名字一样，它也是有结构的数据，只是数据样式，数据类型和数据大小是可变。常见的半结构化数据格式有:`csv`, `json`和`parquet`

* **无结构的数据（Unstructured data）**，目前我们遇到的85%的数据都是这种格式。如图形，音频文件，社交文化等。大部分的数据处理，都是从无结构的数据开始，通过ETL(Extract-Transform-Load)，变型和其它技术，最终变成结构化的数据。


### 数据形态和`Datasets/DataFrames/RDDs`

现在让我们结合数据形态以及spark的抽象概念来看看如何使用spark读写数据。

在Spark 2.0.0之前，我们只需要使用`RDDs`和`map()`函数来按照需要改变数据。但是，当使用`Dataset/DataFrame`, 变得稍微复杂一些， 我们可以直接读入类似一张表的数据，包含数据类型和列信息，也可以更高效的工作。

通常来说:

1. 使用`SparkContext`和`RDDs`来处非无结构的数据

2. 使用`SparkSession`和` Datasets/DataFrames`来处理结构化和半结构化的数据。`SparkSession`有统一的标准处理各种格式的数据，如: `.csv`, `.json`, `.parquet`, `.jdbc`, `.orc`。还有一个可插拔的结构叫做`DataSource API`, 可以处理任意类型的结构化数据


### 加载数据到RDD

创建RDD最简单的方式就是使用编程语言(Scala, Python, Java)原始的数据结构，
`SparkContext`对象有个方法`parallelize`可以实现这个功能。

```python
>>> rdd = sc.parallelize([1,2,3])

>>> rdd.take(3)
[1, 2, 3]
```

加载外部数据最简单的方式就是读取一个文本文件。如果是单机模式的话，这个很简单，主要确保文件在本机即可。但是在集群环境下时需要注意，文件需要存在集群上的每个节点上，在分布式环境下，我们可以使用`addFile`方法，让Spark把文件拷贝到集群的每个机器上，如:

```python
>>> from pyspark.files import SparkFiles

>>> sc.addd("data/spam.data")

>>> sc.addFile("data/spam.data")

>>> in_file = sc.textFile(SparkFiles.get("spam.data"))

>>> in_file.take(1)
[u'0 0.64 0.64 0 0.32 0 0 0 0 0 0 0.64 0 0 0 0.32 0 1.29 1.93 0 0.96 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.778 0 0 3.756 61 278 1']
```

通常，我们输入的文件都是CSV或TSV文件。一般在创建RDDs时，都需要先解析数据。
通常有两种做法:

1. 自己写函数读取和解析CSV文件
2. 使用第三库，比如`opencsv`

我们看看自己写函数的方式

```
>>> !cat data/Line_of_numbers.csv
42,42,55,61,53,49,43,47,49,60,68,54,34,35,35,39
>>> inp_file = sc.textFile("data/Line_of_numbers.csv")

>>> numbers_rdd = inp_file.map(lambda line: line.split(','))

>>> numbers_rdd.take(10)
[[u'42',
  u'42',
  u'55',
  u'61',
  u'53',
  u'49',
  u'43',
  u'47',
  u'49',
  u'60',
  u'68',
  u'54',
  u'34',
  u'35',
  u'35',
  u'39']]
```

得到的数据都是字符串类型，我们希望把它转换为整型或浮点型

```
>>> numbers_rdd = inp_file.flatMap(lambda line:
...            line.split(',')).map(lambda x:float(x))
...

>>> numbers_rdd.take(10)
[42.0, 42.0, 55.0, 61.0, 53.0, 49.0, 43.0, 47.0, 49.0, 60.0]

>>> numbers_sum = numbers_rdd.sum()

>>> numbers_sum
766.0
```
注意上面解析文件时先使用的是`flatMap`，再使用的`map`。`flatMap`会将结果变成数组

```
>>> inp_file.map(lambda line: line.split(',')).take(10)
[[u'42',
  u'42',
  u'55',
  u'61',
  u'53',
  u'49',
  u'43',
  u'47',
  u'49',
  u'60',
  u'68',
  u'54',
  u'34',
  u'35',
  u'35',
  u'39']]

>>> inp_file.flatMap(lambda line: line.split(',')).take(10)
[u'42', u'42', u'55', u'61', u'53', u'49', u'43', u'47', u'49', u'60']
```

从Spark获取数据还可以使用`collect()`方法，用它可以获得原始的数据形式，跟`parallelize()`作用相反，`parallelize()`解析数据把它转换成RDD，`collect()`获取执行结果。

```python
>>> inp_file.flatMap(lambda line: line.split(',')).collect()
[u'42',
 u'42',
 u'55',
 u'61',
 u'53',
 u'49',
 u'43',
 u'47',
 u'49',
 u'60',
 u'68',
 u'54',
 u'34',
 u'35',
 u'35',
 u'39']
```


### 保存RDD数据

使用`saveAsTextFile`方法可以保存RDD数据

```python
>>> !cat data/Line_of_numbers.csv
42,42,55,61,53,49,43,47,49,60,68,54,34,35,35,39

>>> inp_file = sc.textFile("data/Line_of_numbers.csv")

>>> inp_file.saveAsTextFile("out.txt")

>>> !file out.txt/*
out.txt/_SUCCESS:   empty
out.txt/part-00000: ASCII text
out.txt/part-00001: empty
```
可以看到是以目录的形式保存的数据

读取保存的数据

```python
>>> read_file = sc.textFile("out.txt")

>>> read_file.take(10)
[u'42,42,55,61,53,49,43,47,49,60,68,54,34,35,35,39']
```

## Spark2.0 概念

下图是一个数据掮客的架构:

![Stack Architecture](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-14-StackArchitecture.png)

* Data Hub
* Analytics Hub
* Reporting Hub
* Visualization
* ETL


### Data Hub

数据中心 存储所有数据，数据来自于ETL服务，Kafaka等。数据可以是多种主题: 如市场数据，交易数据，文本日志，社交数据，非结构化的数据。也可以是一些跟时间序列相关的数据。


### Reporting Hub

报告中心 通常包含结构化和聚合之后的数据，用于日常报表和可视化面板。spark在这里主要做ETL和变形。数据可视化工具,比如:

* Tableau
* Quilk
* Pentaho

可以直接通过SparkSQL读取Spark里的数据。

### Analytics Hub

分析中心 是数据分析师花费大部分时间的地方。分析中心可以从数据中心获取大量数据，
然后生成intermediate Datasets，特征提取，model Datasets。

DataFrames, MLlib, GraphX, and ML pipelines 也在这里生成。



### Spark Full Stack


![Spark Full Stack](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-15-SparkFullStack.png)


### [大数据存储Parquet](https://parquet.apache.org/)


## Spark SQL

![Spark SQL Architeture](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-01-15-SparkSQLArchitecture.png)



下面的例子使用的数据文件下载地址:

<https://github.com/xsankar/fdps-v3>

### 单表查询

读取csv文件, 返回的`employees`是一个`pyspark.sql.dataframe.DataFrame`类型

```python
>>> print('Running Spark version: %s' % spark.version)
Running Spark version: 2.2.1

>>> employees = spark.read.option('header', 'true').csv('data/NW-Employees.csv')
```

统计总行数

```python
>>> print('Employees has %d rows' % employees.count())
Employees has 9 rows
```

打印前5行

```python
>>> employees.show(5)
+----------+---------+---------+--------------------+---------+--------+--------+-----+-------+-------+---------+
|EmployeeID| LastName|FirstName|               Title|BirthDate|HireDate|    City|State|    Zip|Country|ReportsTo|
+----------+---------+---------+--------------------+---------+--------+--------+-----+-------+-------+---------+
|         1|   Fuller|   Andrew|Sales Representative|  12/6/48| 4/29/92| Seattle|   WA|  98122|    USA|        2|
|         2|  Davolio|    Nancy|Vice President, S...|  2/17/52| 8/12/92|  Tacoma|   WA|  98401|    USA|        0|
|         3|Leverling|    Janet|Sales Representative|  8/28/63| 3/30/92|Kirkland|   WA|  98033|    USA|        2|
|         4|  Peacock| Margaret|Sales Representative|  9/17/37|  5/1/93| Redmond|   WA|  98052|    USA|        2|
|         5|Dodsworth|     Anne|       Sales Manager|   3/2/55|10/15/93|  London| null|SW1 8JR|     UK|        2|
+----------+---------+---------+--------------------+---------+--------+--------+-----+-------+-------+---------+
only showing top 5 rows
```

返回前3行

```python
>>> employees.head(3)
[Row(EmployeeID='1', LastName='Fuller', FirstName='Andrew', Title='Sales Representative', BirthDate='12/6/48', HireDate='4/29/92', City='Seattle', State='WA', Zip='98122', Country='USA', ReportsTo='2'),
 Row(EmployeeID='2', LastName='Davolio', FirstName='Nancy', Title='Vice President, Sales', BirthDate='2/17/52', HireDate='8/12/92', City='Tacoma', State='WA', Zip='98401', Country='USA', ReportsTo='0'),
 Row(EmployeeID='3', LastName='Leverling', FirstName='Janet', Title='Sales Representative', BirthDate='8/28/63', HireDate='3/30/92', City='Kirkland', State='WA', Zip='98033', Country='USA', ReportsTo='2')]
```

查看每一列的数据类型

```python
>>> for column_name, data_type in employees.dtypes:
...     print(f'{column_name}: {data_type}')
...
EmployeeID: string
LastName: string
FirstName: string
Title: string
BirthDate: string
HireDate: string
City: string
State: string
Zip: string
Country: string
ReportsTo: string
```

查看schema

```python
>>> employees.printSchema()
root
 |-- EmployeeID: string (nullable = true)
 |-- LastName: string (nullable = true)
 |-- FirstName: string (nullable = true)
 |-- Title: string (nullable = true)
 |-- BirthDate: string (nullable = true)
 |-- HireDate: string (nullable = true)
 |-- City: string (nullable = true)
 |-- State: string (nullable = true)
 |-- Zip: string (nullable = true)
 |-- Country: string (nullable = true)
 |-- ReportsTo: string (nullable = true)
```

默认读取的数据类型都是`String`,可以设置`inferSchema`为`true`来根据数据自动推测类型。(如下: `EmployeeID`和`EmployeeID`自动推测为int)

```python
>>> employees = spark.read.option('header', 'true').option('inferSchema', 'true').csv('data/NW-Employees.csv')

>>> for column_name, data_type in employees.dtypes:
...     print(f'{column_name}: {data_type}')
...
EmployeeID: int
LastName: string
FirstName: string
Title: string
BirthDate: string
HireDate: string
City: string
State: string
Zip: string
Country: string
ReportsTo: int

>>> employees.printSchema()
root
 |-- EmployeeID: integer (nullable = true)
 |-- LastName: string (nullable = true)
 |-- FirstName: string (nullable = true)
 |-- Title: string (nullable = true)
 |-- BirthDate: string (nullable = true)
 |-- HireDate: string (nullable = true)
 |-- City: string (nullable = true)
 |-- State: string (nullable = true)
 |-- Zip: string (nullable = true)
 |-- Country: string (nullable = true)
 |-- ReportsTo: integer (nullable = true)
```

创建全局临时视图

```python
>>> employees.createOrReplaceGlobalTempView("EmployeesTable")
```

查询视图(表名前面的`global_temp`不能省略，否则会报`Table or view not found`)

```python
>>> result = spark.sql('select * from global_temp.EmployeesTable')

>>> result.show(5)
+----------+---------+---------+--------------------+---------+--------+--------+-----+-------+-------+---------+
|EmployeeID| LastName|FirstName|               Title|BirthDate|HireDate|    City|State|    Zip|Country|ReportsTo|
+----------+---------+---------+--------------------+---------+--------+--------+-----+-------+-------+---------+
|         1|   Fuller|   Andrew|Sales Representative|  12/6/48| 4/29/92| Seattle|   WA|  98122|    USA|        2|
|         2|  Davolio|    Nancy|Vice President, S...|  2/17/52| 8/12/92|  Tacoma|   WA|  98401|    USA|        0|
|         3|Leverling|    Janet|Sales Representative|  8/28/63| 3/30/92|Kirkland|   WA|  98033|    USA|        2|
|         4|  Peacock| Margaret|Sales Representative|  9/17/37|  5/1/93| Redmond|   WA|  98052|    USA|        2|
|         5|Dodsworth|     Anne|       Sales Manager|   3/2/55|10/15/93|  London| null|SW1 8JR|     UK|        2|
+----------+---------+---------+--------------------+---------+--------+--------+-----+-------+-------+---------+
only showing top 5 rows
```

查看分析结果

```python
>>> employees.explain(True)
== Parsed Logical Plan ==
Relation[EmployeeID#12,LastName#13,FirstName#14,Title#15,BirthDate#16,HireDate#17,City#18,State#19,Zip#20,Country#21,ReportsTo#22] csv

== Analyzed Logical Plan ==
EmployeeID: string, LastName: string, FirstName: string, Title: string, BirthDate: string, HireDate: string, City: string, State: string, Zip: string, Country: string, ReportsTo: string
Relation[EmployeeID#12,LastName#13,FirstName#14,Title#15,BirthDate#16,HireDate#17,City#18,State#19,Zip#20,Country#21,ReportsTo#22] csv

== Optimized Logical Plan ==
Relation[EmployeeID#12,LastName#13,FirstName#14,Title#15,BirthDate#16,HireDate#17,City#18,State#19,Zip#20,Country#21,ReportsTo#22] csv

== Physical Plan ==
*FileScan csv [EmployeeID#12,LastName#13,FirstName#14,Title#15,BirthDate#16,HireDate#17,City#18,State#19,Zip#20,Country#21,ReportsTo#22] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/Users/linliang/Src/github/fdps-v3/data/NW-Employees.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<EmployeeID:string,LastName:string,FirstName:string,Title:string,BirthDate:string,HireDate:...
```

过滤查询

```ptython
>>> result = spark.sql('select * from global_temp.EmployeesTable where State="WA"')

>>> result.show(5)
+----------+---------+---------+--------------------+---------+--------+--------+-----+-----+-------+---------+
|EmployeeID| LastName|FirstName|               Title|BirthDate|HireDate|    City|State|  Zip|Country|ReportsTo|
+----------+---------+---------+--------------------+---------+--------+--------+-----+-----+-------+---------+
|         1|   Fuller|   Andrew|Sales Representative|  12/6/48| 4/29/92| Seattle|   WA|98122|    USA|        2|
|         2|  Davolio|    Nancy|Vice President, S...|  2/17/52| 8/12/92|  Tacoma|   WA|98401|    USA|        0|
|         3|Leverling|    Janet|Sales Representative|  8/28/63| 3/30/92|Kirkland|   WA|98033|    USA|        2|
|         4|  Peacock| Margaret|Sales Representative|  9/17/37|  5/1/93| Redmond|   WA|98052|    USA|        2|
|         8| Callahan|    Laura|Inside Sales Coor...|   1/7/58|  3/3/94| Seattle|   WA|98105|    USA|        2|
+----------+---------+---------+--------------------+---------+--------+--------+-----+-----+-------+---------+
```

综合上面的所有语句到单个文件

`single_table.py`

```python
# -*- coding: utf-8 -*-

from pyspark.sql import SparkSession
import os

spark = SparkSession.builder.appName('SQL Demo').getOrCreate()

print('Running Spark version: %s' % spark.version)

filePath = '.'

# 读取csv文件
employees = spark.read.option('header', 'true').csv(os.path.join(filePath, 'data/NW-Employees.csv'))

# 统计总行数
print('Employees has %d rows' % employees.count())
# 打印前5行
employees.show(5)
# 返回前3行
employees.head(3)


# 查看每一列的数据类型，默认读取的数据都是String
for column_name, data_type in employees.dtypes:
    print(f'{column_name}: {data_type}')

employees.printSchema()

# 创建全局的临时视图
employees.createOrReplaceGlobalTempView("EmployeesTable")
# 查询视图
result = spark.sql('select * from global_temp.EmployeesTable')
result.show(5)

# 查看分析结果
employees.explain(True)

# 添加过滤条件
result = spark.sql('select * from global_temp.EmployeesTable where State="WA"')
result.show(5)

spark.stop()
```

### 多表查询

```python
>>> orders = spark.read.option('header', 'true').option("inferSchema", "true").csv('data/NW-Orders.csv')

>>> order_details = spark.read.option('header', 'true').option("inferSchema", "true").csv('data/NW-Order-Details.csv')

>>> orders.createOrReplaceTempView("OrdersTable")

>>> order_details.createOrReplaceTempView("OrderDetailsTable")

>>> result = spark.sql('''
...    SELECT OrderDetailsTable.OrderID, ShipCountry,
...    UnitPrice, Qty, Discount FROM OrdersTable INNER JOIN OrderDetailsTable ON
...    OrdersTable.OrderID = OrderDetailsTable.OrderID
... ''')

>>> result.show(10)
+-------+-----------+---------+---+--------+
|OrderID|ShipCountry|UnitPrice|Qty|Discount|
+-------+-----------+---------+---+--------+
|  10248|     France|     34.8|  5|     0.0|
|  10248|     France|      9.8| 10|     0.0|
|  10248|     France|     14.0| 12|     0.0|
|  10249|    Germany|     42.4| 40|     0.0|
|  10249|    Germany|     18.6|  9|     0.0|
|  10250|     Brazil|     16.8| 15|    0.15|
|  10250|     Brazil|     42.4| 35|    0.15|
|  10250|     Brazil|      7.7| 10|     0.0|
|  10251|     France|     16.8| 20|     0.0|
|  10251|     France|     15.6| 15|    0.05|
+-------+-----------+---------+---+--------+
only showing top 10 rows


>>> result = spark.sql('''
...   SELECT ShipCountry, SUM(OrderDetailsTable.UnitPrice *
...    Qty * Discount) AS ProductSales FROM OrdersTable INNER JOIN
...    OrderDetailsTable ON OrdersTable.OrderID = OrderDetailsTable.OrderID GROUP
...    BY ShipCountry
... ''')

>>>

>>> result.show(10)
+-----------+------------------+
|ShipCountry|      ProductSales|
+-----------+------------------+
|     Sweden|5028.5599999999995|
|    Germany|        14355.9965|
|     France| 4140.437499999999|
|  Argentina|               0.0|
|    Belgium|1310.1250000000002|
|    Finland|          968.3975|
|      Italy|           934.995|
|     Norway|               0.0|
|      Spain|           1448.69|
|    Denmark|         2121.2275|
+-----------+------------------+
only showing top 10 rows

# 排序
>>> result.orderBy("ProductSales", ascending=False).show(10)
+-----------+------------------+
|ShipCountry|      ProductSales|
+-----------+------------------+
|        USA|17982.369499999997|
|    Germany|        14355.9965|
|    Austria|11492.791500000001|
|     Brazil|         8029.7585|
|    Ireland|          7337.485|
|     Canada|5137.8099999999995|
|     Sweden|5028.5599999999995|
|     France| 4140.437499999999|
|  Venezuela|          4004.261|
|    Denmark|         2121.2275|
+-----------+------------------+
only showing top 10 rows

```
