---
layout: post
title: redis 事务
categories: [redis, NoSQL]
description: redis 支持的简单事务
keywords: redis, NoSQL
imgpath: /blog/images/2016_2H/
---

> redis 作为一款数据存储系统，当然也支持事务，但是要注意和 mysql 数据库中事务的区别，和 mysql 中的事务相比，redis 支持的事务非常简单（不支持回滚，只能取消）。

> Redis 保证了一个事务中的所有命令要么都执行，要么都不执行。如果在发送 `exec` 命令前客户端断线了，则 Redis 会清空事务队列，事务中的所有命令都不会执行。而一旦客户端发送了 `exec` 命令，所有的命令就都会被执行，即使此后客户端断线也没关系，因为 Redis 中已经记录了所有要执行的命令。

## redis 和 mysql 事务的对比

  | mysql | redis
---| ----- | -----
开启 | `start transaction` 或者 `begin` | `multi`
语句 | 普通 sql | 普通命令
失败 | `rollback` 回滚 | `discard` 取消
成功 | `commit` 提交 | `exec`

:bell: 在 `multi` 后面的语句中，语句出错可能有 2 种情况：

* 运行前就能发现错误的命令。  
 （如输入一个不存在的命令或命令参数个数不正确）。  
  这种情况，`exec` 报错，**所有语句得不到执行**，相当于**执行了**  `discard`。

* 运行时才发现错误的命令。  
  比如：`zadd` 操作 **link** 对象。  
  `exec` 之后，会**执行正确的语句，并跳过所有不正确的语句**。

:bell: Redis 的事务没有关系数据库提供的回滚（rollback）功能。为此开发者必须在事务执行出错后自己收拾剩下的摊子（将数据库复原回事务执行前的状态等）。

## redis 事务运行实例

看下面 2 个模拟银行账户转账的例子：

* 运行前就能发现错误的语法。

![]({{ page.imgpath }}redis-transaction-demo1.png)

* 运行时才发现错误的命令。

![]({{ page.imgpath }}redis-transaction-demo2.png)

:memo: 所以本文开头提到，redis 只支持简单的事务，不支持 **回滚**，只能 **取消**。

## 其它问题

### 模拟春节买票场景

> 我正在买票（买票执行操作 `decr ticket` 和 `decrby money 100`），现在车票只剩下 1 张了，如果在我开启事务（`multi`）和提交事务（`exec`）之前，票被别人买走了--即，ticket 变成 0 了。  
我该如何观察这种情景，并不再提交。

如果不做任何处理，上面的情况执行的结果可能如下：

```bash
# 初始化，剩余 1 张票，我的账户余额为 300 （穷人一枚，求土豪接济 （-、-））
redis 127.0.0.1:6379> set ticket 1
OK
redis 127.0.0.1:6379> set myAccount 300
OK

# 开启事务
redis 127.0.0.1:6379> multi
OK
redis 127.0.0.1:6379> decr ticket
QUEUED
redis 127.0.0.1:6379> decrby myAccount 100
QUEUED

# 这时，其它客户端执行了 decr ticket，ticket = 0

redis 127.0.0.1:6379> exec
1) (integer) -1
2) (integer) 200
# ticket = -1，出不了票，还把钱扣了
# 小爷这小暴脾气，这还得了  (-、-)
```

悲观的想法：  
世界充满危险 :scream:，肯定有人和我抢，给 ticket 上锁，只有我能操作。**[悲观锁]**

乐观的想法：  
没有那么多人和我抢 :smile:，因此，我只需要注意--有没有人更改 ticket 的值就可以了。**[乐观锁]**

:memo: redis 的事务中，启用的是乐观锁，只负责监测 key 有没有被改动。具体的命令：`watch` 命令。

使用 `watch` 命令后执行上面的流程：

```bash
# 初始化，剩余 1 张票，我的账户余额为 100 （就剩一张毛爷爷了，金主在哪里）
redis 127.0.0.1:6379> set ticket 1
OK
redis 127.0.0.1:6379> set myAccount 100
OK

# ********************************
# 开启对 ticket 的监视
redis 127.0.0.1:6379> watch ticket
OK

# ticket 被监视后发生了变化，可以在这里执行，也可以在事务提交前在其它客户端执行
redis 127.0.0.1:6379> decr ticket
(integer) 0
# ********************************

# 开启事务
redis 127.0.0.1:6379> multi
OK
redis 127.0.0.1:6379> decr ticket
QUEUED
redis 127.0.0.1:6379> decrby myAccount 100
QUEUED

# 如果是其它客户端修改 ticket，要在下面的 exec 之前

redis 127.0.0.1:6379> exec
(nil)

redis 127.0.0.1:6379> mget ticket myAccount
1) "0"
2) "100"
# 好了，告诉你个好消息：你的钱没少。
# 告诉你个坏消息：没票了，你回不了家了（-、-）
```

:memo: `watch key [key ...]`，监听一个或多个 key 有没有变化，如果任意一个 key 有变，则事务取消。

:memo: `unwatch` 取消所有 watch 监听。
