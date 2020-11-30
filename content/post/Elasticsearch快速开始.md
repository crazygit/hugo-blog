---
title: Elasticsearch快速入门
date: 2018-04-11T18:15:39+08:00
tags:
    - elasticsearch
categories:
    - Elasticsearch
slug: elasticsearch-getting-started
description: Elasticsearch快速入门, 主要介绍一些Elasticsearch的基本概念和操作
---


# [Elasticsearch快速入门](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)

当前使用版本: 6.2

## [基本概念](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html)

* Near Realtime (NRT)

    实时搜索，创建文档的索引后无需等待，立刻就可以开始搜索

* Cluster

    集群，一个集群可以包含多个节点。需要注意的是集群名字很重要，在同一个网络环境中，节点会自动加入相同集群名的集群。

    <!--more-->

* Node

    节点，每个节点都有自己的名字，它作为集群的一个单元，拥有数据存储和提供搜索能力等功能。
* Index

    索引(Index)是一系列文档(Document)的集合, 索引名全部必须小写，一个集群里可以有任意多个索引。类似关系型数据库中的DB。

* Type

    类似关系型数据库中的Table， 从Elasticsearch 6.0.0, 一个Index里只允许创建一个Type。

* Document

    文档，存储的最小单元,类似关系型数据库中的Row。

* Shards & Replicas

## 安装

本文主要介绍使用docker的安装的方式，主要有三种格式的镜像

```bash
# 基本版本(默认版本)，包含X-pack的基本特性和免费证书
$ docker pull docker.elastic.co/elasticsearch/elasticsearch:6.2.3
# platinum版本，包含X-pack的全部特性和30天试用证书
$ docker pull docker.elastic.co/elasticsearch/elasticsearch-platinum:6.2.3
# oss版本，不包含X-pack，只有开源的Elasticsearch
$ docker pull docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.3
```

选择使用oss版本

```bash
# 安装
$ docker pull docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.3

# 启动服务
$ docker run -p 9200:9200 -p 9300:9300 -e "cluster.name=mycluster" -e "node.name=my_node" docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.3

# 检查启动的服务
$ curl localhost:9200
{
  "name" : "my_node",
  "cluster_name" : "mycluster",
  "cluster_uuid" : "e4xwigyJQp-Er6ZJeIg1wg",
  "version" : {
    "number" : "6.2.3",
    "build_hash" : "c59ff00",
    "build_date" : "2018-03-13T10:06:29.741383Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 探索集群

集群健康状态检查

```bash
$ curl "localhost:9200/_cat/health?v"
epoch      timestamp cluster   status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1523427762 06:22:42  mycluster green           1         1      0   0    0    0        0             0                  -                100.0%
```

节点信息检查

```bash
$ curl "localhost:9200/_cat/nodes?v"
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.17.0.2           11          91   2    0.00    0.03     0.04 mdi       *      my_node
```

### 列出所有索引信息

```bash
$ curl "localhost:9200/_cat/indices?v"
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

### 创建索引

创建名为`customer`的索引

```bash
$ curl -X PUT "localhost:9200/customer?pretty"
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "customer"
}

$ curl "localhost:9200/_cat/indices?v"
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer ltwryTbDSUy2mKATPJ2adA   5   1          0            0      1.1kb          1.1kb
```

### 创建文档

在`customer`索引里，创建ID为1的文档。
注意: 加入索引不存在，elasticearch会自动创建索引。

```bash
$ curl -XPUT 'localhost:9200/customer/_doc/1?pretty' -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'

{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

# 查看创建的结果
$ curl 'localhost:9200/customer/_doc/1?pretty'
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```

### 删除索引

```bash
$ curl -X DELETE 'localhost:9200/customer?pretty'
{
  "acknowledged" : true
}
```

## 修改数据

```bash
# 创建一个文档
$ curl -XPUT 'localhost:9200/customer/_doc/1?pretty' -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'

{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

# 修改文档
$ curl -XPUT 'localhost:9200/customer/_doc/1?pretty' -H 'Content-Type: application/json' -d'
{
  "name": "xxxxx"
}
'

{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

## 通过POST方法，在创建文档时可以不用指定文档的id， elasticsearch会自动生成一个
$ curl -XPOST 'localhost:9200/customer/_doc?pretty' -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'

{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "Umx0s2IBV3QZ-G3Zdj0c",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

### 更新文档

```bash
$ curl -XPOST 'localhost:9200/customer/_doc/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe" }
}
'

