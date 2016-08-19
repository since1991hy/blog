---
layout: post
title: redis 消息通知
categories: [redis, NoSQL]
description: redis 对消息通知的支持
keywords: redis, NoSQL
---

> redis 对消息通知功能的支持很丰富，可以实现任务队列，优先级队列以及“发布/订阅”模式。

## 任务队列

说到队列，很自然就能想到 Redis 的列表类型，之前介绍了使用 `lpush` 和 `rpop` 命令实现队列的概念。如果要实现任务队列，只需要让生产者将任务使用 `lpush` 命令加入到某个键中，另一边让消费者不断地使用 `rpop` 命令从该键中取出任务即可。

不过还有一点不完美的地方：当任务队列中没有任务时，消费者每秒都会调用一次 `rpop` 命令查看是否有新任务，如果可以实现一旦有新任务加入到任务队列就通知消费者就好了。借助 `brpop` 命令可以实现这样的需求。

`brpop` 命令和 `rpop` 命令相似，唯一的区别是当列表中没有元素时 `brpop` 命令会一直阻塞住连接，直到有新元素加入。

`brpop` 命令接收两个参数，第一个是键名，第二个是超时时间，单位是秒。当超过了此时间仍然没有获得新元素的话就会返回 `nil`。超时时间为 “0”, 表示不限制等待的时间，即，如果没有新元素加入任务队列，就会永远阻塞下去。

## 优先级队列

> 问题描述：当有多种任务同时存在时，应该优先执行某一种任务，为了实现这一目的，我们需要实现一个优先级队列。

`brpop` 命令可以同时接受多个键，其完整的命令格式为 `blpop key [key...] timeout`，如 `blpop queue:1 queue:2 0`。意义是同时检测多个键，如果所有的键都没有元素则阻塞，如果其中有一个键有元素则会从该键中弹出元素。

例如，打开两个 **redis-cli** 实例，在实例 A 中：

```bash
redis A > blpop queue:1 queue:2 queue:3 0
```

在实例 B 中：

```bash
redis B > lpush queue:2 task
(integer) 1
```

则实例 A 中会返回：

```bash
1) "queue:2"
2) "task"
```

如果多个键都有元素则按照从左到右的顺序取第一个键中的一个元素。我们现在 **queue:2** 和 **queue:3** 中各加入一个元素：

```bash
redis> lpush queue:2 task1
1) (integer) 1

redis> lpush queue:3 task2
1) (integer) 1
```

然后执行 `brpop` 命令：

```bash
redis> brpop queue:1 queue:2 queue:3 0
1) "queue:2"
2) "task1"
```

借助此特性可以实现区分优先级的任务队列。

## "发布/订阅"模式

除了实现任务队列外，Redis 还提供了一组命令可以让开发者实现**“发布/订阅”(publish/subscribe)** 模式。"发布/订阅"模式同样可以实现进程间的消息传递，原理如下：

“发布/订阅”模式中包含两种角色，分别是发布者和订阅者。订阅者可以订阅一个或若干个频道（channel），而发布者可以向指定的频道发送消息，所有订阅此频道的订阅者都会收到此消息。

发布者发布消息的命令是 `publish`，用法是 `publish channel message`，如向 channel.1 说一声 “hi”：

```bash
redis> publish channel.1 hi
(integer) 0
```

这样消息就发出去了。`publish` 命令的返回值表示接收到这条消息的订阅者的数量。

:bell: 发出去的消息不会被持久化，也就是说当有客户端订阅 channel.1 后只能接收到后续发布到该频道的消息，之前发送的就接收不到了。

订阅频道的命令是 `subscribe`，可以同时订阅多个频道，用法是 `sbuscribe channel [channel ...]`。现在新开一个 redis-cli 实例 A，用它来订阅 channel.1：

```bash
redis A> subscribe channel.1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel.1"
3) (integer) 1
```

执行 `subscribe` 命令后客户端会进入订阅状态，处于此状态下客户端不能使用除 `subscribe`、`unsubscribe`、`psubscribe` 和 `punsubscribe` 这 4 个属于“发布/订阅”模式之外的命令，否则会报错。

进入订阅状态后客户端可能会接收到 3 中类型的回复。每种类型的回复都包含 3 个值，第一个值是消息的类型，根据消息类型的不同，第二、三个值的含义也不同。消息类型可能的取值有以下 3 个。

1. **subscribe**：表示订阅成功的反馈信息。第二个值是订阅成功的频道名称，第三个值是当前客户端订阅的频道数量。

2. **message**：这个类型的回复是我们最关心的，表示收到的消息。第二个值表示产生消息的频道名称，第三个值是消息的内容。

3. **unsubscribe**：表示成功取消订阅某个频道。第二个值是对应的频道名称，第三个值是当前客户端订阅的频道数量，当此值为 0 时客户端会退出订阅状态。

### 按照规则订阅

除了可以使用 `subscribe` 命令订阅指定名称的频道外，还可以使用 `psubscribe` 命令订阅指定的规则。规则支持** blob风格通配符**格式。

```bash
redis C> psubscribe channel.?*
1) "psubscribe"
2) "channel.?*"
3) (integer) 1
```

规则 `channel.?*` 可以匹配 `channel.1` 和 `channel.10`，但不会匹配 `channel.` 。

:bell: 使用 `psubscribe` 命令可以重复订阅一个频道，如果某客户端执行了 `psubscribe channel.?` 和 `psubscribe channel.?*`，这时向 *channel.2* 发布消息后该客户端就会收到两条消息，而同时 `publish` 命令返回的值也是 2 而不是 1。同样的，如果有另一个客户端执行了 `sbuscribe channel.10` 和 `psubscribe channel.?*` 的话，向 *channel.10* 发送命令客户端也会收到两条消息（但是是两种类型：message 和 pmessage）。

`punsubscribe` 命令可以退订指定的规则，用法是 `punsubscribe [pattern [pattern...]]`，如果没有参数则会退订所有规则。

:bell: 使用 `punsubscribe` 命令只能退订通过 `psubscribe` 命令订阅的规则，不会影响直接通过 `sbuscribe` 命令订阅的频道；同样 `unsubscribe` 命令也不会影响通过 `psubscribe` 命令订阅的规则。
