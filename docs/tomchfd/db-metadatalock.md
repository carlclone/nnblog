# 数据库 waiting for metadata lock 问题排查

## 问题描述

新上线一个功能,发现新增表字段的时候卡住不动了

## 排查过程

- `show processlist` 查看当前连接的线程信息, 发现一个处于 sleep 状态且执行时间很长的连接, 且有另一个处于`waiting for metadata lock` 状态的连接 , 确认有事务长时间未提交也未回滚
- 查阅得知 metadata lock 是修改表 meta 信息时加的锁,该锁的级别相当于表锁,猜测某个事务持有该表的部分锁未释放
- 尝试查阅innodb_trx表 `select * from information_schema.innodb_trx ` 获取 sleep 状态事务的最后执行语句,为 NULL,无法得知
- 尝试另一个方法获取执行语句,检查 performance_schema 是否开启,` show variables like 'performance_schema';` , 5.7版本默认开启
- 执行 `select * from performance_schema.events_statements_current` 通过 THREAD_ID 对应上, 查看 SQL_TEXT 一列,成功拿到执行语句,把范围缩小到财务业务
- 代码全局搜索执行语句相关业务,确认有一处错误未回滚的情况, 修复

## 反思

- 使用 defer 避免忘记回滚或提交 
```
var txErr error
defer func() {
    if txErr!=nil { //Rollback }
    else {//Commit }
}()  
```
  
- 使用 gorm 事务闭包
```
db.Transaction(func(tx *gorm.DB) error {
  // 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
  if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
    // 返回任何错误都会回滚事务
    return err
  }

  if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
    return err
  }

  // 返回 nil 提交事务
  return nil
})
```


## 参考资料
- [MySQL如何定位未提交事务执行的SQL语句](https://www.jianshu.com/p/be4965ed802e)
- [mariadb/mysql 官方文档](https://dev.mysql.com/doc/refman/5.6/en/performance-schema.html)
- [postgres 2016用户大会-抽丝剥茧之MySQL疑难杂症排查](https://my.oschina.net/yejr/blog/4622000)
- [MySQL性能突发事件问题排查技巧](https://blog.csdn.net/enmotech/article/details/86684632)
- imysql_wx 微信公众号
- [pg 开发者大会 ppt 等资料](http://postgres.cn/v2/news/typelist/1/%E4%BC%9A%E8%AE%AE%E8%B5%84%E6%96%99)
- [pg 开发者大会视频](https://search.bilibili.com/all?keyword=postgres%20%20%E5%A4%A7%E4%BC%9A)




## 数据库问题排查信息来源

- `show processlist` / `show full processlist` 查看当前连接/线程信息 , 包含线程 id,客户端 ip:port , 状态 , 连接时间

- `information_schema.processlist` 同上

- `information_schema.innodb_trx` 查看当前正在执行的事务信息

- `performance_schema.events_statements_current` 查看当前statement事件的信息,包含执行语句

- `show global variables` / `show variables like '%max_connections%'`  查看所有参数


## 后续

- information_schema和 performance_schema 的深入学习

- 数据库的关键参数设置

- 是否有对整个事务设置超时时间的参数 , 是否合理: 无法确定超时时间的上限 , 且容易掩盖问题, 摧毁事故现场(是否有日志)

- 通过 general log 也可以获得事务的 sql 语句