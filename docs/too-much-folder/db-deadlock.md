# 间隙锁造成的死锁

可重复读隔离级别或更强级别, 加锁有可能加间隙锁 , 和插入意向锁形成死锁

## 复现

```
CREATE TABLE `order_record`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `order_no` int(11) DEFAULT NULL,
  `status` int(4) DEFAULT NULL,
  `create_date` datetime(0) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `idx_order_status`(`order_no`,`status`) USING BTREE
) ENGINE = InnoDB

# mysql> show variables like '%isolation%';
mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.01 sec)

mysql> set autocommit = 0;
Query OK, 0 rows affected (0.00 sec)

事务 1:
1 BEGIN;
2 SELECT id FROM `order_record` where `order_no`=4 for update;
5 INSERT INTO `order_record` (order_no,status,create_date) VALUES (4,1,'2019-01-01');
锁等待......
COMMIT; (未完成)

事务 2:
3 BEGIN;
4 SELECT id FROM `order_record` where `order_no`=5 for update;
6 INSERT INTO `order_record` (order_no,status,create_date) VALUES (5,1,'2019-01-01');
锁等待......
COMMIT; (未完成)
```

## 分析

- 由于 order_no 上的索引不是唯一索引, for update 会在索引上加排他锁, 并且隔离级别是REPEATABLE READ , 除了排他锁还会加间隙锁 , 因为此时表里没有任何记录 , 所以范围是 (-∞,+∞)
- 事务 2 也在索引上加了(-∞,+∞) 的间隙锁
- 事务 1 尝试获取 X insert intention lock (-∞,+∞) , 等待事务 2 的 (-∞,+∞) 的间隙锁
- 事务 2 尝试获取 X insert intention lock (-∞,+∞) , 等待事务 1 的 (-∞,+∞) 的间隙锁
- 死锁产生, 数据库开启了死锁检测, 回滚了事务 2

## 排查数据获取

- show engine innodb status 查看 innodb 日志,包含死锁检测日志

```
事务 1
2021-11-11 15:27:04 0x70000b5c6000
*** (1) TRANSACTION:
TRANSACTION 1141004, ACTIVE 10 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1136, 2 row lock(s), undo log entries 1
MySQL thread id 72, OS thread handle 123145493192704, query id 1380 localhost 127.0.0.1 root update
INSERT INTO `order_record` (order_no,status,create_date) VALUES (4,1,'2019-01-01')
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3193 page no 4 n bits 72 index idx_order_status of table `test`.`order_record` trx id 1141004 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;


事务 2
*** (2) TRANSACTION:
TRANSACTION 1141005, ACTIVE 7 sec inserting
mysql tables in use 1, locked 1
3 lock struct(s), heap size 1136, 2 row lock(s), undo log entries 1
MySQL thread id 70, OS thread handle 123145492914176, query id 1381 localhost 127.0.0.1 root update
INSERT INTO `order_record` (order_no,status,create_date) VALUES (5,1,'2019-01-01')
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 3193 page no 4 n bits 72 index idx_order_status of table `test`.`order_record` trx id 1141005 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3193 page no 4 n bits 72 index idx_order_status of table `test`.`order_record` trx id 1141005 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** WE ROLL BACK TRANSACTION (2)
```

- select * from innodb_lock_waits; 查看处于锁等待的事务
- select * from innodb_locks; 查看持有锁的事务


## 解决方案
- order_no 设为唯一索引
- 开启死锁检测  , show variables like '%deadlock%'; 检查死锁检测是否开启
- 设置 lock_wait_timeout,  show variables like '%timeout%'; 检查 lock_wait 超时时间
- gap lock 默认开启, innodb_locks_unsafe_for_binlog 为 disable

## 预防措施

- 避免长事务, 拆解事务
- 更新表尽量使用主键
- 使用 RC 隔离级别 (如果业务允许幻读和不可重复读) , show variables like '%isolation%'; 检查隔离级别
- 保持执行顺序一致
- 调小lockwait timeout , 避免大量事务等待消耗系统资源

## 死锁介绍

- 独占资源,有排他锁出现 (MVCC 解决一部分)
- 阻塞时不主动让步 (lockwait_timeout)
- 不可抢占 (强行回滚其中一个)
- 循环等待 (死锁检测,使用图算法检测环 | 执行顺序保证 )


## 间隙锁什么时候会加

- 间隙锁加在两个记录之间 , 如果某个方向没有记录则是无穷大 , 且插入意向锁也是间隙锁的一种 ([mysql 文档-Insert Intention Locks](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-insert-intention-locks)) 
- 跟间隙锁存在冲突关系的，是“往这个间隙中插入一个记录”这个操作。间隙锁之间都不存在冲突关系。
- 间隙锁的意义是提高执行效率


```
间隙锁的加锁规则
加锁规则里面，包含了两个“原则”、两个“优化”和一个“bug”

原则 1：加锁的基本单位是 next-key lock。希望你还记得，next-key lock 是前开后闭区间。
原则 2：查找过程中访问到的对象才会加锁。
优化 1：索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。
优化 2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。
一个 bug：唯一索引上的范围查询会访问到不满足条件的第一个值为止
```

数据库锁兼容性检查
![image-20191120140931373](/img/db-deadlock.png)

## 参考资料
- [36 | 记一次线上SQL死锁事故：如何避免死锁？](https://time.geekbang.org/column/article/117247)
- [mysql 文档-Insert Intention Locks](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-insert-intention-locks)
- [深入理解MySQL的间隙锁](https://www.jianshu.com/p/09fe0c700984)
- [意向锁的意义](https://www.jianshu.com/p/325a492a859b)
- [手把手教你分析Mysql死锁问题](https://zhuanlan.zhihu.com/p/130235093)
- [死锁条件](https://time.geekbang.org/column/article/141816)
