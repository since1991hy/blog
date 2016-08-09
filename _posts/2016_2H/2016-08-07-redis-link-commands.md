---
layout: post
title: redis 命令之链表操作
categories: [redis, NoSQL]
description: redis 中 list 类型的操作命令
keywords: redis, NoSQL
---

> redis 常见链表类操作命令手册。

## 常见 string 操作命令

命令 | 作用 | 备注
----| ----- | ------
lpush key value [value...] | 把值插入到链表头部 | l --> left
rpush key value [value...] | 把值插入到链表尾部 | r --> right
lrange key start stop | 返回链表中 [start, stop] 中的元素 | :bell: 规律：左数从 0 开始，右数从 -1 开始。所以，查看所有链表元素可以 `lrange key 0 -1`
lpop key | 返回并删除链表头元素 | l --> link(list)
rpop key | 返回并删除链表尾元素 |
lrem key count value | 从 key 中删除 value | rem --> remove <br/> :bell: 删除 count 的绝对值个 value 后结束：a. count > 0,从表头删除；b. count < 0,从表尾删除；c. count = 0, 移除表中所有与 value 相等的值。
ltrim key start stop | 剪切 key 对应的链表，切 [start, stop] 一段，并把该段重新赋给 key |
lindex key index | 返回 index 索引上的值，eg：`lindex list 2` |
llen key | 计算链表的元素个数 |
linsert key before/after search value | 在 key 链表中寻找 `search`，并在 search 之前/之后插入 value | :bell: 一旦找到一个 search 后，命令就结束了，因此不会插入多个 value。
rpoplpush source dest | 把 source 的尾部拿出，放在 dest 头部并返回该单元值 | source 和 dest 可相同。
brpop/blpop key timeout | 等待弹出 key 的尾/头元素，timeout 为等待超时时间，如果 timeout 为 0，则一直等待 | b --> block <br/> block until one is available

### rpoplpush 使用场景

task + back 双链表完成安全队列。

业务逻辑：
1. `rpoplpush task back`，返回 task 链表中的最后一项任务，并将其添加到 back 队列中。
2. 接收返回值，并做业务处理。
3. 如果业务处理成功，`rpop back` 清除任务；  
   如果不成功，下次从 back 链表中取任务去执行。

### brpop/blpop 使用场景

长轮询 ajax，在线聊天时能够用到。

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
