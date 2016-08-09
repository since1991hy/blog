---
layout: post
title: redis 命令之哈希操作
categories: [redis, NoSQL]
description: redis 中 hash 类型的操作命令
keywords: redis, NoSQL
---

> redis 常见哈希类操作命令手册。

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

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
