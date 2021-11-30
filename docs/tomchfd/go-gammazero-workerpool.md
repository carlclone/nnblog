协程池
workerpool
连接池

池子的几个基本构成 java 性能调优实战

探活
... 忘了

使用问题排查的写法重写几个源码阅读文章

描述现象, 问题



# 为什么我写的代码很乱, 维护起来困难 , bug 也多

抽象程度不够 , 比如那个击鼓传花的并发题 , 使用 token 令牌来抽象就显得很简单

singleflight 里的 call 抽象也很棒
画图, 拟真 , 抽象 , 面向对象


## 接口
New

Submit
Stop

SubmitWait
StopWait

## 写法学习

- 里层有 select 或者 for 的时候,如何 break 外层的 for ?

给代码块添加标签, break 标签

- 如何暂停 worker ?

给 worker 发一个带 context 的func任务, 阻塞在 ctx.Done , 想恢复就 cancel()

```
整个pool 是用事件循环的思路写的 , 唯一用到锁的地方是 stop
在Stopped()和 Pause()和 stop()里用到了
Stopped()和 stop()很好理解, 停止不是原子操作(持续一段时间) , 读取是否停止的操作不能同时执行
stop()和 Pause() ,  停止和暂停 , 功能有冲突点 , 两个都是会持续一段时间的操作 , 有重叠
因此暂停的期间不能停止, 反之亦然
但是暂停之后可以停止 , 因此"暂停"(阻塞)了的 goroutine 此时要被回收 , 所以要监听 stopSignal 停止 , 而不是阻塞在 ctx.Done()
停止之后就什么都不能做了
```


额外实现了一个 double end queue (支持无限扩容,使用也要小心,以后可以用在其他项目) , 而不是存在 channel 里

- stop 功能复杂度瞬间上来了


stop 需要做的几件事

1. 回收 goroutine
2. 把队列里还未完成的任务执行完

回关闭 task queue , dispatch 里监听到channel关闭时停止 LOOP, 开始回收 goroutine , 如果设置了 wait , 则会跑完队列里剩余的任务

3. 此时不能再提交新任务
实现: 直接 close taskqueue , 往里面继续发会发生什么来着? panic:send on closed channel ,  这里是不是该告知用户已经关了的错误信息啊
   
没必要, 用户自己关的自己保证

- 这里还有个 idle worker 关闭的小优化 

好像不是很必要,感觉也没占用多少? 不太了解

用到了 timer , 记得timer 也要关闭 , 不然会占用资源

关闭操作只能执行一次 用到了 sync.Once , 因为 close 已经 close 的 channel 会 panic