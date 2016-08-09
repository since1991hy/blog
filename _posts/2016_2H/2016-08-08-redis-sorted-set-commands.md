---
layout: post
title: redis 命令之有序集合操作
categories: [redis, NoSQL]
description: redis 中 sorted set 类型的操作命令
keywords: redis, NoSQL
---

> redis 常见有序集合类操作命令手册。  
> **sorted set** 允许进行排序，那么按照什么来排序呢，你肯定要给它一个排序因子或者叫排序的依据，所以 **sorted set** 中每个元素还要有一个排序因子或权重来作为排序的依据，我们统称为 **score**。

## 常见 sorted set 操作命令

命令 | 作用 | 备注
----| ----- | ------
zadd key score1 value1 score2 value2... | 添加元素 |
zrange key start stop [withscores] | 把集合排序后，返回名次在 [start, stop] 的元素，默认是升序排列。<br/> `withscores` 是把 score 也同时打印出来 |
zrangebyscore key min max [withscores] limit offset N | 集合升序排序后，取 score 在 [min, max] 内的元素，并跳过 offset 个元素，取出 N 个 | `limit offset N` 和 mysql 的分页比较像，`offset` 是偏移量，从第 `offset` 个开始取出 N 个。
zrank key member | 查询 member 的排名（升序，从 0 开始） |
zrevrank key member | 查询 member 的排名（降序，从 0 开始） | rev --> reverse
zrem key value1 value2... | 删除集合中的元素 |
zremrangebyscore key min max | 按照 **score** 来删除，删除 score 在 [min, max] 之间的元素 |
zremrangebyrank key start end | 按照 **排名** 删除元素，删除名次在 [start, end] 之间的元素 |
zcard key | 返回元素个数 |
zcount key min max | 返回 score 在 [min, max] 区间内元素的数量 |
zinterstore destination numkeys key1 [key2...] [weights weight1 [weight2...]] [aggregate sum/min/max] | 求 key1, key2 的交集，key1, key2 的权重分别是 weight1, weight2。<br/> 聚合的方法有：sum/min/max。<br/> 聚合的结果保存在 destination 中。 |
zunionstore destination numkeys key1 [key2...] [weights weight1 [weight2...]] [aggregate sum/min/max] | |

### zinterstore 详解

> 如何理解 `weights`？

可以通过 `weights` 设置不同 key 的权重，交集时，该 key 的最终 score 的值为 `score * weight`。

> 如何理解 aggregate？

:bell: 如果有交集，交集元素在各个集合中分别有自己的 score，`aggregate` 指定了聚合之后交集元素应该使用什么样的 score。

* `aggregate sum` : 各集合的 score 相加，作为聚合后的 score。

* `aggregate min` : 各集合中的 score 的最小值，作为聚合后的 score。

* `aggregate max` : 各集合中的 score 的最大值，作为聚合后的 score。

### zinterstore 示例

```bash
# 向 z1，z2 两个有序集合中添加元素
redis 127.0.0.1:6379> zadd z1 2 a 3 b 4 c
(integer) 3
redis 127.0.0.1:6379> zadd z2 2.5a 1 b 8 d
(integer) 3

# 默认使用 sum 方式聚合，聚合 z1，z2，结果保存到 tmp 中
# 等价于 `zinterstore tmp 2 z1 z2 aggregate sum`
redis 127.0.0.1:6379> zinterstore tmp 2 z1 z2
(intger) 2

# 查看聚合结果
redis 127.0.0.1:6379> zrange tmp 0 -1
1)"b"
2)"a"
redis 127.0.0.1:6379> zrange tmp 0 -1 withscores
1)"b"
2)"4"
3)"a"
4)"4.5"


# 示例聚合方式 min
redis 127.0.0.1:6379> zinterstore tmp 2 z1 z2 aggregate min
(integer) 2
redis 127.0.0.1:6379> zrange tmp 0 -1 withscores
1)"b"
2)"1"
3)"a"
4)"2"

# 权重示例
redis 127.0.0.1:6379> zinterscore tmp 2 z1 z2 weights 1 2
(intger) 2
redis 127.0.0.1:6379> zrange tmp 0 -1 withscores
1)"b"
2)"5"
3)"a"
4)"7"

```

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
