---
layout: post
title: redis 命令之字符串操作
categories: [redis, NoSQL]
description: redis 中 string 类型的操作命令
keywords: redis, NoSQL
---

> redis 常见字符串类操作命令手册。字符串类型是 Redis 中最基本的数据类型，它能存储任何形式的字符串，包括二进制数据。你可以用其存储用户的邮箱、JSON 化的对象甚至是一张图片。一个字符串类型键允许存储的数据的最大容量是 512MB。  
> 字符串是其他 4 种数据类型的基础，其他数据类型和字符串类型的差别从某种角度上来说只是组织字符串的形式不同。例如，列表类型是以列表的形式组织字符串，而集合类型是以集合的形式组织字符串。

## 常见 string 操作命令

命令 | 作用 | 备注
----| ----- | ------
set key value [ex 秒数]/[px 毫秒数] [nx]/[xx] | 设定字符串 | :bell: 如果 `ex`，`px` 同时写，以后面的有效期为准。<br/> `nx`：表示 key 不存在时执行操作；`xx`：表示 key 存在时执行操作。
mset key val [key val...] | 一次性设置多个键值 | m --> mulit
get key | 获取 key 的值 |
mget key [key...] | 一次性获取多个 key 值 |
setrange key offset value | 把字符串的 offset 偏移字节改成 value | 如：`set greet hello`，`setrange greet 2 x`，此时，`get greet` 得到 “hexlo”。
append key value | 把 value 追加到 key 的原值上 |
getrange key start stop | 获取字符串中 [start, stop] 范围的值 | :bell: 对于字符串的下标，左数从 0 开始，右数从 -1 开始。所以，`getrange key 0 -1` 可以获取整个字符串的长度。
getset key newvalue | 获取并返回旧值，设置新值 |
incr key | 指定的 key 的值自增 1，并返回自增后的值 | :bell: 1. 不存在的 key 当成 0，在 incr 操作；2. 范围为 64 位有符号数。
incrby key intnumber | 指定的 key 增加 intnumber | :bell: intnumber 必须是整数
incrbyfloat key floatnumber | 指定的 key 增加 floatnumber |
decr key | 指定的 key 自减 1 |
decrby key intnumber | 指定的 key 减 number |
getbit key offset | 获取值的二进制表示中，对应位上的值（从左，从 0 编号） |
setbit key offset value | 设置 offset 对应二进制位上的值，返回该位的旧值 | :bell: 1. 如果 offset 过大，则会在中间填充 0；2. offset 最大 `2^32 - 1`，可推出最大的字符串为 512M。
bitop operation destkey key1 [key2...] | 对 key1，key2 作 operation 操作，并将结果保存到 destkey 上。 operation 操作可以是：`and`, `or`, `not`, `xor`。| 对于 `not` 操作，key 不能多个

## 实践

### 文章访问量统计

博客的一个常见的功能是统计文章的访问量，我们可以为每篇文章使用一个名为 `post:文章 id:page.view` 的键来记录文章的访问量，每次访问文章的时候使用 `incr` 命令使相应的键值递增。

### 生成自增 ID

如何为每篇文章生成一个唯一的 ID 呢？在关系数据库中我们通过设置字段属性为 `auto_increment` 来实现每增加一条记录自动为其生成一个唯一的递增 ID 的目的，而在 Redis 中可以通过另一种模式来实现：**对于每一类对象使用名为 `对象类型（复数形式):count`** 的键递增该键的值。

### 存储文章结构

由于每个字符串类型的键只能存储一个字符串，而一篇 blog 是由标题、正文、作者与发布时间等多个元素构成。为了存储这些元素，我们需要使用序列号函数（如 PHP 中的 `serialize` 和 JavaScript 中的 `JSON.stringify` ）将他们转换为一个字符串。

至此写出发布新文章时与 Redis 操作相关的伪代码：

```php
# 首先获得新文章的 ID
$postID = INCR posts:Count

# 将博客文章的诸多元素序列化成字符串
$serailizedPost = serialize($title, $content, $author, $time)

# 将序列化后的字符串存入一个字符串类型的键中
SET post:$postID:data $serailizedPost
```

获取文章数据的伪代码如下（以访问 ID 为 42 的文章为例）：

```php
# 从 Redis 中读取文章数据
$serializedPost = GET post:42:data

# 将文章数据反序列化成文章的各个元素
$title, $content, $author, $time = unserialize($serializedPost)

# 获取并递增文章的访问量
$count = INCR post:42:page.view
```

> 上述代码的问题：想要做一个博客列表页，在列表页中每篇文章只需要显示标题部分，可是使用上面的方法，若想获得文章的标题，必须把整个文章数据字符串取出来反序列化，而其中占用空间最大的文章内容部分却是不需要的，这会在传输和处理时造成资源的极大浪费。

:memo: 除了使用序列化函数将文章的多个元素存入一个字符串类型键中外，还可以对每个元素使用一个字符串类型键来存储。【参见哈希结构】

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
