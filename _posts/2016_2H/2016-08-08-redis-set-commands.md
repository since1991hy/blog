---
layout: post
title: redis 命令之集合操作
categories: [redis, NoSQL]
description: redis 中 set 类型的操作命令
keywords: redis, NoSQL
---

> redis 常见集合类操作命令手册。集合的性质：无序性，确定性，唯一性。

## 常见 set 操作命令

命令 | 作用 | 备注
----| ----- | ------
sadd key value [value...] | 往集合 key 中增加元素，返回增加成功的个数 |
smembers key | 返回集合中所有的元素 |
srem value1 value2... | 删除集合中值为 value1，value2 的元素，返回值：忽略不存在的元素后，真正删除掉的元素的个数 |
spop key | 返回并删除集合 key 中的 1 个随机元素 | **随机**体现了集合的**无序性**。<br/> 可使用的场景：抽奖。（保障了随机和不重复）
srandmember key | 返回集合 key 中随机的 1 个元素 |
sismember key value | 判断 value 是否在集合 key 中：是返回 1，否返回 0。 |
scard key | 返回集合中元素的个数 |
smove source dest value | 把 source 中的 value 删除，并添加到 dest 集合中 |
sinter key1 key2 key3 | 求出 key1, key2, key3 三个集合中的交集并返回 |
sunion key1 key2 key3 | 求出 key1, key2, key3 的并集并返回 |
sdiff key1 key2 key3 | 求出 key1 与 key2, key3 的差集，即，key1 - key2 - key3 |
sinterstore dest key1 key2 key3 | 求出 key1, key2, key3 三个集合的交集，并赋值给 dest |
sunionstore dest key1 key2 key3 | 求出 key1, key2, key3 三个集合的并集，并赋值给 dest |
sdiffstore dest key1 key2 key3 | 求出 key1 与 key2, key3 的差集，并赋值给 dest |

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
