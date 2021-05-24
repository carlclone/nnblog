---
title: Redis应用场景-理解总结
tags: 归档
date: 2017-08-09 14:57:31
---

无

<!-- more -->

# 数据类型
**所有类型都是基于键值对存在**

## String 字符串（也可以是数字）
常用命令`set,get,decr,incr,mget`
## Hash 哈希映射表 （PHP的数组就是hash table）
常用命令`hget,hset,hgetall`

使用场景：和php数组作用类似。结构化的信息，如用户的一系列信息，商品的信息，由许多子信息构成，使用字符串需要序列化存储，修改需要反序列化再修改，Redis的Hash结构可以使你像在数据库中Update一个属性一样只修改某一项属性值。

## List 
常用命令`lpush,rpush,lpop,rpop,lrange`
使用数据结构中的双向链表实现，lrange类似于线性表查询操作(关注列表，粉丝列表)，push和pop操作类似数据结构中的队列和栈的操作

## Set 集合
常用命令`sadd,spop,smembers,sunion `
类似于list，但是set中不允许有重复元素（自动排重）

使用场景：set提供了判断某个成员是否在一个set集合内。可以存储一些集合性的数据。比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。使用求交集实现共同关注，共同喜好，使用差集实现二度好友（可能认识的好友），还有求并集。

最后，上面对集合的操作，可以使用不同的命令选择返回给客户端，或者存到新的集合。

## Sorted Set 有序集合
常用命令`zadd,zrange,zrem,zcard`

和set类似，但多了一个score字段实现自动排序，有序的并且不重复的集合

使用场景：
1.微博的博文发布时间作为score，则取出来的时候就已经按时间排序
2.带优先级的消息队列，普通消息，重要消息

## Pub/Sub
发布和订阅功能，对某一个key值进行发布和订阅，发布后，所有订阅该key的客户端都会收到消息

使用场景：即时消息系统，实时聊天，群聊

还有些不常用的不列举了

# 其他使用场景
在数据类型介绍中对一些场景进行了总结，这里是一些补充的部分。
1.用于快速读取数据（缓存）
2.作为计数器（Counter），如点赞，收藏数

# 具体使用Demo，优化案例
1.显示最新的项目列表
`SELECT * FROM foo WHERE ... ORDER BY time DESC LIMIT 10   `  常用的显示最新10条数据，数据越多越慢

Demo：用户的最新n条评论
使用Redis的List的队列功能，先进先出，在存入数据库的同时将对应评论的ID LPUSH到列表中。
`LPUSH latest.comments` <ID>

裁剪列表长度为5000 
`LTRIM latests.comments 0 5000`  ，当需要获取超出5000个的最新评论的时候，才去数据库读取
```
FUNCTION get_latest_comments(start, num_items):  
    id_list = redis.lrange("latest.comments",start,start+num_items - 1)  
    IF id_list.length < num_items  
        id_list = SQL_DB("SELECT ... ORDER BY time LIMIT ...")  
    END  
    RETURN id_list  
END  
```

2.删除和过滤
这个还没看懂
```
我们可以使用LREM来删除评论。如果删除操作非常少，另一个选择是直接跳过评论条目的入口，报告说该评论已经不存在。
       有些时候你想要给不同的列表附加上不同的过滤器。如果过滤器的数量受到限制，你可以简单的为每个不同的过滤器使用不同的Redis列表。毕竟每个列表只有5000条项目，但Redis却能够使用非常少的内存来处理几百万条项目。
```
3.排行榜相关

每次获得新得分时，我们用这样的代码
` ZADD leaderboard  <score>  <username> `

获取前100名高分用户`ZREVRANGE leaderboard 0 99`   从大到小 ZRANGE从小到大
获得特定用户的全球排名 `ZRANK leaderboard <username>`

4.按照用户投票和时间排序（还不是很明白）
  score = points / time^alpha 
  因此用户的投票会相应的把新闻挖出来，但时间会按照一定的指数将新闻埋下去。


5.处理过期项目(具体应用场景？)


6.计数

7.特定时间内的特定项目

8、实时分析正在发生的情况，用于数据统计与防止垃圾邮件等

9、Pub/Sub

10、队列

11、缓存

# 一些概念
Redis是单线程的

# 参考
[Redis应用场景](http://blog.csdn.net/hguisu/article/details/8836819)
[Redis官方文档](https://redis.io/documentation)
[Redis中文命令文档](http://www.redis.cn/commands.html)