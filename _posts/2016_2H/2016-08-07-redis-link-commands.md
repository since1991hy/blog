---
layout: post
title: redis 命令之链表操作
categories: [redis, NoSQL]
description: redis 中 list 类型的操作命令
keywords: redis, NoSQL
---

> redis 常见链表类操作命令手册。链表类型（list）可以存储一个有序的字符串列表，常用的操作是向列表两端添加元素，或者获取列表的某一个片段。

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

---

另外，把列表类型作为队列使用时，`rpoplpush` 命令可以很直观地在多个队列中传递数据。当 **source** 和 **destination** 相同时，`rpoplpush` 命令会不断地将队尾的元素移到队首，借助这个特性，我们可以实现一个**网站监控系统**：

使用一个队列存储需要监控的网址，然后监控程序不断地使用 `rpoplpush` 命令循环取出一个网址来测试可用性。这里使用 `rpoplpush` 命令的好处在于在程序执行过程中仍然可以不断地向网址列表中加入新网址，而且整个系统容易扩展，允许多个客户端同时处理队列。

### brpop/blpop 使用场景

长轮询 ajax，在线聊天时能够用到。

## 实践

### 存储文章 ID 列表，实现分页功能

可以使用列表类型键 `posts:list` 记录文章 ID 列表。当发布新文章时使用 `lpush` 命令将新文章的 ID 加入这个列表中，另外删除文章时也要记得把列表中的文章 ID 删除，就像这样：`lrem posts:list 1 要删除的文章 ID`。

有了文章 ID 列表，就可以使用 `lrange` 命令来实现文章的分页显示了。

另外，使用列表类型键存储文章 ID 列表有以下两个问题：

1. 文章的发布时间不易修改：修改文章的发布时间不仅要修改 `post:文章 ID` 中的 **time** 字段，还需要按照实际的发布时间重新排列 `post:list` 中的元素顺序，而这一操作相对比较繁琐。

2. 当文章数量较多时访问中间的页面性能较差：列表类型是通过双向链表实现的，所以当列表元素非常多时访问中间的元素效率并不高。

但是如果博客不提供修改文章时间的功能并且文章数量也不多时，使用列表类型也不失为一种好办法。

### 存储评论列表

在博客中还可以使用列表类型键存储文章的评论。考虑到 **读取评论时需要获得评论的全部数据（评论者姓名，联系方式，评论时间和评论内容），不像文章一样有时只需要文章标题而不需要文章正文** 。所以适合将一条评论的各个元素序列化成字符串后作为列表类型键中的元素来存储。

使用列表类型键 `post:文章 ID:comments` 来存储某个文章的所有评论。发布评论的伪代码如下（以 ID 为 42 的文章为例）：

```php
# 将评论序列化成字符串
$serializedComment = serialize($author, $email, $time, $content)

LPUSH post:42:comments $serializedComment
```

读取评论时同样使用 `lrange` 命令，然后执行**反序列化**即可。

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
