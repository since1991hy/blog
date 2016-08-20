---
layout: post
title: Redis 持久化之 AOF
categories: [redis, NoSQL]
description: AOF 方式的持久化
keywords: redis, NoSQL
---

> 当使用 Redis 存储非临时数据时，一般需要打开 AOF 持久化来降低进程终止导致的数据丢失。AOF 可以将 Redis 执行的每一条写命令追加到硬盘文件中，这一过程显然会降低 Redis 的性能，但是大部分情况下这个影响是可以接受的，另外使用较快的硬盘可以提高 AOF 的性能。

## 开启 AOF

默认情况下 Redis 没有开启 **AOF（append only file）** 方式的持久化，可以通过 `appendonly` 参数启用：

```bash
appendonly yes
```

开启 AOF 持久化后每执行一条命令会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件。AOF 文件的保存位置和 RDB 文件的位置相同，都是通过 `dir` 参数设置的，默认的文件名是 `appendonly.aof`，可以通过 `appendfilename` 参数修改：

```bash
appendfilename appendonly.aof
```

## AOF 的实现

AOF 文件以纯文本的形式记录了 Redis 执行的写命令，例如在开启 AOF 持久化的情况下执行了如下 4 个命令：

```shell
set foo 1
set foo 2
set foo 3
get foo
```

Redis 会将前 3 条命令写入 AOF 文件中，此时 AOF 文件中的内容如下：

```shell
*2
$6
SELECT
$1
0

*3
$3
set
$3
foo
$1
1

*3
$3
set
$3
foo
$1
2

*3
$3
set
$3
foo
$1
1
```

可见 AOF 文件的内容正是 Redis 客户端向 Redis 发送的原始通信协议的内容。

从中可见 Redis 确实只记录了前 3 条命令。然而这时有一个问题是：**前 2 条命令其实都是冗余的**，因为这两条的执行结果会被第三条命令覆盖。随着执行的命令越来越多，AOF 文件的大小也会越来越大，即使内存中实际的数据可能并没有多少。

很自然地，我们希望 Redis 可以自动优化 AOF 文件，就上例而言，就是将前两条无用的记录删除，只保留第三条。实际上 Redis 也是这样做的，**每当达到一定条件时 Redis 就会自动重写 AOF 文件**，这个条件可以在配置文件中设置：

```
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

`auto-aof-rewrite-percentage` 参数的意义是当前的 AOF 文件大小超过上一次重写时的 AOF 文件大小的百分之多少时会再次进行重写，如果之前没有重写过，则以启动时的 AOF 文件大小为依据。

:bell: 这个参数在 AOF 文件稍大一点的时候还可以接受，当 AOF 文件比较小时，就不太好了。如：1KB->2KB 触发重写，2KB->4KB 触发重写...这么小的文件根本没有重写的必要嘛。

`auto-aof-rewrite-min-size` 参数限制了允许重写的最小 AOF 文件大小，通常在 AOF 文件很小的情况下，即使其中有很多冗余的命令我们也不太关心。

除了让 Redis 自动执行重写外，我们还可以主动使用 `bgrewriteaof` 命令手动执行 AOF 重写。

上例中的 AOF 文件重写后的内容如下：

```shell
*2
$6
SELECT
$1
0

*3
$3
set
$3
foo
$1
3
```

可见冗余的命令已经被删除了。重写的过程只和内存中的数据相关，和之前的 AOF 文件无关，这与 RDB 很相似，只不过二者的文件格式完全不同。

在启动时 Redis 会逐个执行 AOF 文件中的命令来将硬盘中的数据载入到内存中，载入的速度相较 RDB 会慢一些。

## 同步硬盘数据

虽然每次执行更改数据库内容的操作时，AOF 都会将命令记录在 AOF 文件中，但是事实上，由于操作系统的缓存机制，数据并没有真正地写入硬盘，而是进入了系统的硬盘缓存。在默认情况下系统每 30 秒会执行一次同步操作，以便将硬盘缓存中的内容真正地写入硬盘，在这 30 秒的过程中如果系统异常退出则会导致硬盘缓存中的数据丢失。一般来讲，启用 AOF 持久化的应用都无法容忍这样的损失，这就需要 Redis 在写入 AOF 文件后主动要求系统将缓存内容同步到硬盘中。在 Redis 中可以通过 `appendfsync` 参数设置同步的时机：

```
# appendfsync always
appendfsync everysec
# appendfsync no
```

默认情况下 Redis 采用 `everysec` 规则，即每秒执行一次同步操作。`always` 表示每次执行写入都会执行同步，这是最安全也是最慢的方式。`no` 表示不主动进行同步操作，而是完全交由操作系统来做（即每 30 秒一次），这是最快但最不安全的方式。一般情况下，使用默认值 `everysec` 就足够了，既兼顾了性能又保证了安全。

Redis 允许同时开启 AOF 和 RDB，既保证了数据安全又使得进行备份等操作十分容易。此时需要重新启动 Redis 后，Redis 会使用 AOF 文件来恢复数据，因为 AOF 方式的持久化可能丢失的数据更少。
