# go 并发读写 map 问题一例

## 问题描述

上线一个新功能,发现服务自动重启, QPS 也降的很厉害, 回滚后开始排查

## 排查过程

- 检查代码,猜测是由于类型转换没有判断第二个 ok 返回值`i:=str.(int)`, 不能转换导致 panic

排除,被转换的值是确定的整数字符串 , 并且 panic 错误会被 recovery 中间件捕获,不至于导致重启, 但这种写法也还是有问题

修正:
```
i,ok:=str.(int)
if !ok { return }
fmt.Println(i)
```

- 查看历史日志, 所有 goroutine 的调用栈都被打印出来了, 日志量特别多

- 发现一条日志: `concurrent map read and map write` , 并且是 fatal error,不能被 recover() 捕获

- 检查代码,得知框架对从配置中心获取的配置文件进行了缓存, 获取 config map 的返回值是同一个 map , 导致在外层修改时造成了并发 bug
- 这里的新需求不涉及也不适合对同一个 map 进行操作,排除加锁和其他线程安全 map 解决方案, 参考 gin 框架 `initQueryCache` 函数 , 使用`[]map[string]string(config)` , 强制转换生成一个新 map , 或使用 `copier` 拷贝

## 后续思考

- 有时候并发问题在生产中隐藏的很深, 并不像 demo 代码那么容易看出来

- 如果不是有特别的性能要求或者很大, 都尽量保证 immutable 的特性, 提高项目可维护性

- 想起了eloquent js 一书和函数式编程 , 不可变的特性,降低复杂度

- 崩溃时大量 goroutine 调用栈打印出来污染日志的问题有好的解决方法吗

## 参考资料

- [自研or借力：集成Gin替换已有核心](https://time.geekbang.org/column/article/423982)
- [清晰、可直接套用的Go编码规范](https://time.geekbang.org/column/article/385440)
- [mohuishou-go错误处理最佳实践](https://lailin.xyz/post/go-training-03.html)