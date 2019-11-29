---
title: fluentd教程(含实例)
date: 2019-11-29T15:29:29+08:00
tags:
    - fluentd, docker, elasticsearch, kamba, log
categories:
    - DevOps
slug: fluentd-tutorial
description: >
    fluentd教程，含使用实例，开箱即用
---


fluentd是一个开源的日志收集系统，能够收集各式各样的日志, 并将日志转换成方便机器处理的json格式。

![fluentd日志架构](http://images.wiseturtles.com/1574818088.png)

<!--more-->

## 安装

不同操作系统的安装方式不同，具体可以参考:

[官方文档: Installation](https://docs.fluentd.org/installation)

另外在生产环境中安装Fluentd之前，也需要对操作系统做一些配置，如:

* 设置好NTP时间同步
* 调整允许操作的文件符最大个数
* 优化内核中与网络相关的参数等

具体配置可以参考:

[官方文档: Before Installation](https://docs.fluentd.org/installation/before-install)


本文为了便于快速测试，直接使用fluentd的docker镜像来启动fluentd服务。

```bash
# 下载fluentd镜像
$ docker pull fluent/fluentd:v1.7-1
```

### 准备配置文件

首先创建一些简单的文件方便测试。本文所有使用到的配置文件都已经上传到github,可以直接下载使用

```bash
$ git clone https://github.com/crazygit/fluentd_demo
```

当然也可以自己动手完成

```bash
$ mkdir fluentd_demo

# 注意: 本文后续所有命令都在fluentd_demo目录下执行
$ cd fluentd_demo

# 创建用于保存fluentd的配置文件的etc目录和保存日志文件的log目录
$ mkdir -p etc log

# 再创建一个简单的配置文件
$ cat etc/fluentd_basic_setup.conf

```
```bash
<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>
<match test.cycle>
  @type stdout
</match>
```

配置文件解释:

* `source`部分使用了`http`输入插件，在`8888`端口启动一个web服务用于收集日志

* `match`部分定义只要日志匹配`test.cycle`标签，就将日志输出到标准输出

目前不用太关心配置文件的格式，后面会有详细的介绍。

创建好的目录结构如下:

```
$ tree
.
├── etc
│   └── fluentd_basic_setup.conf
└── log
```

### 启动容器

```bash
$ docker run -p 8888:8888 --rm -v $(pwd)/etc:/fluentd/etc -v $(pwd)/log:/fluentd/log fluent/fluentd:v1.7-1 -c /fluentd/etc/fluentd_basic_setup.conf -v
```

命令参数解释如下:

* `-p`参数映射容器的`8888`端口到宿主机的`8888`端口，方便我们直接从宿主机直接访问容器
* 第一个`-v`参数用于挂载本地的`etc`目录到容器内的`etc`目录，让容器能够使用本地的配置文件
* 第二个`-v`参数用于挂载本地的`log`目录到容器内的`log`目录，便于保存输出的日志文件
* `-c`参数用于设置容器内的`fluentd`时候启动的使用`/fluentd/etc/fluentd_basic_setup.conf`配置文件
* 最后一个参数`-v`用于设置容器内的`fluentd`时候启动的开启`verbose`模式，便于调试发现问题。

正常启动后，能看到如下输出

```
fluentd -c /fluentd/etc/fluentd_basic_setup.conf -v
2019-11-27 02:03:22 +0000 [info]: fluent/log.rb:322:info: parsing config file is succeeded path="/fluentd/etc/fluentd_basic_setup.conf"
2019-11-27 02:03:22 +0000 [info]: fluent/log.rb:322:info: using configuration file: <ROOT>
  <source>
    @type http
    port 8888
    bind "0.0.0.0"
  </source>
  <match test.cycle>
    @type stdout
  </match>
</ROOT>
2019-11-27 02:03:22 +0000 [info]: fluent/log.rb:322:info: starting fluentd-1.7.4 pid=6 ruby="2.5.7"
2019-11-27 02:03:22 +0000 [info]: fluent/log.rb:322:info: spawn command to main:  cmdline=["/usr/bin/ruby", "-Eascii-8bit:ascii-8bit", "/usr/bin/fluentd", "-c", "/fluentd/etc/fluentd_basic_setup.conf", "-v", "-p", "/fluentd/plugins", "--under-supervisor"]
2019-11-27 02:03:23 +0000 [info]: fluent/log.rb:322:info: gem 'fluentd' version '1.7.4'
2019-11-27 02:03:23 +0000 [info]: fluent/log.rb:322:info: adding match pattern="test.cycle" type="stdout"
2019-11-27 02:03:23 +0000 [info]: fluent/log.rb:322:info: adding source type="http"
2019-11-27 02:03:23 +0000 [info]: #0 fluent/log.rb:322:info: starting fluentd worker pid=14 ppid=6 worker=0
2019-11-27 02:03:23 +0000 [debug]: #0 fluent/log.rb:302:debug: listening http bind="0.0.0.0" port=8888
2019-11-27 02:03:23 +0000 [info]: #0 fluent/log.rb:322:info: fluentd worker is now running worker=0
```

另外，也可以通过环境变量`FLUENTD_CONF`设置需要使用的配置文件

```bash
# 下面的命令和上面的是等效的
$ docker run -p 8888:8888 --rm -v $(pwd)/etc:/fluentd/etc/ -e FLUENTD_CONF=fluentd_basic_setup.conf fluent/fluentd:v1.7-1
```

### 测试

```bash
# 任意伪造一个用户登录的日志，向fluentd服务提交日志
$ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: Keep-Alive
Content-Length: 0
```

在`fluentd`的容器里能看到如下输出

```
2019-11-27 02:05:16.583154500 +0000 test.cycle: {"action":"login","user":2}
```

## Fluentd事件的生命周期

### 什么是事件(Event)

正如开篇提到过，Fluentd是一个日志收集系统，那么一条日志消息，在Fluentd里就认为是一个事件(`Event`)。

### 事件结构

Fluentd的事件由下面三部分组成


* 标签(`tag`): 用于说明这个事件是哪里产生的，可用于后面的事件路由
* 时间(`time`): 事件是什么时候发生的，时间格式为: [Epoch time](https://en.wikipedia.org/wiki/Unix_time), 即Unix时间戳
* 记录(`record`): 事件内容本身，JSON格式

所有的输入插件都需要解析原始日志，生成满足上面结构的事件字段，比如，一条Apache的访问日志:

```
192.168.0.1 - - [28/Feb/2013:12:00:00 +0900] "GET / HTTP/1.1" 200 777
```
通过`in_tail`输入插件处理之后，将会得到下面的输出

```bash
tag: apache.access # 由配置文件指定
time: 1362020400   # 28/Feb/2013:12:00:00 +0900
record: {"user":"-","method":"GET","code":200,"size":777,"host":"192.168.0.1","path":"/"}
```

### 事件的处理流程

当fluentd收到一个事件之后，会经过一系列的处理流程:

1. 如修改事件的相关字段
2. 过滤掉一些不关心的事件
3. 路由事件输出到不同的地方

下面将一一介绍介绍事件的处理流程

#### 过滤器(Filter）

Filter用于定义一个事件是该被接受或者是被过滤掉(抛弃掉)。使用示例如下:

```bash
$ cat etc/fluentd_filter_demo.conf
```

```bash
<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>

<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^logout$
  </exclude>
</filter>

<match test.cycle>
  @type stdout
</match>
```
上面的示例配置了让我们直接过滤掉用户`logout`的事件。

重启fluentd,使用上面的定义的配置文件

```bash
$ docker run -p 8888:8888 --rm -v $(pwd)/etc:/fluentd/etc -v $(pwd)/log:/fluentd/log fluent/fluentd:v1.7-1 -c /fluentd/etc/fluentd_filter_demo.conf -v
```

测试
```bash
$ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-type: text/plain
Connection: Keep-Alive
Content-length: 0

$ curl -i -X POST -d 'json={"action":"logout","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-type: text/plain
Connection: Keep-Alive
Content-length: 0
```

我们向fluentd发送了两个事件，分别是用户`login`和`logout`的事件。

检查`fluend`的输出

```
2019-11-27 03:38:28.973757600 +0000 test.cycle: {"action":"login","user":2}
```

可以看到只输出了用户`login`的事件，`logout`事件被过滤掉了。


#### 标识符(Labels)

从前面的例子里，我们可以看到，`fluentd`的处理流程是根据我们在配置文件中的定义，从上到下依次执行的。假如我们在配置文件里定义了比较多输入源，不同的输入源需要使用不同的filters时，如果仍然按照从上到下执行的顺序的话，由于不同的处理需求，我们的配置文件可能变得非常复杂。

通过`label`，我们可以为不同的输入源指定不同的处理流程，示例如下:

```bash
$ cat etc/fluentd_labels.conf
```
```bash
<source>
  @type http
  bind 0.0.0.0
  port 8888
  @label @STAGING  # 注意这里我们添加了label
</source>

<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^login$
  </exclude>
</filter>

<label @STAGING>
  <filter test.cycle>
    @type grep
    <exclude>
      key action
      pattern ^logout$
    </exclude>
  </filter>

  <match test.cycle>
    @type stdout
  </match>
</label>
```

上面的示例文件里，我们首先定义了一个filter过滤掉`login`事件

```
<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^login$
  </exclude>
</filter>
```
接着又在`label`块里面过滤掉了`logout`事件

```
<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^login$
  </exclude>
</filter>
```

如果按照从上到下的顺序执行，那么我们将看不到任何`login`和`logout`的事件。但是实际结果如何呢？让我们来测试一下。

使用上面定义的配置文件启动fluentd

```bash
$ docker run -p 8888:8888 --rm -v $(pwd)/etc:/fluentd/etc -v $(pwd)/log:/fluentd/log fluent/fluentd:v1.7-1 -c /fluentd/etc/fluentd_labels.conf -v
```

提交事件来测试, 同样向fluentd发送两个事件，分别是用户`login`和`logout`的事件。

```bash
$ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: Keep-Alive
Content-Length: 0

$ curl -i -X POST -d 'json={"action":"logout","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: Keep-Alive
Content-Length: 0
```

查看fluentd输出

```
2019-11-27 03:51:45.088464900 +0000 test.cycle: {"action":"login","user":2}
```

可以看到，只有`logout`事件被过滤了，原因是我们为输入设置了label

```
<source>
  @type http
  bind 0.0.0.0
  port 8888
  @label @STAGING  # 注意这里我们添加了label
</source>
```

因此跳过中间设置的一些filter，只运行了`<label @STAGING>...</lable>`标签块里的`filter`

#### 缓冲区(Buffers)

在前面的例子中，我们使用的都是`stdout`这样一个没有缓冲区的输出，在生产环境中，我们用到的输出基本都是有缓冲区
的,比如`s3`, `forward`,`mongodb`等，这些输出插件在收到事件之后，会将事件先保存到缓冲区，然后等满足特定条件之后，再将事件输出到目标输出。


## 配置文件语法

配置文件由一下几部分组成

### 配置文件中的术语

* `source`: 指定输入源

  例如:
  ```bash
  # 在24224端口接受TCP事件
  <source>
    @type forward
    port 24224
  </source>

  <source>
    @type http
    port 9880
  </source>
  ```

  输入源可以一次指定多个， `@type`参数指定使用哪一个输入插件。
  fluentd支持各种输入插件, 比如:

  * in_tail
  * in_forward
  * in_udp
  * in_tcp
  * in_unix
  * in_http
  * in_syslog
  * in_exec
  * in_dummy
  * in_windows_eventlog

  插件的具体使用可以参考文档:
  <https://docs.fluentd.org/input>

* `match`: 指定输出的目的地

  例如:
  ```bash
  # 将满足myapp.acccess标签的事件全部输出到
  # /var/log/fluent/access.%Y-%m-%d
  <match myapp.access>
    @type file
    path /var/log/fluent/access
  </match>
  ```

  输出也可以一次指定多个， `@type`参数指定使用哪一个输出插件。
  fluentd支持各种输出插件, 比如:

  * out_copy
  * out_null
  * out_roundrobin
  * out_stdout
  * out_exec_filter
  * out_forward
  * out_mongo or out_mongo_replset
  * out_exec
  * out_file
  * out_s3
  * out_webhdfs

  插件的具体使用可以参考文档:
  <https://docs.fluentd.org/output>

* `filter`: 指定事件的处理流程

  可以多个filter串联起来，比如:

  ```
  Input -> filter 1 -> ... -> filter N -> Output
  ```

  例如:
  ```
  <filter myapp.access>
  @type record_transformer
  <record>
    host_param "#{Socket.gethostname}"
  </record>
  </filter>
  ```

  上面的`filter`会添加`host_param`参数到事件

  fluentd也内置了各种`filter`, 比如:

  * grep
  * record-transformer
  * filter_stdout
  * geoip
  * parser

  filter的具体使用可以参考文档:

  <https://docs.fluentd.org/filter>

* `system`: 指定系统级别的配置

  可以设置的参数有

  * log_level
  * suppress_repeated_stacktrace
  * emit_error_log_interval
  * suppress_config_dump
  * without_source
  * process_name (only available in system directive. No fluentd option)

  例如:

  ```bash
  <system>
  # equal to -qq option
  log_level error
  # equal to --without-source option
  without_source
  # ...
  </system>
  ```

* `label`: 用于分组特定的filter和match

  例如:

  ```bash
  <source>
  @type forward
  </source>

  <source>
    @type tail
    @label @SYSTEM
  </source>

  <filter access.**>
    @type record_transformer
    <record>
      # ...
    </record>
  </filter>
  <match **>
    @type elasticsearch
    # ...
  </match>

  <label @SYSTEM>
    <filter var.log.middleware.**>
      @type grep
      # ...
    </filter>
    <match **>
      @type s3
      # ...
    </match>
  </label>
  ```

  上面的配置文件中:

  `in_forward`的事件将经过`record_transformer` 过滤器和`elasticsearch`输出。

  `in_tail`输入的事件将经过`grep`过滤器和`s3`输出。


  另外: `<label @ERROR>`属于内置的配置，用于保存内部错误，比如:
  缓冲区已经满了或者无效的事件等。

* `@include`: 引入其它的配置文件。可以将配置文件拆分为多个，便于复用。当要使用的时候，直接使用`@include`引入即可，例如：

  ```bash
  # 通过绝对路径引入
  @include /path/to/config.conf

  # 通过相对路径引入，相对于当前配置文件的路径
  @include extra.conf

  # 模糊匹配，所有符合条件的会根据文件名的字母顺序依次导入
  # 比如: a.conf, b.conf, ..., z.conf
  # 因此, 要注意各个配置文件不应该有顺#序依赖，如果有顺序依赖，请明确指出导入的文件名
  @include config.d/*.conf

  # 使用在线的配置
  @include http://example.com/fluent.conf
  ```

  `@include`指定也可以用于导入相关的参数信息，比如:

  有如下配置文件
  ```bash
  <match pattern>
    @type forward
    # other parameters...
    <buffer>
      @type file
      path /path/to/buffer/forward
      @include /path/to/out_buf_params.conf
    </buffer>
  </match>

  <match pattern>
    @type elasticsearch
    # other parameters...
    <buffer>
      @type file
      path /path/to/buffer/es
      @include /path/to/out_buf_params.conf
    </buffer>
  </match>
  ```

  参数配置文件`/path/to/out_buf_params.conf`

  ```
  flush_interval 5s
  total_limit_size 100m
  chunk_limit_size 1m
  ```

### 配置文件的模式匹配(patterns)


#### 通配符

如前面的示例可以看到，fluented主要根据事件的tag来分区不同的处理流程

虽然我们可以明确指定需要处理的tag，比如:`<filter app.log>`来指定只处理tag为`app.log`的事件。我们也可以在`filter`和`match`中通过通配符，来处理同一类tag的事件

tag通常是一个字符串，由`.`分隔，比如`myapp.access`

* `*`: 匹配满足一个tag部分的事件, 比如: `a.*`, 它将匹配`a.b`这样的tag, 但是不会处理`a`或者`a.b.c`这类tag
* `**`: 匹配满足0个或多个tag部分，比如: `a.**`, 它将匹配`a`, `a.b`, `a.b.c`这三种tag
* `{X, Y, Z}`: 匹配满足`X`,`Y`或者`Z`的tag, 比如: `{a, b}`将匹配`a`或者`b`,但是不会匹配`c`。

  这种格式也可以和通配符组合使用,比如`a.{b.c}.*`或`a.{b.c}.**`

* `#{...}` 会把花括号内的字符串当做是`ruby`的表达式处理。比如

  ```
  <match "app.#{ENV['FLUENTD_TAG']}">
    @type stdout
  </match>
  ```

  如果设置了环境变量`FLUENTD_TAG`为`dev`,那上面等价于`app.dev`

* 当指定了多个模式时（使用一个或多个空格分开）,只要满足其中任意一个就行。

  比如:
  `<match a b>`匹配`a`和`b`
  `<match a.** b.*>`匹配`a`, `a.b`, `a.b.c`, `b.d`等


#### 多个match之间的顺序注意

当有多个match, 需要注意一下它们的顺序， 如下面的例子，第二个match永远也不会生效

```
# ** matches all tags. Bad :(
<match **>
  @type blackhole_plugin
</match>

<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```

正确的写发应该是将确定的tag尽量写在前面，模糊匹配的写在后面。

```bash
<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>

# Capture all unmatched tags. Good :)
<match **>
  @type blackhole_plugin
</match>
```

如果需要将输出到多个match,需要使用`out_copy`插件。

另外需要注意顺序的是`filter`和`match`, 如果将`filter`放在`match`之后，那么它也永远不会生效，正确的用法如下：

```bash
# You should NOT put this <filter> block after the <match> block below.
# If you do, Fluentd will just emit events without applying the filter.
<filter myapp.access>
  @type record_transformer
  ...
</filter>

<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```

### 常见的配置文件示例

下面给出了一些常见的使用场景的配置文件写法


#### 简单的输入，过滤，输出

```bash
<source>
  @type forward
</source>

<filter app.**>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match app.**>
  @type file
  # ...
</match>
```

#### 多个输入

```bash
<source>
  @type forward
</source>

<source>
  @type tail
  tag system.logs
  # ...
</source>

<filter app.**>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match {app.**,system.logs}>
  @type file
  # ...
</match>
```

#### 使用Label的输出

```bash
<source>
@type forward
</source>

<source>
  @type dstat
  @label @METRICS # dstat events are routed to <label @METRICS>
  # ...
</source>

<filter app.**>
  @type record_transformer
  <record>
    # ...
  </record>
</filter>

<match app.**>
  @type file
  # ...
</match>

<label @METRICS>
  <match **>
    @type elasticsearch
    # ...
  </match>
</label>
```

#### 重新设置标签并重新路由([fluent-plugin-route](https://github.com/tagomoris/fluent-plugin-route )插件的使用)

```bash
<match worker.**>
  @type route
  remove_tag_prefix worker
  add_tag_prefix metrics.event

  <route **>
    copy # For fall-through. Without copy, routing is stopped here.
  </route>
  <route **>
    copy
    @label @BACKUP
  </route>
</match>

<match metrics.event.**>
  @type stdout
</match>

<label @BACKUP>
  <match metrics.event.**>
    @type file
    path /var/log/fluent/bakcup
  </match>
</label>
```

#### 根据事件内容重新路由([fluent-plugin-rewrite-tag-filter](https://github.com/fluent/fluent-plugin-rewrite-tag-filter)插件的使用)

```bash
<source>
  @type forward
</source>

# event example: app.logs {"message":"[info]: ..."}
<match app.**>
  @type rewrite_tag_filter
  <rule>
    key message
    pattern ^\[(\w+)\]
    tag $1.${tag}
  </rule>
  # you can put more <rule>
</match>

# send mail when receives alert level logs
<match alert.app.**>
  @type mail
  # ...
</match>

# other logs are stored into file
<match *.app.**>
  @type file
  # ...
</match>
```

#### 重新路由事件到其它的Label([out_relabel](https://docs.fluentd.org/output/relabel)插件的使用)

```bash
<source>
  @type forward
</source>

<match app.**>
  @type copy
  <store>
    @type forward
    # ...
  </store>
  <store>
    @type relabel
    @label @NOTIFICATION
  </store>
</match>

<label @NOTIFICATION>
  <filter app.**>
    @type grep
    regexp1 message ERROR
  </filter>

  <match app.**>
    @type mail
  </match>
</label>
```

### 配置文件中的参数类型

在配置文件中使用的插件，大部分都可以接受1个或多个参数，也可以指定参数类型

* `string`: 字符串类型
* `integer`: 整型
* `float`: 浮点型
* `size`: 字节(bytes)类型.为了便于阅读，它的值有几种常见的写法

  * `<INTEGER>k`或`<INTEGER>K` 表示使用单位`KB`
  * `<INTEGER>m`或`<INTEGER>M` 表示使用单位`M`
  * `<INTEGER>g`或`<INTEGER>G` 表示使用单位`G`
  * `<INTEGER>t`或`<INTEGER>T` 表示使用单位`T`
  * 纯数字时表示使用默认单位为`B`

* `time`: 时间类型，默认单位为秒， 同样为了便于阅读，它的值也有几种常见的写法

  * `<INTEGER>s`, 表示使用单位`秒`
  * `<INTEGER>m`, 表示使用单位`分钟`
  * `<INTEGER>h`, 表示使用单位`小时`
  * `<INTEGER>d`, 表示使用单位`天`
  * 纯数字时表示使用默认单位为`秒`， `0.1`表示`100ms`

* `array`: 数组类型。有两种写法:

  * 完整格式的写法: `["key1", "key2"]`
  * 简写: `key1,key2`

* `hash`: 字典类型。也有两种写法:

  * 完整格式的写法: `{"key1":"value1", "key2":"value2"}`
  * 简写: `key1:value1,key2:value2`

### 常见的参数

以`@`开始的参数表示fluented的保留参数

* `@type`: 指定使用的参加类型
* `@id`: 指定插件的ID
* `@label`: 指定事件的标识符
* `@log_level`: 指定类型


### 模块(section)支持

下面的模块并不是所有的插件都支持，具体使用请结合使用的插件查看

* `parse`: 指明如何解析原始内容，如解析nginx, apache日志等
* `buffer`: 配置如何缓冲输出
* `format`: 配置如何格式化事件
* `extract`: 从事件中提取值
* `inject`: 向事件注入一些属性
* `transport`: 用于指定某些插件的输入输出时`server`的连接信息
* `storage`: 指定如何保存插件本身的状态

每个模块都有对应的插件，使用详情可以查看官方文档


### 检查配置文件的格式

可以使用下面的命令检查配置文件的格式是否正确

```bash
$ docker run --rm -v $(pwd)/etc:/fluentd/etc/ fluent/fluentd:v1.7-1 --dry-run -c /fluentd/etc/fluentd_basic_setup.conf
```

## 使用Fluentd收集docker容器的日志

### docker容器日志格式

在使用fluentd收集docker日志时，默认会将日志分成4个部分:

分别是:
* `container_id`: 容器的ID
* `container_name`:容器的名字
* `source`: 日志的类型，`stdout`或`stderr`
* `log`: 日志本身

### 收集Docker容器的日志示例

1. 首先创建一个配置文件`etc/fluentd_docker.conf`

```bash
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<match **>
    @type stdout
</match>
```

```bash
# 启动一个fluentd服务
$ docker run -it -p 24224:24224 -v $(pwd)/etc:/fluentd/etc/ -e FLUENTD_CONF=fluentd_docker.conf fluent/fluentd:v1.7-1

# 启动一个docker容器并使用fluentd收集日志
$ docker run --log-driver=fluentd -p 5000:80 httpd

# 访问服务产生日志
$ curl http://127.0.0.1:5000
<html><body><h1>It works!</h1></body></html>
```

在fluentd的容器日志里，能看到下面的输出

```
2019-11-28 01:43:23.000000000 +0000 8da5b8365552: {"container_id":"8da5b8365552b4c0c610ff5df3dc28509bfc5781ec580628143b00cf997d5b72","container_name":"/confident_curie","source":"stdout","log":"172.17.0.1 - - [28/Nov/2019:01:43:23 +0000] \"GET / HTTP/1.1\" 200 45"}
```

### 在docker-compose中使用示例

为了介绍下对日志常用的处理场景，这里将收集的日志分成三部分:

* 输出到fluentd的容器的标准输出，方便直接查看
* 保存一份到文件，按照日期自动切割日志
* 保存一份到Elasticsearch，方便通过Kibana面板直接查看

同时为了方便人查看，输出到fluentd标准输出和保存到日志文件的，只显示log字段的信息(不用`contanier_id`, `container_name`等暂时不关心的信息)，保存到Elastisearch的日志保存完整的结构。

首先，根据我们上面的需求，创建fluentd配置文件


```bash
$ cat etc/fluentd_docker_compose.conf
```

```bash
<source>
    @type forward
    port 24224
    bind 0.0.0.0
</source>

# docker相关的日志处理
<match docker.**>
    # docker相关的日志输出三份，一份输出到fluentd容器的标准输出，便于实时查看，另一份保存到文件, 还有一份保存到Elasticsearch
    @type copy

    # 输出到标准输出
    <store>
        @type stdout
        # 默认输出的格式是json格式，由于docker生成的日志，包含了容器信息等其他信息，不是很方便人去阅读。
        # 这里只输出我们关心的log字段
        # 使用stdout作为主format，single_value为子format，这样可以在输出log的同时保留直接tag和time信息
        <format>
           @type stdout
           output_type single_value
           message_key log
           add_newline true
        </format>
    </store>

    # 输出到文件
    <store>
        @type file
        # 使用tag和日期作为保存日志的文件名
        path /fluentd/log/${tag}/%Y%m%d
        # 合并多个flush chunk块到一个文件
        append true
        # 使用gzip压缩生成的日志文件
        compress gzip
        <format>
            @type stdout
            output_type single_value
            message_key log
            add_newline true
        </format>
        # 使用文件作为缓冲区
        <buffer tag, time>
            @type file
            chunk_limit_size 1M
            # 每隔30秒写一次日志
            flush_interval 30s
            # flush_at_shutdown true
           flush_mode interval
        </buffer>
    </store>

    # 输出到Eleastichsearch
    <store>
       @type elasticsearch
       host elasticsearch
       port 9200
       logstash_format true
       logstash_prefix fluentd
       logstash_dateformat %Y%m%d
       include_tag_key true
       type_name access_log
       tag_key @log_name
       flush_interval 1s
    </store>
</match>

# 其它日志处理
<match **>
    @type stdout
</match>
```

由于使用到了elasticsearch输出插件，而默认的fluentd中并没有安装这个插件，因此，我们需要自己定义`Dockfile`来安装elasticsearch插件


```
$ cat fluentd/Dockerfile
```

```Dockerfile

FROM fluent/fluentd:v1.7-1
USER root
RUN ["fluent-gem", "install", "fluent-plugin-elasticsearch"]
USER fluent
```

最后创建`docker-compose.yaml`文件

```bash
$ cat docker-compose.yaml
```

```yaml
version: '3'
services:
  httpd:
    image: httpd
    ports:
      - "5000:80"
    networks:
      - webnet
    depends_on:
      - fluentd
    logging:
      driver: fluentd
      options:
        fluentd-address: "localhost:24224"
        fluentd-retry-wait: '1s'
        fluentd-max-retries: '10'
        tag: docker.httpd

  fluentd:
    build: ./fluentd/
    volumes:
      - ./etc/:/fluentd/etc
      - ./log/:/fluentd/log
    ports:
      - "24224:24224"
      - "24224:24224/udp"
    environment:
      - FLUENTD_CONF=fluentd_docker_compose.conf
    networks:
      - webnet
    depends_on:
      - elasticsearch
      - kibana

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.3
    ports:
      - "9200:9200"
    networks:
      - webnet
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - discovery.type=single-node
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

  kibana:
    image: docker.elastic.co/kibana/kibana-oss:6.2.3
    environment:
      SERVER_NAME: kibana-server
      ELASTICSEARCH_URL: http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - webnet

networks:
  webnet:

volumes:
  es_data:
    driver: local
```

最后启动服务

```bash
$ docker-compose up
```

实际使用中发现，采用上面的方式启动服务后，有时间fluentd没法收集到httpd服务的日志，最后发现原因是如果在fluentd服务还没准备就绪的情况下就启动httpd服务，就会产生这种现象。因此，建议的做法是先启动fluentd, 再启动httpd

```bash
$ docker-compose up fluentd
# 等fluentd服务就绪后，再启动httpd服务
$ docker-compose up httpd
```

当然更优雅点的做法是控制docker-compose中服务的启动顺序，具体可以参考:

<https://docs.docker.com/compose/startup-order/>


测试，访问 httpd服务

```bash
# 可以多执行几次，产生多一些访问记录
$ curl http://127.0.0.1/5000
```

最后可以分别在fluentd的容器的终端，log目录，以及elasticsearch中看到保存的访问记录信息了。如下是通过Kibana面板看到的请求情况

![Kibana面板](http://images.wiseturtles.com/1575000628.png)

## 更多

更多关于Fluentd的使用方式可以参考官方文档

* [各种使用fluentd收集日志的场景](https://docs.fluentd.org/how-to-guides)
