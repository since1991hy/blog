---
layout: post
title: redis 命令之集合操作
categories: [redis, NoSQL]
description: redis 中 set 类型的操作命令
keywords: redis, NoSQL
imgpath: /blog/images/2016_2H/
---

> redis 常见集合类操作命令手册。集合的性质：无序性，确定性，唯一性。  
> 集合类型的常用操作是向集合中加入或删除元素、判断某个元素是否存在等，由于**集合类型在 redis 内部是使用值为空的散列表（hash table）实现的**，所以这些操作的时间复杂度都是 O(1)。

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

## 实践

### 存储文章标签

考虑到一篇文章的所有标签都是互不相同的，而且展示时对这些标签的排列顺序并没有要求，可以使用集合类型键存储文章标签。

对每篇文章使用键名为 `post:文章 ID:tags` 的键存储该篇文章的标签。具体操作伪代码如下：

```php
# 给 Id 为 42 的文章增加标签
SADD post:42:tags tag1 tag2 tag3

# 删除标签
SREM post:42:tags tag1

# 显示所有的标签
$tags = SMEMBERS post:42:tags
print $tags
```

### 通过标签搜索文章

有时还需要列出某个标签下的所有文章，设置需要获得同时属于某几个标签的文章列表，这种需求在传统关系数据库中实现起来比较复杂（可能需要在**文章表、标签表、文章标签中间表**之间进行多表关联查询）。

在 redis 中实现起来就很简单了。

具体做法是为每个标签使用一个名为 `tag:标签名称:posts` 的集合类型键存储标有该标签的文章 ID 列表。存储结构如下图：

![]({{ page.imgpath }}redis-store-blog-tags-by-set.png)

最简单的，获取标记为 **"MySQL"** 标签的文章时只需要使用命令 `smembers tag:MySQL:posts` 即可。

如果要实现找到同时属于 **Java、MySQL、Redis** 3 个标签的文章，只需要将 `tag:Java:posts`、`tag:MySQL:posts`、`tag:Redis:posts` 这 3 个键去交集，借助 `sinter` 命令即可轻松完成。

## 更多

**点击 [这里](http://redis.io/commands) 去官网查看更多的，完整的操作命令。**
