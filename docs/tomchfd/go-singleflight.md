# singleflight

在工作中用到了 singleflight, 想看看源码

## 如果是我来实现

先明确需求: 多个请求同时来, 缓存失效, 只有一个请求去获取数据, 其他等待 , 当第一个请求完成的时候, 唤醒所有其他请求


有没有能让其他协程等待某个信号, 然后全部一起唤醒的功能? 信号量

巧妙地利用变量引用 , 实现类似linux 的硬链接功能 
只有在全部引用消失才会被gc掉

抽象出一个 call 辅助对象 , 表示inflight 或已完成的请求
map 在真正使用时才实例化, lazy allocation

画个时序图
使用 flag + channel 实现一个简单版的
使用 atomic 操作保证 happens-before 执行顺序保证

waitgroup 可以是一个 goroutine 等待一组 , 也可以是一组 goroutine 等待一个