{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

### 删除文档

```bash
$ curl -XDELETE 'localhost:9200/customer/_doc/2?pretty'

{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "result" : "not_found",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

### 批量处理

同时创建两个文档

```bash
$ curl -XPOST 'localhost:9200/customer/_doc/_bulk?pretty' -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
'

{
  "took" : 14,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 4,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "index" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```

更新和删除文档

```bash
$ curl -XPOST 'localhost:9200/customer/_doc/_bulk?pretty' -H 'Content-Type: application/json' -d'
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
'

{
  "took" : 20,
  "errors" : false,
  "items" : [
    {
      "update" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 5,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 4,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "delete" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 2,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}
```

当批量操作时，如果某个操作失败，后面的操作仍然会继续执行，并且根据执行顺序，依次返回每个操作的执行状态。

## 探索数据

在了解了基本操作之后，让我们来多添加一些数据，做一些数据分析的工作。

下载测试用的数据集

```bash
$ wget "https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true" -O accounts.json
```

导入数据

```bash
$ curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"

$ curl "localhost:9200/_cat/indices?v"
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  EnHJEWvLRv2tZLoI7Z_lXw   5   1       1000            0    474.7kb        474.7kb
```

### 查询数据

查询数据，主要有两种方式：

* [REST request URI](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-uri-request.html)
* [REST request body](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-request-body.html)

后者跟前者比，可以执行更复杂的操作，查询体也是json格式，便于阅读。前者主要是使用起来更加方便。

比如:

使用`request URI`的格式

```bash
$ curl "http://localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"
```

使用`request body`的格式

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
```

### 查询语法

Elasticsearch提供了一个类似json格式的查询语法，叫做[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl.html)。

限定返回查询结果的个数

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "size": 1
}
'
```

分页效果

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
'
```

排序

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
'
```

### 执行查询


默认情况下，查询返回的`_source`会包含所有的字段，我们也可以限制返回某些我们关心的字段

只返回`account_number`和`balance`

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
'
```
#### match_all

返回所有结果，没有查询条件限制

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
}
'
```
#### match

限定查询条件，返回`account_number`为20的结果

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "account_number": 20 } }
}
'
```

查询`address`里包含`mill`的结果（不区分大小写）

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill" } }
}
'
```

查询`address`里包含`mill`或`lane`的记录（不区分大小写)

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'
```

#### match_phrase

查询`address`里包含`mill`和`lane`的记录（不区分大小写)

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'
```

#### bool must

返回`address`包含`mill`和`lane`的结果，与上面的`match_phrase`效果一样

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

#### bool should

返回`address`包含`mill`或`lane`的结果，与`{"match": { "address": "mill lane"}`效果一样

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

#### bool must_not

返回`address`包即不包含`mill`，也不包含`lane`的结果

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

#### 组合

bool的多个条件还可以相互组合

返回`age=40`并且`state!=ID`的结果

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'
```

### 过滤(Filter)

过滤出`20000<balance<30000`的结果

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
```


### 聚合(Aggregations)

设置`size=0`是因为我们只关心聚合后的结果

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'
```

上面的语句等价于

```sql
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
```

聚合嵌套

```bash
$ curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```


## [Eleasticsearch安装及配置](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)详细介绍了各种安装Elasticsearch的方法，以及如何配置Elasticsearch。另外，
还介绍了将Elasticsearch迁移到生产环境时，需要关注的配置项等。


## docker-compose.yml

使用`docker compose`, 可以很方便的在本机创建一个关于elastic stack小集群， 对于开发和测试非常方便。
使用下面的`docker-compose.yml`文件，通过`docker compose`命令，可以在本地直接运行`elasticsearch`和`kibana`服务，`kibana`里开发工具的`Console`可以很方便的和`elasticsearch`交互。


```yaml
version: "3"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.3
    container_name: elasticsearch
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - http.host=0.0.0.0
      - transport.host=127.0.0.1
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es_data:/usr/share/elasticsearch/data
    networks:
      - esnet

  kibana:
    image: docker.elastic.co/kibana/kibana-oss:6.2.3
    container_name: kibana
    environment:
      SERVER_NAME: kibana-server
      ELASTICSEARCH_URL: http://elasticsearch:9200
    networks:
      - esnet
    depends_on:
      - elasticsearch
    ports:
      - "5601:5601"
volumes:
  es_data:
    driver: local

networks:
  esnet:
```
关于集成更多elastic stack服务，可以参考:
<https://github.com/elastic/stack-docker/blob/master/docker-compose.yml>

kibana Console运行效果如下：

![kibana console](https://cdn.jsdelivr.net/gh/crazygit/static@main/img/2018-04-12-kibana-devtool.png)
