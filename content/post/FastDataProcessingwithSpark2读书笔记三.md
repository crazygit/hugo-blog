---
title: 《Fast Data Processing with Spark 2（Third Edition)》读书笔记三
date: 2018-01-16T22:55:11+08:00
tags:
    - spark
categories:
    - spark
slug: fast-data-processing-with-spark2-note-three
description: >
    本文主要介绍DataSets/DataFrame的常用接口和函数

---

本书其它笔记[Fast Data Processing with Spark 2（Third Edition)》读书笔记目]({{< ref "FastDataProcessingwithSpark2读书笔记目录.md" >}})

## 数据分析的主力Datasets/DataFrames

### DataSets概述

Spark中，Dataset就是一组各式各样的列，类似一张excel表格或关系型数据库中的表。可以用于类型检查和语义化查询。

在R和Python语言中，使用的依然是DataFrame类，但是包含了所有的DataSet APIs。因此可以这样认为，DataSet在Python和R语言中就叫做DataFrame。

在Scala和Java语言中，使用的是DataSet接口，不存在DataFrame。

<!--more-->

### DataSet API概览

![DataSet APIs](http://images.wiseturtles.com/2018-01-16-DataSetAPIs.png)

下面的例子使用的数据文件下载地址:

<https://github.com/xsankar/fdps-v3>


### 常见的Dataset接口和函数


#### 读写操作

这个例子读取的csv文件内容如下:

```bash
$ cat data/spark-csv/cars.csv
year,make,model,comment,blank
2012,Tesla,S,No comment,
1997,Ford,E350,Go get one now they are going fast,
2015,Chevy,Volt,,
2016,Volvo,XC90,Good Car !,
```

读取和保存

```python
# -*- coding: utf-8 -*-

from pyspark.sql import SparkSession
import os

spark = SparkSession.builder.appName("Datasets APIs Demo")\
    .master("local")\
    .config("spark.logConf", "true")\
    .config("spark.logLevel", "ERROR")\
    .getOrCreate()
print(f"Running Spark Version {spark.version}")

filePath = "."
cars = spark.read.option("header", "true")\
        .option("inferSchema", "true")\
        .csv(os.path.join(filePath, "data/spark-csv/cars.csv"))

print(f"Cars has {cars.count()} rows")

cars.printSchema()


# 保存为csv格式
cars.write.mode("overwrite").option("header", "true").csv(
    os.path.join(filePath, "data/cars-out-csv.csv")
)

# 按年分区，保存为parquet格式
cars.write.mode("overwrite").partitionBy("year").parquet(
    os.path.join(filePath, "data/cars-out-pqt"))
spark.stop()

```

`mode`参数有:

* `overwrite`: 如果文件存在，则覆写已经存在的文件
* `append`: 追加写入到已经存在的文件
* `ignore`: 会忽略`Data Exists`, 并且不会保存数据，使用该模式需要特别注意
* `error`: 如果文件存在，抛出异常

查看保存结果

```bash
$ tree data/cars-out-csv.csv data/cars-out-pqt
data/cars-out-csv.csv
├── _SUCCESS
└── part-00000-aa27bc5f-aaeb-4f0c-804f-2d5320803d0d-c000.csv
data/cars-out-pqt
├── _SUCCESS
├── year=1997
│   └── part-00000-160cc90f-7c7a-4d88-b1c8-16c4fd8ec831.c000.snappy.parquet
├── year=2012
│   └── part-00000-160cc90f-7c7a-4d88-b1c8-16c4fd8ec831.c000.snappy.parquet
├── year=2015
│   └── part-00000-160cc90f-7c7a-4d88-b1c8-16c4fd8ec831.c000.snappy.parquet
└── year=2016
    └── part-00000-160cc90f-7c7a-4d88-b1c8-16c4fd8ec831.c000.snappy.parquet

4 directories, 7 files
```

从保存结果可以看出，保存为csv文件时，是一个目录，而不是单个的csv文件。

保存为`parquet`格式时，由于按照年来分区，因此生成了4个年份的子目录。这样做的好处是可以节约查询时间，比如当查询语句包含`year=2015`, 只会去查询`year=2015`这个目录的数据。在数据量大时，非常有助于提高查询效率。

读取保存的数据

```python
# -*- coding: utf-8 -*-

from pyspark.sql import SparkSession
import os

spark = SparkSession.builder.appName("Datasets APIs Demo")\
    .master("local")\
    .config("spark.logConf", "true")\
    .config("spark.logLevel", "ERROR")\
    .getOrCreate()
print(f"Running Spark Version {spark.version}")

filePath = "."
cars = spark.read.parquet(os.path.join(filePath, "data/cars-out-pqt"))

print(f"Cars has {cars.count()} rows")
cars.printSchema()

spark.stop()
```

#### 聚合函数

首页让我们加载测试数据

```python
>>> import os

>>> filePath = "."

>>> cars = spark.read.option('header', 'true')\
...         .option('inferSchema', 'true')\
...         .csv(os.path.join(filePath, 'data/car-data/car-mileage.csv'))

>>> print(f"Cars has {cars.count()} rows")
Cars has 32 rows

>>> cars.printSchema()
root
 |-- mpg: double (nullable = true)
 |-- displacement: double (nullable = true)
 |-- hp: integer (nullable = true)
 |-- torque: integer (nullable = true)
 |-- CRatio: double (nullable = true)
 |-- RARatio: double (nullable = true)
 |-- CarbBarrells: integer (nullable = true)
 |-- NoOfSpeed: integer (nullable = true)
 |-- length: double (nullable = true)
 |-- width: double (nullable = true)
 |-- weight: integer (nullable = true)
 |-- automatic: integer (nullable = true)

>>> cars.show(5)
+-----+------------+---+------+------+-------+------------+---------+------+-----+------+---------+
|  mpg|displacement| hp|torque|CRatio|RARatio|CarbBarrells|NoOfSpeed|length|width|weight|automatic|
+-----+------------+---+------+------+-------+------------+---------+------+-----+------+---------+
| 18.9|       350.0|165|   260|   8.0|   2.56|           4|        3| 200.3| 69.9|  3910|        1|
| 17.0|       350.0|170|   275|   8.5|   2.56|           4|        3| 199.6| 72.9|  3860|        1|
| 20.0|       250.0|105|   185|  8.25|   2.73|           1|        3| 196.7| 72.2|  3510|        1|
|18.25|       351.0|143|   255|   8.0|    3.0|           2|        3| 199.9| 74.0|  3890|        1|
|20.07|       225.0| 95|   170|   8.4|   2.76|           1|        3| 194.1| 71.8|  3365|        0|
+-----+------------+---+------+------+-------+------------+---------+------+-----+------+---------+
only showing top 5 rows
```

`descripe`统计列

```python
>>> cars.describe("mpg","hp","weight","automatic").show()
+-------+-----------------+-----------------+----------------+-------------------+
|summary|              mpg|               hp|          weight|          automatic|
+-------+-----------------+-----------------+----------------+-------------------+
|  count|               32|               32|              32|                 32|
|   mean|        20.223125|          136.875|       3586.6875|            0.71875|
| stddev|6.318289089312789|44.98082028541039|947.943187269323|0.45680340939917435|
|    min|             11.2|               70|            1905|                  0|
|    max|             36.5|              223|            5430|                  1|
+-------+-----------------+-----------------+----------------+-------------------+
```

分组统计

```python
>>> cars.groupBy("automatic").avg("mpg","torque").show()
+---------+------------------+-----------------+
|automatic|          avg(mpg)|      avg(torque)|
+---------+------------------+-----------------+
|        1|17.324782608695646|257.3636363636364|
|        0|27.630000000000006|          109.375|
+---------+------------------+-----------------+

# 不分组的情况
>>> cars.groupBy().avg("mpg","torque").show()
+---------+-----------+
| avg(mpg)|avg(torque)|
+---------+-----------+
|20.223125|      217.9|
+---------+-----------+
```

其它的聚合函数

```python
>>> from pyspark.sql.functions import avg, mean

>>> cars.agg(avg(cars["mpg"]), mean(cars["torque"]) ).show()
+---------+-----------+
| avg(mpg)|avg(torque)|
+---------+-----------+
|20.223125|      217.9|
+---------+-----------+
```


#### 统计函数

从前面的DataSets API概览图里可以看到，统计相关的函数都是在`sql.stat.*`下面。

下面看一个利用统计函数的例子

```python
>>> import os

>>> filePath = "."

>>> cars = spark.read.option('header', 'true')\
...         .option('inferSchema', 'true')\
...         .csv(os.path.join(filePath, 'data/car-data/car-mileage.csv'))

>>> cars.show(5)
+-----+------------+---+------+------+-------+------------+---------+------+-----+------+---------+
|  mpg|displacement| hp|torque|CRatio|RARatio|CarbBarrells|NoOfSpeed|length|width|weight|automatic|
+-----+------------+---+------+------+-------+------------+---------+------+-----+------+---------+
| 18.9|       350.0|165|   260|   8.0|   2.56|           4|        3| 200.3| 69.9|  3910|        1|
| 17.0|       350.0|170|   275|   8.5|   2.56|           4|        3| 199.6| 72.9|  3860|        1|
| 20.0|       250.0|105|   185|  8.25|   2.73|           1|        3| 196.7| 72.2|  3510|        1|
|18.25|       351.0|143|   255|   8.0|    3.0|           2|        3| 199.9| 74.0|  3890|        1|
|20.07|       225.0| 95|   170|   8.4|   2.76|           1|        3| 194.1| 71.8|  3365|        0|
+-----+------------+---+------+------+-------+------------+---------+------+-----+------+---------+
only showing top 5 rows

# 计算相关性
>>> cor = cars.stat.corr("hp", "weight")

>>> print("hp to weight: Correlation = %.4f" % cor)
hp to weight: Correlation = 0.8834

# 计算协方差
>>> cov = cars.stat.cov("hp", "weight")

>>> print("hp to weight: Covariance = %.4f" % cov)
hp to weight: Covariance = 37667.5403

# 交叉表显示
>>> cars.stat.crosstab("automatic", "NoOfSpeed").show()
+-------------------+---+---+---+
|automatic_NoOfSpeed|  3|  4|  5|
+-------------------+---+---+---+
|                  1| 23|  0|  0|
|                  0|  1|  5|  3|
+-------------------+---+---+---+
```

交叉表的功能是分组统计数据。上面的交叉表类似下面的效果

```python
>>> cars.createOrReplaceGlobalTempView("Cars")

>>> spark.sql("Select automatic, NoOfSpeed, count(*) from global_temp.Cars group by automatic, NoOfSpeed").show()
+---------+---------+--------+
|automatic|NoOfSpeed|count(1)|
+---------+---------+--------+
|        0|        5|       3|
|        1|        3|      23|
|        0|        3|       1|
|        0|        4|       5|
+---------+---------+--------+
```

交叉表在数据统计中非常有用，有利于我们观察各组数据直接的关系。下面是利用交叉表查看泰坦尼克号中幸存者和他的属性关系。


```python
>>> import os

>>> filePath = "."

>>> passengers = spark.read.option("header", "true")\
...     .option("inferSchema", "true")\
...     .csv(os.path.join(filePath, 'data/titanic3_02.csv'))

>>> result = passengers.select(passengers['Pclass'], passengers['Survived'], passengers['Gender'], passengers['Age'],
...                            passengers['SibSp'], passengers['Parch'],passengers['Fare'])
...

# Pclass: 票的等级
# SibSp: 兄弟姐妹/配偶在船上
# Parch: 父母/子女在船上
# Fare: 票价
>>> result.show(5)
+------+--------+------+------+-----+-----+--------+
|Pclass|Survived|Gender|   Age|SibSp|Parch|    Fare|
+------+--------+------+------+-----+-----+--------+
|     1|       1|female|  29.0|    0|    0|211.3375|
|     1|       1|  male|0.9167|    1|    2|  151.55|
|     1|       0|female|   2.0|    1|    2|  151.55|
|     1|       0|  male|  30.0|    1|    2|  151.55|
|     1|       0|female|  25.0|    1|    2|  151.55|
+------+--------+------+------+-----+-----+--------+
only showing top 5 rows

>>> result.printSchema()
root
 |-- Pclass: integer (nullable = true)
 |-- Survived: integer (nullable = true)
 |-- Gender: string (nullable = true)
 |-- Age: double (nullable = true)
 |-- SibSp: integer (nullable = true)
 |-- Parch: integer (nullable = true)
 |-- Fare: double (nullable = true)

>>> result.groupBy('Gender').count().show()
+------+-----+
|Gender|count|
+------+-----+
|female|  466|
|  male|  843|
+------+-----+

>>> result.stat.crosstab("Survived", "Gender").show()
+---------------+------+----+
|Survived_Gender|female|male|
+---------------+------+----+
|              1|   339| 161|
|              0|   127| 682|
+---------------+------+----+

>>> result.stat.crosstab("Survived","SibSp").show()
+--------------+---+---+---+---+---+---+---+
|Survived_SibSp|  0|  1|  2|  3|  4|  5|  8|
+--------------+---+---+---+---+---+---+---+
|             1|309|163| 19|  6|  3|  0|  0|
|             0|582|156| 23| 14| 19|  6|  9|
+--------------+---+---+---+---+---+---+---+

# 因为Age是浮点型，直接用Age创建的交叉表的结果非常长，不便于观察。所以将它转换为整型
>>> ageDist = result.select(result["Survived"], (result["age"] - result["age"] % 10).cast("int").name("AgeBracket"))

>>> ageDist.show(5)
+--------+----------+
|Survived|AgeBracket|
+--------+----------+
|       1|        20|
|       1|         0|
|       0|         0|
|       0|        30|
|       0|        20|
+--------+----------+
only showing top 5 rows

# 20s和30s之间的人最多，还有很多不知道年龄的人
>>> ageDist.crosstab("Survived", "AgeBracket").show()
+-------------------+---+---+---+---+---+---+---+---+---+----+
|Survived_AgeBracket|  0| 10| 20| 30| 40| 50| 60| 70| 80|null|
+-------------------+---+---+---+---+---+---+---+---+---+----+
|                  1| 50| 56|127| 98| 52| 32| 10|  1|  1|  73|
|                  0| 32| 87|217|134| 83| 38| 22|  6|  0| 190|
+-------------------+---+---+---+---+---+---+---+---+---+----+
```

#### 科学计算函数

科学统计函数基本都在`sql.functions`下面。

如: log(), log10(), sqrt(), cbrt(), exp(), pow(), sin(), cos(), tan(), acos(), asin(), atan(), toDegrees(), toRadians()


使用示例如下:

创建一个DataFrame

```python
>>> from pyspark.sql import Row

>>> row = Row("val")

>>> l = [1,2,3]

>>> rdd = sc.parallelize(l)

# 有两种方式将rdd转换为dataframe
# 方法一:
>>> df = spark.createDataFrame(rdd.map(row))

# 方法二
# >>> df = rdd.map(row).toDF()

>>> df.show()
+---+
|val|
+---+
|  1|
|  2|
|  3|
+---+
```

一些示例

```python
>>> from pyspark.sql.functions import log, log10, sart，log1p

>>> df.select(df['val'], log(df['val']).name('ln')).show()
+---+------------------+
|val|                ln|
+---+------------------+
|  1|               0.0|
|  2|0.6931471805599453|
|  3|1.0986122886681098|
+---+------------------+

>>> df.select(df['val'], log10(df['val']).name('log10')).show()
+---+-------------------+
|val|              log10|
+---+-------------------+
|  1|                0.0|
|  2| 0.3010299956639812|
|  3|0.47712125471966244|
+---+-------------------+

>>> df.select(df['val'], sqrt(df['val']).name('sqrt')).show()
+---+------------------+
|val|              sqrt|
+---+------------------+
|  1|               1.0|
|  2|1.4142135623730951|
|  3|1.7320508075688772|
+---+------------------+

>>> df.select(df['val'], log1p(df['val']).name('ln1p')).show()
+---+------------------+
|val|              ln1p|
+---+------------------+
|  1|0.6931471805599453|
|  2|1.0986122886681096|
|  3|1.3862943611198906|
+---+------------------+
```

对于给定的直角三角形的两个直角边，求其斜边的长度

```python
>>> import os

>>> filePath = '.'

>>> data = spark.read.option("header", "true")\
...                  .option("inferScheam", "true")\
...                  .csv(os.path.join(filePath, "data/hypot.csv"))

>>>

>>> data.show()
+---+---+
|  X|  Y|
+---+---+
|  3|  4|
|  5| 12|
|  7| 24|
|  9| 40|
| 11| 60|
| 13| 84|
+---+---+

>>> from pyspark.sql.functions import hypot

>>> data.select(data["X"], data["Y"],
...         hypot(data["X"], data["Y"]).name("hypot")).show()
+---+---+-----+
|  X|  Y|hypot|
+---+---+-----+
|  3|  4|  5.0|
|  5| 12| 13.0|
|  7| 24| 25.0|
|  9| 40| 41.0|
| 11| 60| 61.0|
| 13| 84| 85.0|
+---+---+-----+

```


#### 实战

下面让我们通过一个实战，使用前面学到的接口和函数。

我们使用Northwind Sales的数据集来分析下面的问题:

* How many orders were placed by each customer?
* How many orders were placed in each country?
* How many orders were placed for each month/year?
* What is the total number of sales for each customer, year-wise?
* What * is the average order by customer, year-wise?


读取数据

```python
>>> import os

>>> filePath = '.'

>>> orders = spark.read.option("header", "true")\
...                    .option("inferSchema", "true")\
...                    .csv(os.path.join(filePath, "data/NW/NW-Orders-01.csv")
... )

>>> print(f"Orders has {orders.count()} rows")
Orders has 830 rows

>>> orders.show(5)
+-------+----------+----------+-------------------+-----------+
|OrderID|CustomerID|EmployeeID|          OrderDate|ShipCountry|
+-------+----------+----------+-------------------+-----------+
|  10248|     VINET|         5|1996-07-02 00:00:00|     France|
|  10249|     TOMSP|         6|1996-07-03 00:00:00|    Germany|
|  10250|     HANAR|         4|1996-07-06 00:00:00|     Brazil|
|  10251|     VICTE|         3|1996-07-06 00:00:00|     France|
|  10252|     SUPRD|         4|1996-07-07 00:00:00|    Belgium|
+-------+----------+----------+-------------------+-----------+
only showing top 5 rows

>>> orders.printSchema()
root
 |-- OrderID: integer (nullable = true)
 |-- CustomerID: string (nullable = true)
 |-- EmployeeID: integer (nullable = true)
 |-- OrderDate: timestamp (nullable = true)
 |-- ShipCountry: string (nullable = true)


>>> orderDetails = spark.read.option("header", "true")\
...                    .option("inferSchema", "true")\
...                    .csv(os.path.join(filePath, "data/NW/NW-Order-Details.csv"))

>>> print(f"Order Details has {orderDetails.count()} rows")
Order Details has 2155 rows

>>> orderDetails.show(5)
+-------+---------+---------+---+--------+
|OrderID|ProductId|UnitPrice|Qty|Discount|
+-------+---------+---------+---+--------+
|  10248|       11|     14.0| 12|     0.0|
|  10248|       42|      9.8| 10|     0.0|
|  10248|       72|     34.8|  5|     0.0|
|  10249|       14|     18.6|  9|     0.0|
|  10249|       51|     42.4| 40|     0.0|
+-------+---------+---------+---+--------+
only showing top 5 rows


>>> orderDetails.printSchema()
root
 |-- OrderID: integer (nullable = true)
 |-- ProductId: integer (nullable = true)
 |-- UnitPrice: double (nullable = true)
 |-- Qty: integer (nullable = true)
 |-- Discount: double (nullable = true)

```

解答问题1: `How many orders were placed by each customer?`

```python
>>> orderByCustomer = orders.groupBy("CustomerID").count()

>>> from pyspark.sql.functions import desc

>>> orderByCustomer.sort(desc("count")).show(5)
+----------+-----+
|CustomerID|count|
+----------+-----+
|     SAVEA|   31|
|     ERNSH|   30|
|     QUICK|   28|
|     HUNGO|   19|
|     FOLKO|   19|
+----------+-----+
```

解答问题2: `How many orders were placed in each country?`

```python
>>> orderByCountry = orders.groupBy("ShipCountry").count()

>>> orderByCountry.sort(desc("count")).show(5)
+-----------+-----+
|ShipCountry|count|
+-----------+-----+
|    Germany|  122|
|        USA|  122|
|     Brazil|   82|
|     France|   77|
|         UK|   56|
+-----------+-----+
only showing top 5 rows
```

后面三个问题可以如下解答

```python
# -*- coding: utf-8 -*-

from pyspark.sql import SparkSession
from pyspark.sql.functions import to_date, month, year
import os

spark = SparkSession.builder.appName("Datasets APIs Demo") \
    .master("local") \
    .config("spark.logConf", "true") \
    .config("spark.logLevel", "ERROR") \
    .getOrCreate()
print(f"Running Spark Version {spark.version}")

filePath = "."
orders = spark.read.option("header", "true") \
    .option("inferSchema", "true") \
    .csv(os.path.join(filePath, "data/NW/NW-Orders-01.csv"))

orderDetails = spark.read.option("header", "true") \
    .option("inferSchema", "true") \
    .csv(os.path.join(filePath, "data/NW/NW-Order-Details.csv"))

result = orderDetails.select(orderDetails['OrderID'],
                             ((orderDetails['UnitPrice'] * orderDetails['Qty'] - (
                                 orderDetails['UnitPrice'] * orderDetails['Qty'] * orderDetails['Discount'])).
                              name('OrderPrice')))

result.show(5)

# 设置DataFrame orderTot的别名为OrderTotal
orderTot = result.groupBy('OrderID').sum('OrderPrice').alias('OrderTotal')
orderTot.sort("OrderID").show(5)

orders_df = orders.join(orderTot, orders["OrderID"] == orderTot["OrderID"], "inner") \
    .select(orders["OrderID"],
            orders["CustomerID"],
            orders["OrderDate"],
            orders["ShipCountry"].alias("ShipCountry"),
            orderTot["sum(OrderPrice)"].alias("Total"))

orders_df.sort("CustomerID").show()

# 过滤出空行
orders_df.filter(orders_df["Total"].isNull()).show()

# 添加Date列
orders_df2 = orders_df.withColumn('Date', to_date(orders_df['OrderDate']))
orders_df2.show(5)
orders_df2.printSchema()

# 添加Month和Year列
orders_df3 = orders_df2.withColumn("Month", month(orders_df2["OrderDate"])) \
    .withColumn("Year", year(orders_df2["OrderDate"]))
orders_df3.show(5)
orders_df3.printSchema()

# 问题3
ordersByYM = orders_df3.groupBy("Year", "Month").sum("Total").alias("Total")
ordersByYM.sort(ordersByYM["Year"], ordersByYM["Month"]).show()

# 问题4
ordersByCY = orders_df3.groupBy("CustomerID", "Year").sum("Total").alias("Total")
ordersByCY.sort(ordersByCY["CustomerID"], ordersByCY["Year"]).show()

# 问题5
ordersCA = orders_df3.groupBy("CustomerID").avg("Total").alias("Total")
ordersCA.sort("avg(Total)", ascending=False).show()

spark.stop()
```