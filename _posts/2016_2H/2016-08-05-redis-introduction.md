---
layout: post
title: redis 简介及下载安装
categories: [redis, NoSQL]
description: redis 简介
keywords: redis, NoSQL
imgpath: /blog/images/2016_2H/
---
>redis 是一个开源的，高级的 key-value 存储系统，可以用用来存储字符串，哈希结构，链表，集合，有序集合。因此，常用来提供数据结构服务。

## 什么是 redis

[官网](http://redis.io) 介绍如下：

>Redis is an open source (BSD licensed), in-memory **data structure store**, used as database, cache and message broker.

>>**What is Memcached?**
Free & open source, high-performance, distributed memory object **caching system**.

 :bell: 注意：redis 说自己**是**一个**数据结构存储**，可以用来做缓存，而**不**像 memcached 那样说自己**是缓存系统**。

## redis 和 memcached 相比的独特之处

* redis 可以用来做存储，而 memcached 是用来做缓存。（这个特点主要是因为其有“**持久化**”的功能。）
* 存储的数据有“结构”，对于 memcached 来说，存储的数据只有 1 种类型--字符串，而 redis 则可以存储字符串，链表，哈希结构，集合，有序集合。

## 下载安装

### 1. 到 [官网](http://redis.io) 下载最新版或者最新 stable 版

```bash
cd /usr/local/src

#下载 redis 源码包
wget http://download.redis.io/releases/redis-3.2.3.tar.gz
```

### 2. 解压并进入目录

```shell
# 解压
tar -zxvf redis-3.2.3.tar.gz

# 进入解压后的目录
cd redis-3.2.3
```

### 3. 编译

不用 `configure`，直接 `make`（如果是 32 位机器 `make 32bit`）

### 4. 可选步骤 `make test`

直接运行可能报错，提示需要安装 **tcl**

```bash
# 安装 tcl 脚本库
yum install tcl

# 运行测试
make test
```

### 5. 安装到指定目录

```bash
# PREFIX 指定安装目录进行安装
make PREFIX=/usr/local/redis install
```

安装后的目录结构如下：  
![]({{ page.imgpath }}redis-directory-structure.png "redis 目录结构")

### 6. 复制配置文件

```bash
# 从 redis 源码中拷贝一份配置文件
cp /usr/local/src/redis-2.3.2/redis.conf /usr/local/redis
```

### 7. 启动与连接

```bash
cd /urs/local/redis

# 指定配置文件启动服务器
./bin/redis-server redis.conf
```
服务器启动后的界面如下：  
![]({{ page.imgpath }}redis-startup-gui.png)

:bell: 注意：这个窗口关闭后 redis 服务器就关闭了，想要用客户端连接就必须开启另一个 bash 终端。

:memo: 可以让 redis 服务器以后台服务的形式运行，实现方式：修改配置文件 **redis.conf**.

```bash
vim /usr/local/redis/redis.conf
# 将其中一行配置 【daemonize no】修改为 【daemonize yes】
```
再次启动 redis 服务器：  
![]({{ page.imgpath }}redis-startup-daemon.png)

使用客户端进行连接：

```bash
# 进入 redis 安装目录
cd /usr/local/redis

# 连接
# 默认连接本地的 6379 端口
./bin/redis-cli
# 显示指定连接信息
./bin/redis-cli -h localhost -p 6379
```
