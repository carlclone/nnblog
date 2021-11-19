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
	calls       [steps]uint64   //用于统计不同容量范围的 buffer 的获取次数 , 如 step 0 存放 0-64 字节的 buffer 获取次数
	calibrating uint64 // 校准中的标志位, 不能执行 put 操作

	defaultSize uint64 // 当pool 中没有buffer 时,初始化的默认大小 , 会根据统计结果动态变化
	maxSize     uint64 // 当 buffer 放回时的size 判断, 避免 buffer > maxSize 造成内存泄漏

	pool sync.Pool // 底层还是使用 sync.Pool 对象池
}

```

结构和使用都非常简单, 实例化 Pool 就行

## Get 主要流程
先看看主体的流程 , 比较短 , 不画脑图了

- 尝试从 sync.Pool 中获取 ,获取不到则创建一个新的 ByteBuffer ,size 为 defaultSize

```
func (p *Pool) Get() *ByteBuffer {
	v := p.pool.Get()
	if v != nil {
		return v.(*ByteBuffer)
	}
	return &ByteBuffer{
		B: make([]byte, 0, atomic.LoadUint64(&p.defaultSize)),
	}
}
```

## Put 主要流程

- 先获取该 buffer 所在的区间下标 , 该区间调用次数+1 (统计逻辑)
- 然后判断该区间是否触发了校准阈值, 如果是, 开始校准 (校准逻辑写在下面的细节优化里)
- 如果未触发,判断该 buffer 的 cap 是否超过 maxSize, 超过则丢弃
- 否则 reset 切片的 len为 0 , `b.B = b.B[:0]` , 并且放入池子中

```
func (p *Pool) Put(b *ByteBuffer) {
	idx := index(len(b.B))

	if atomic.AddUint64(&p.calls[idx], 1) > calibrateCallsThreshold {
		p.calibrate()
	}

	maxSize := int(atomic.LoadUint64(&p.maxSize))
	if maxSize == 0 || cap(b.B) <= maxSize {
		b.Reset()
		p.pool.Put(b)
	}
}
```

## 细节优化

有很多细节上的优化, 算是复习了一下之前学到的知识点

### cpu cache line

这里使用了最小 64 (2^6次方) 字节并已他为倍数设置了 bytebuffer 的minSize 和 maxSize
```
    minBitSize = 6 // 2**6=64 is a CPU cache line size
    steps      = 20

	minSize = 1 << minBitSize
	maxSize = 1 << (minBitSize + steps - 1)
```


Q: 为什么要用 64 字节? 

因为 cpu cache 从内存加载到各级 cache 是以 64 字节的 cache line 为单位的 , 使用 64 字节可以避免最坏情况下多次访问内存 , 比如 50 字节 (没有64 字节对齐)

Q: 之前有看过说 go 编译器会做字节对齐, 那这里还需要额外对齐吗?

需要,因为编译器的对齐是 4 或 8 字节对齐

资料:

- [计算机组成原理- 高速缓存（上）：“4毫秒”究竟值多少钱？](https://time.geekbang.org/column/article/107477)
- [系统性能调优必知必会-01 | CPU缓存：怎样写代码能够让CPU执行得更快？]()
- [go 中的内存对齐](https://juejin.cn/post/6975177721097748516)


### 基于统计的阈值校准

- 当某个 cap 范围内的调用次数超过 42000 次时 , 会触发校准操作 , 重新计算 defaultSize  和 maxSize , 并重新统计
- defaultSize 用于当池子中没有可用 buffer 时 , 创建的新 buffer 的size
- maxSize 用于限制放回的 buffer,如果超过 maxSize 则丢弃

- 统计分为 20 个范围(0-64kB,64-128kB .... ) , 计算buffer 放回时,落在每个cap范围的buffer数量, 并做排序
- defaultSize 取命中次数最多 的 buffer size , 减少扩容造成的性能消耗
- maxSize 取从大到小排序后前百分之 95 的命中中 最大的 size , 保证百分之 95 的放回操作都不会被丢弃 (缓解 gc)

```
func (p *Pool) calibrate() {
	if !atomic.CompareAndSwapUint64(&p.calibrating, 0, 1) {
		return
	}

	a := make(callSizes, 0, steps)
	var callsSum uint64
	for i := uint64(0); i < steps; i++ {
		calls := atomic.SwapUint64(&p.calls[i], 0)
		callsSum += calls
		a = append(a, callSize{
			calls: calls,
			size:  minSize << i,
		})
	}
	sort.Sort(a)

	defaultSize := a[0].size
	maxSize := defaultSize

	maxSum := uint64(float64(callsSum) * maxPercentile)
	callsSum = 0
	for i := 0; i < steps; i++ {
		if callsSum > maxSum {
			break
		}
		callsSum += a[i].calls
		size := a[i].size
		if size > maxSize {
			maxSize = size
		}
	}

	atomic.StoreUint64(&p.defaultSize, defaultSize)
	atomic.StoreUint64(&p.maxSize, maxSize)

	atomic.StoreUint64(&p.calibrating, 0)
}
```

### 原子操作代替锁

类似这种 flag 标志位的, 使用原子操作开销要比锁性能好 , 这里的标志位是 calibrating (校准中)
```
if !atomic.CompareAndSwapUint64(&p.calibrating, 0, 1) {
		return
	}
```


## 其他

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

## 性能测试

这里有个使用 go benchmark 对不同的 buffer pool 的对比, bytebufferpool 的性能还是很好的 , [buffer pool bench](https://github.com/omgnull/go-benchmark)

```
BenchmarkGenericBuf-8                      1000000    1741 ns/op     2624 B/op    4 allocs/op
BenchmarkGenericStackBuf-8                 1000000    1695 ns/op     2624 B/op    4 allocs/op
BenchmarkAllocBuf-8                        1000000    1770 ns/op     2624 B/op    4 allocs/op
BenchmarkSyncPoolBuf-8                    50000000      43.5 ns/op      0 B/op    0 allocs/op
BenchmarkBpoolPoolBuf-8                   10000000     177 ns/op        0 B/op    0 allocs/op
BenchmarkByteBufferPoolBuf-8              50000000      39.0 ns/op      0 B/op    0 allocs/op
BenchmarkEasyJsonBuffer-8                  3000000     503 ns/op      608 B/op    4 allocs/op
BenchmarkEasyJsonBuffer_OptimizedSize-8   20000000      85.8 ns/op     32 B/op    1 allocs/op
```

## 参考资料

- [wiki-内存屏障](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C)
- [go 项目开发实战 - 性能分析 , 测试]()
- [go benchmark实践与原理](https://zhuanlan.zhihu.com/p/80578541)
- [高性能字节池 - bytebufferpool 源码分析](https://maolonglong.tech/golang/bytebufferpool/)
