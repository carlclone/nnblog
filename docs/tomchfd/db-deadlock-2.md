# 复现不同索引之间的死锁

不同索引之间的锁也可能造成死锁,常见的如主键索引和辅助索引之间的锁循环等待

| 事务 A                                             | 事务 B                                      |
| -------------------------------------------------- | ------------------------------------------- |
| BEGIN                                              | BEGIN                                       |
| update order_record set status=10 where order_no=4 | update order_record set status=1 where id=4 |



| 事务 A                             | 事务 B                             |
| ---------------------------------- | ---------------------------------- |
| 获取 idx_order_status 辅助索引行锁 |                                    |
|                                    | 获取主键索引行锁                   |
| 获取主键索引行锁                   |                                    |
|                                    | 获取 idx_order_status 辅助索引行锁 |



## 复现

需要使用 go 协程进行并发模拟,2个协程,不停地执行这两个语句,并且要保持effected rows>0 , 否则似乎会被数据库优化器优化掉,并不加锁直接返回

```
package main

import (
	"fmt"
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
	"time"
)

func main() {
	db, err := gorm.Open("mysql", "root:root@(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local")
	if err!= nil{
		panic(err)
	}
	defer db.Close()

		go func() {
			for {
				err:=db.Exec("update order_record set status=1 where id=4").Error
				if err!=nil {
					fmt.Println(err.Error())
				}
			}
		}()
	
		go func() {
			for {
				err:=db.Exec("update order_record set status=10 where order_no=4").Error
				if err!=nil {
					fmt.Println(err.Error())
				}
			}
		}()

	time.Sleep(1*time.Hour)
}

```



## 排查

- show innodb engine status 查看日志

```
LATEST DETECTED DEADLOCK
------------------------
2021-11-13 12:28:59 0x7000099ff000
*** (1) TRANSACTION:
TRANSACTION 3416559, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1136, 3 row lock(s)
MySQL thread id 86892, OS thread handle 123145462673408, query id 1820252 localhost 127.0.0.1 root Searching rows for update
update order_record set status=10 where order_no=4
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3193 page no 3 n bits 72 index PRIMARY of table `test`.`order_record` trx id 3416559 lock_mode X locks rec but not gap waiting
Record lock, heap no 5 PHYSICAL RECORD: n_fields 6; compact format; info bits 0
 0: len 4; hex 80000004; asc     ;;
 1: len 6; hex 0000003421ee; asc    4! ;;
 2: len 7; hex 58000010440a70; asc X   D p;;
 3: len 4; hex 80000004; asc     ;;
 4: len 4; hex 80000001; asc     ;;
 5: SQL NULL;

*** (2) TRANSACTION:
TRANSACTION 3416558, ACTIVE 0 sec updating or deleting
mysql tables in use 1, locked 1
3 lock struct(s), heap size 1136, 2 row lock(s), undo log entries 1
MySQL thread id 86891, OS thread handle 123145463787520, query id 1820251 localhost 127.0.0.1 root updating
update order_record set status=1 where id=4
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 3193 page no 3 n bits 72 index PRIMARY of table `test`.`order_record` trx id 3416558 lock_mode X locks rec but not gap
Record lock, heap no 5 PHYSICAL RECORD: n_fields 6; compact format; info bits 0
 0: len 4; hex 80000004; asc     ;;
 1: len 6; hex 0000003421ee; asc    4! ;;
 2: len 7; hex 58000010440a70; asc X   D p;;
 3: len 4; hex 80000004; asc     ;;
 4: len 4; hex 80000001; asc     ;;
 5: SQL NULL;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3193 page no 4 n bits 80 index idx_order_status of table `test`.`order_record` trx id 3416558 lock_mode X locks rec but not gap waiting
Record lock, heap no 7 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000004; asc     ;;
 1: len 4; hex 8000000a; asc     ;;
 2: len 4; hex 80000004; asc     ;;

*** WE ROLL BACK TRANSACTION (1)
```

- 分析日志, 事务 1 等待 index PRIMARY  ,事务 2 持有 index PRIMARY  ,事务 2 等待 index idx_order_status , 事务 2 被回滚

# 后续

关于数据库优化器的策略,优化过程日志查看 CMU15-445 回顾