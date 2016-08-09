---
layout: post
title: redis 命令之字符串操作
categories: [redis, NoSQL]
description: redis 中 string 类型的操作命令
keywords: redis, NoSQL
---

> redis 常见字符串类操作命令手册。

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

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
