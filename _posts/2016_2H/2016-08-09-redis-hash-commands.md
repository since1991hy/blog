---
layout: post
title: redis 命令之哈希操作
categories: [redis, NoSQL]
description: redis 中 hash 类型的操作命令
keywords: redis, NoSQL
imgpath: /blog/images/2016_2H/
---

> redis 常见哈希类操作命令手册。Redis 是采用字典结构以键值对的形式存储数据的，而散列类型（hash）的键值也是一种字典结构，其存储了字段（field）和字段值的映射，但字段值只能是字符串，不支持其他数据类型，换句话说，散列类型不能嵌套其他的数据类型。

:bell: 除了散列类型外，Redis 的其他数据类型同样不支持数据类型嵌套。比如集合类型的每个元素都只能是字符串，不能是另一个集合或散列表。

## 常见 hash 操作命令

命令 | 作用 | 备注
----| ----- | ------
hset key field value | 把 key 中的 field 域的值设为 value | :bell: 如果没有 field 则直接添加；如果有则覆盖原 field 域的值
hmset key field1 value1 [field2 value2...] | 设置 field1->field N 个域，对应的值是 value1->value N | h --> hash
hget key field | 返回 key 中 field 域的值 |
hmget key field1 [field2...] | 返回 key 中 field1 [field2...] 的值 |
hgetall key | 返回 key 中所有域及其值 |
hdel key field | 删除 key 中的 field 域 |
hlen key | 返回 key 中元素的数量 |
hexists key field | 判断 key 中有没有 field 域，有返回 1，没有返回 0 |
hincrby key field value | 把 key 中的 field 域的值增长 **整数值** value。 |
hincrbyfloat key field value | 把 key 中的 field 域的值增长 **浮点值** value。 |
hkeys key | 返回 key 中所有的 field |
hvals key | 返回 key 中所有的 value |

## 实践

### 存储文章数据结构（续）

> 在讲解 redis 字符串时介绍了可以将文章对象序列化后使用一个字符串类型键存储，可是这种方法无法提供对博客的单个字段进行读写操作；另外，即使只需要文章标题，程序也不得不将包括文章内容在内的所有文章数据取出并反序列化，比较消耗资源。

除此之外，还有一种方法是组合使用多个字符串类型键来存储一篇文章的数据，如下图所示：

![]({{ page.imgpath }}redis-store-blog-multistring.png)

使用这种方法的好处在于无论获取还是修改文章数据，都可以只针对某一属性进行操作，十分方便。

但是使用 hash 类型则更适合此场景，使用散列类型的存储结构如下图所示：

![]({{ page.imgpath}}redis-store-blog-hash.png)

使用散列类型存储文章数据看起来更加直观，也更容易维护（比如可以使用 `hgetall` 命令获得一个对象的所有字段，删除一个对象时只需要删除一个键），另外存储同样的数据散列类型往往比字符串类型更加节约空间。

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
