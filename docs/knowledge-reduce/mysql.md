---
title: 知识点的reduce-mysql
date: 2019-12-09
tags: 计算机基础
---

&nbsp;
<!-- more -->

# 关联查询 hasone底层是什么sql。配置表之间的关联关系 创建表entity

有拆分的也有 join 的

https://gin-gonic.com/zh-cn/docs/introduction/
https://gorm.io/zh_CN/docs/performance.html

https://blog.csdn.net/qq_28053177/article/details/82187821

https://solutionsreview.com/business-intelligence/the-best-apache-kafka-books-and-upcoming-titles-to-consider/

# join 和 where 哪个先执行, 和分成两条查哪个更好

取决于数据库的抽样统计结果和优化器的判断, 两者都差不多, 拆分查询对后期拆分业务, 扩展有好处 , join 减少一次网络请求

https://stackoverflow.com/questions/3960561/where-and-join-order-of-operation
高性能 mysql

# 为什么很少看到子查询

因为在 5.5 之前的版本, mysql 对子查询的执行计划优化是很差的(参考高性能 MySQL), 因此很多人都养成了习惯 , 使用 join 和拆分查询的方式,而不是子查询

# B-Tree 和B more Tree (4、b+树特性)

B more Tree产生的原因: 常规的树结构不能很好地支持"区间查询" (需要进行一次从根节点开始的中序遍历,时间复杂度O(n) , 因为树结构不支持从某个节点开始有序遍历)

和跳跃表(多级链表索引,每一级都定位到更小的范围,最后找到元素)很相似 , 但却是两个不一样的演化分支 

所有的数据都放在叶子节点 , 并串成链表 , 其他节点都是索引节点 , 不存数据

继承了树结构优势的同时支持了快速的区间查询

所以B+树更加适合在区间查询的情况，通常B+树用于数据库索引，而B树则常用于文件索引。

#  Optimistic Lock / Pessimistic Lock

前置知识 :加锁操作会有额外的CPU消耗

乐观锁假设发生冲突的情况很少 , 不对数据进行加锁操作 , 只在更新或者查询的时候检查这个时间段内是否被其他人修改 , 后续处理一般是重试
悲观锁则相反,使用数据的时候立即加锁,其他人只能等待锁释放后才能使用

InnoDB的RC和RR事务底层的MVCC实现就是一个乐观锁, 在正确的场景下使用乐观锁可以提高并发度

等等！乐观锁不就是汇编里的原子操作CAS ，先检查内存中的旧值，如果被修改了就更新失败


# 事务在InnoDB中的部分实现细节 vs Redis


# ACID

> WARNING : 和分布式中CAP的C区分开,不是一个概念

Atomicity 事务的语句要么全部执行要么全部回滚

Isolation 多个事务同时执行互不影响  (InnoDB提供了多种隔离级别,供用户在并发度和隔离性之间权衡后选择合适的级别 , 较常使用的是RC和RR , 使用MVCC实现)

Durability 持久性 , redolog 先顺序写磁盘 , 之后再写入正确的位置(要求随机IO,耗时) , 官方文档成为doublewrite buffer 

Consistency 定义最模糊的一个性质 , 但根据官方文档里的实现和Wikipedia:
保证数据库约束不被违反 (依据wiki的定义主要是这条)

(依据官方文档的实现:doublewrite buffer和crash recovery)
保证事务提交后,未来的事务一定能看到影响 (崩溃后重启,会从redolog中读取写入正确位置后再执行其他事务)
保证事务中的操作能正确,有效地执行

## 资料
https://dev.mysql.com/doc/refman/5.6/en/mysql-acid.html
https://en.wikipedia.org/wiki/ACID#Consistency
https://developpaper.com/question/how-to-understand-consistency-in-mysql/

# Clustered Index vs Non-Clustered Index

CI 的叶子节点保存的是整行的完整数据 , NCI的叶子节点只保存了CI的主键ID

如果从NCI查,并且不是覆盖索引,此时还需要返回CI中再查一次


# Unique Index vs Normal Index

UI每次维护的时候还需要检查唯一性 , 写性能损耗比NI的高
查性能的差异可以忽略不计,因此能用NI的时候用NI

# 不适合建立索引的场景

频繁写 , 频繁维护索引, 在非递增的情况下还容易产生页分裂
重复值过多 , 用"二分查找"的思维去理解, 例子:性别字段, 过滤掉一半之后依然需要遍历表一半的数据 (时间复杂度退化成O(n/2))
varchar字段建索引更容易产生页分裂 (未查证)

# 联合索引

可以理解为多维有序数组多层遍历,取符合的数据

当然实际上是Bmore Tree的区间查询过程, 先搜索树找到第一个符合条件的叶子节点,再从叶子节点链表向后遍历

5.5 之前只能匹配第一列,然后依个回主键索引再判断第二个条件

5.6 之后的索引下推功能,可以直接先排除掉不符合条件的数据,减少回主键索引查询的次数

最左前缀原则: 最左N个字段 or 字符串索引的最左M个字符

后续:测试+查高性能MySQL

参考:
mysql 45讲-第5讲


# 7. select * from t where a=? and b>? order by c limit 0,100 如何加索引

a b c
1 2 3
1 3 9
1 9 3
2 9 4
. . .
. . .

基于MySQL 5.6之后的索引下推规则分析

(a,b,c)的情况: 先筛选出3条数据,再判断b 

(b,a,c)的情况: 可能筛选出n多条的数据,再每一条判断a

这个案例下 (a,b,c) 更优

仔细分析一下这题为什么要这样加索引

加上c 减少filesort , a,b顺序可以用到联合索引的两列 , (b,a,c) 只能用到第一列索引, 因为在树搜索的时候会产生多个分支, 所以引擎选择从第一个元素向后遍历



