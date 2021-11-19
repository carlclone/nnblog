# bytebufferpool

今天来阅读一下 valyala大神的 bytebufferpool 字节对象缓冲池 , 看看在sync.pool的基础上做了什么优化

[pool.go](https://github.com/valyala/bytebufferpool/blob/master/pool.go)

## 暴露的接口


```
func (p *Pool) Get() *ByteBuffer {
func (p *Pool) Put(b *ByteBuffer) {
```

和 sync.Pool 一样只提供了 Get 和 Put 接口 , ByteBuffer 是另外实现的类似 byte.Buffer 的结构,提供相同的接口和据说更好的性能 (找个时间压测一下)

## 主要结构体


```
type Pool struct {
```

结构和使用都非常简单, 实例化 Pool 就行

## Get 主要流程
先看看主体的流程 , 比较短 , 不画脑图了

- 尝试从 sync.Pool 中获取 ,获取不到则创建一个新的 ByteBuffer ,size 为 defaultSize

## Put 主要流程

- 先获取该 buffer 所在的区间下标 , 该区间调用次数+1 (统计逻辑)
- 然后判断该区间是否触发了校准阈值, 如果是, 开始校准 (校准逻辑写在下面的细节优化里)
- 如果未触发,判断该 buffer 的 cap 是否超过 maxSize, 超过则丢弃
- 否则 reset 切片的 len为 0 , `b.B` = b.B[:0] , 并且放入池子中

## 细节优化

有很多细节上的优化, 算是复习了一下之前学习的知识点

- cpu cache line
这里使用了最小 64 字节并已他为倍数, 为什么要用 64 位, 因为 cpu cache 从内存加载到各级 cache 是以 64 字节的 cache line 为单位的 , 使用 64 字节可以避免最坏情况下多次访问内存 , 比如 50 字节 (没有64 字节对齐)

[计算机组成原理- 高速缓存（上）：“4毫秒”究竟值多少钱？](https://time.geekbang.org/column/article/107477)
[系统性能调优必知必会-01 | CPU缓存：怎样写代码能够让CPU执行得更快？]()


- 基于统计的阈值校准

当某个 cap 范围内的调用次数超过 42000 次时 , 会触发校准操作 , 重新计算 defaultSize  和 maxSize , 并重新统计
defaultSize 用于当池子中没有可用 buffer 时 , 创建的新 buffer 的size
maxSize 用于限制放回的 buffer,如果超过 maxSize 则丢弃


分为 20 个范围, 计算put 回时,落在每个cap范围的buffer数量, 并做排序 , 
defaultSize 取命中次数最多的范围的 buffer size , 减少扩容造成的性能消耗
maxSize 取从大到小排序后前百分之 95 的命中中 最大的 size , 保证百分之 95 的放回操作都不会被丢弃

- 原子操作代替锁

类似这种 flag 标志位的, 使用原子操作开销要比锁性能好 , 这里的标志位是 calibrating (校准中)
```
if !atomic.CompareAndSwapUint64(&p.calibrating, 0, 1) {
		return
	}
```


## 其他

- 使用 atomic 包操作 64 位数值

如果不使用原子操作 , 可能会读到赋值了一半的值 (比如只赋值了前 32 位) , 无法预测会读到什么值

因为64 位数值的赋值和取值操作有可能不是原子的(比如 32 位下) , 并且可能乱序执行

atomic 包则保证了 64 位的操作是原子的,并且在多处理器,多核,有 cpu cache 的情况下 , 不会被乱序执行(内存屏障) 

单条指令的一致性有总线协议 和缓存一致性协议保障(硬件实现) , 多条指令则由内存屏障确保有序保障一致性

除了原子性 , 现在又多了可见性和有序性

https://blog.huoding.com/2021/10/08/958

- atomic 包

主要提供了三种接口, RMW (read-modify-write ), 加载 , 存储 , 记住传的是指针  

[go 并发-12丨 atomic：要保证原子操作，一定要使用这几种方法]()

- 没有分不同buffer 大小的池子

在 go 并发课中读到了这样的一种优化 , 但在这里没看到

但作者在注释里写了 , 希望由用户自行判断 buffer 的使用类型 , 不同使用类型用不同的 pool
```
// Distinct pools may be used for distinct types of byte buffers.
// Properly determined byte buffer types with their own pools may help reducing
// memory waste.
```

像 mysql 的连接池也是一样 , 耗时的操作也可以放到另外一个连接池去 , 减少对短时操作的影响

这里有另外一个库实现了 , 实现也不难,再这个 pool 的基础上再封装一层pool 的数组,不同大小的 buffer 从不同的 pool 中取 参考 go 并发 [pool 性能提升大杀器](file:///Volumes/COURSE/%E6%96%B0%E5%88%86%E7%B1%BB/Go%E5%92%8C%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E5%92%8C%E5%AD%98%E5%82%A8%E5%92%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/go%E5%B9%B6%E5%8F%91/10%E4%B8%A8%20Pool%EF%BC%9A%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%A4%A7%E6%9D%80%E5%99%A8.html)


## atomic 测试 , 汇编代码查看 , gdb 调试
很难测啊,算了
汇编代码可以看看 给出路径

go build之后
sudo gdb 程序名 或 gdb 程序名
run

list 和 layout next 继续 next 可以看汇编窗口

https://www.cnblogs.com/beyondblog/p/4423173.html

## 发现还有 32 位的 atomic 操作
在看 atomic 包的时候发现还有各种 32 位的操作, 64 位的可以理解(因为有 32 位的机器) , 难道还有16 位的机器用 go 吗?

尝试用 64 位和 32 位编译 , 发现汇编是一样的 ,

翻阅代码发现只是一个 doc.go , 但是同目录下有个 asm.S , 看来是用汇编写的, 原来 arm 平台的 int32 操作不是单条指令的 , 所以为了兼容多平台 , gammazero/workerpool 对 int32 也采用了 atomic 包

## 性能测试
## 和其他byte buffer pool库对比 , go 写性能测试代码 , 如何性能测试

库里面是怎么写不同库的性能测试代码的

https://maolonglong.tech/golang/bytebufferpool/

https://github.com/omgnull/go-benchmark
https://omgnull.github.io/go-benchmark/buffer/

https://zhuanlan.zhihu.com/p/80578541

go 项目开发实战 - 性能分析 , 测试

## 参考资料

[wiki-内存屏障](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C)