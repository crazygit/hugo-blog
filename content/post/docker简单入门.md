---
title: docker简单入门
date: 2018-02-12T21:15:29+08:00
tags:
    - docker
categories:
    - docker容器
slug: docker-get-started
description: >
    Dokcer使用简单入门
---

## Docker安装

根据自己的操作系统，参考[官方文档](https://docs.docker.com/install/)完成安装

## 测试是否安装成功，运行第一个容器

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:66ef312bbac49c39a89aa9bcc3cb4f3c9e7de3788c944158df3ee0176d32b751
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
...
```

<!--more-->

## 创建镜像

### 准备创建镜像需要的文件

创建一个空的目录`docker-demo`，在里面放入三个文件

`Dockerfile`

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

`app.py`

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)


@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"),
                hostname=socket.gethostname(), visits=visits)


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

`requirements.txt`

```
Flask
Redis
```

创建好的效果如下:

```	bash
$ ls
Dockerfile		app.py			requirements.txt
```

### 创建镜像

```
$ docker build -t friendlyhello .

$ docker images

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```

### 运行容器

```
$ docker run -p 4000:80 friendlyhello
```
访问<localhost:4000>查看效果


## 分享镜像
由于Dockerhub国内访问速度特别慢，这里用阿里云代替

### 阿里云Docker

* [web控制台](https://cr.console.aliyun.com/)
* [容器Hub](https://dev.aliyun.com/search.html)

### 登录

```
$ docker login registry.cn-hangzhou.aliyuncs.com
Username: your_username
Password:  your_password
Login Succeeded
```

### 给镜像打标签

```
$ docker tag friendlyhello registry.cn-hangzhou.aliyuncs.com/crazygit/get-started:part2
```

### 发布镜像

```
$ docker push registry.cn-hangzhou.aliyuncs.com/crazygit/get-started:part2
```

## 镜像加速

默认从docker hub上面下载镜像实在太慢了，可以选择使用国内的镜像，比如下面两家:

* [阿里云](https://cr.console.aliyun.com/)
* [DaoCloud](http://get.daocloud.io/)
* [Docker 中国官方镜像加速](https://www.docker-cn.com/registry-mirror)

以使用阿里云为例，创建开发者账号，获取加速器链接，我的加速器链接链接是

<https://b06kc77a.mirror.aliyuncs.com>

中国官方镜像加速方法:

```
docker pull registry.docker-cn.com/myname/myrepo:mytag
```
