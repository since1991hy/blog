---
layout: post
title: redis 命令之通用操作
categories: [redis, NoSQL]
description: redis 中和具体的存储结构无关的普遍适用的操作命令
keywords: redis, NoSQL
---

> redis 由于其支持的丰富的数据结构，有很多操作命令，以下列举一些常用的和数据结构无关的通用的操作命令。

## 通用 key 操作命令

命令 | 作用 | 样例 | 备注
----| -----| -----| ------
keys pattern | 查询相应的 key | `keys *` //返回所有 key <br/> `keys o*` //返回所有以 "o"开头的 key | 允许模糊查询 key，有 3 个通配符：`*`,`?`,`[]` <ul><li>*:通配任意多个字符</li><li>通配单个字符</li><li>通配括号中的 1 个字符</li></ul>
randomkey | 返回随机 key 值 | `randomkey` |
type key | 返回 key 存储的值的类型 | `type name` //返回 string | 返回类型有：`string`, `list`, `set`, `order set`, `hash`
exists key | 判断 key 是否存在 | `exists name` | 存在返回 `1`，不存在返回 `0`
del key1 [key2 ...] | 删除一个或多个键 | `del name` | 返回真正删除的 key 的数量（不存在的 key 忽略掉返回 0）
rename key newkey | 给 key 改名为 newkey | `rename pwd password` | :bell: 如果 newkey 已存在，则 newkey 的原值被覆盖
renamenx key newkey | 把 key 改名为 newkey | `renamenx pwd password` | 发生修改返回 1，未发生修改返回 0；<br/> nx-->not exists，即，newkey 不存在时才改名。
move key dbid | 将 key 移动到其它数据库 | `move name 1` //将 name 移动到索引为 1 的数据库 | 连接后默认的 dbid 为 0
expire key 整型值 | 设置 key 的生命周期，以秒为单位 | `expire name 10` |
ttl key | 查询 key 的生命周期，返回秒数 | `ttl name`| :bell: 早期的版本，对于不存在的 key 和已过期的 key 都返回 -1；<br/> redis 2.8 之后，对于不存在的 key 返回 -2。
pexpire key 整型值 | 设置 key 的生命周期，以毫秒为单位 | `pexpire name 10000` |
pttl key | 查询 key 的生命周期，返回毫秒数 | `pttl name` |
persist key | 把指定 key 置为永久有效 | `persist name` |

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
