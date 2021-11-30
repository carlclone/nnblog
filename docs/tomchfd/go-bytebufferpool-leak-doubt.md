# go bytebufferpool 错误使用一例

写新需求的时候发现一个远古代码, 使用 `bytebufferpool` 对象池库时只取出不放回, 同事觉得会造成内存泄漏, 持着怀疑态度做了个实验

## 复现

写了个 demo 尝试复现 , 开启 pprof 采集数据

| 指标                 | 放回  | 不放回        |
| -------------------- | ----- | ------------- |
| mac 监控实际内存占用 | 8MB   | 46MB          |
| inuse_space          | 512KB | 512KB         |
| alloc_space          | 512KB | 49GB 持续增长 |
| cpu 使用             | 100%  | 200%          |


&nbsp;


结果是没有发生内存泄漏, 但是会造成 cpu 巨大的 gc 压力 , 实际占用内存也偏大 , 可能是因为还没来得及回收


## 分析源码

**Get 的执行过程**

- byte.Buffer 在 pool 内部是链表 的一个节点 ,  取出的时候执行 `popHead` 从链表中去除, 于是在 pool 内部没有引用了 
- 在外部函数执行栈结束后也没有引用了
- 这种类型的堆变量会在go 定期的 gc 中被清除掉
- 但是只 get 不 put 的话,就相当于没有使用到"池"的功能了, 每次都是重新创建销毁 , 所以要注意
- sync.pool 继续放回也有可能被 gc 回收掉 (pool 里实现的机制 ,注册了 poolCleanup 函数到 runtime 的 gc 里) , 因为 sync.pool 没有最大容量限制,都可以 put

## 其他使用上的坑

### byte.Buffer 扩容 造成的内存泄漏

既然这样使用不会造成内存泄漏,那是不是 pool 就没有内存泄漏问题了?

查阅资料的过程中发现go 内置的 sync.pool 会有另外一种导致内存泄漏的问题 , 当从 pool 中取出 buffer 的时候还只是默认大小的 buffer , 但是在使用期间可能会造成 buffer 的扩容, 放回后 buffer 也依然是扩容后的大小, 如果扩容发生的次数特别多就会造成大量内存浪费 , 而且因为 pool 的回收机制, 这些大的 buffer 可能不被回收,  也是某种内存泄漏

**复现**

参考 go 官方库 [issue #23199](https://github.com/golang/go/issues/23199)

**解决方案**

在放回 pool 之前判断 , 如果大小大于某个值就直接丢弃

- encoding , json  , putEncodeState 函数
```
const maxSize = 1 << 16 //64KiB
if cap > maxSize {
    return
}
```

- fmt package, free 函数
```
if cap(p.buf) > 64<<10 {  65536字节 = 64KB
    return
}
```

- bytebufferpool 里的处理

```
// maxSize 的默认值为32MB 
minBitSize = 6 // 2**6=64 is a CPU cache line size
steps      = 20

minSize = 1 << minBitSize
maxSize = 1 << (minBitSize + steps - 1)     1<<25 = 33554432字节 = 32MB

func (p *Pool) Put(b *ByteBuffer) {
	idx := index(len(b.B))

	if atomic.AddUint64(&p.calls[idx], 1) > calibrateCallsThreshold {
		p.calibrate()
	}
    //如果小于 maxSize则放入 pool 中,否则忽略(丢弃)
	maxSize := int(atomic.LoadUint64(&p.maxSize))
	if maxSize == 0 || cap(b.B) <= maxSize {  //cap 返回 buffer 长度,单位字节
		b.Reset()
		p.pool.Put(b)
	}
}


```


### 内存浪费

除了扩容造成的内存泄漏(浪费)之外 , 还有使用大小不合适的 buffer 造成的浪费

比如池子里都是4KB 的 buffer , 但实际使用只需要用到 512 Byte



## 参考资料

- go 并发 10 - 性能提升大杀器
- go 语言项目实战 38 - 分析 go 代码性能 
- [golang的对象池sync.pool源码解读](https://zhuanlan.zhihu.com/p/99710992)
- valyala 性能挖掘达人 , fasthttp , bytebufferpool



## 测试 demo

- 使用 net/http/pprof 获取性能数据 (支持 inuse_space 查看)
```
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof"
	"pkg/mod/github.com/valyala/bytebufferpool"
)


func main() {
	go func() {
		log.Println(http.ListenAndServe("localhost:6060", nil))
	}()

	fakeLogByte:=[]byte{}
	for i:=0;i<10000;i++ {
		fakeLogByte=append(fakeLogByte,'l','o','g')
	}

	for {
		RequestLog(fakeLogByte)
	}

}


func RequestLog(fakeLogByte []byte) {
	bb := bytebufferpool.Get()
	bb.Write(fakeLogByte)
	//bytebufferpool.Put(bb)
}
```

- 通过命令行生成性能数据文件 (没有 inuse_space 数据)
```
//bytebuffer_pool_test.go

package test

import (
	_ "net/http/pprof"
	"pkg/mod/github.com/valyala/bytebufferpool"
	"testing"
)

func TestByteBufferPool(t *testing.T) {
	fakeLogByte:=[]byte{}
	for i:=0;i<10000;i++ {
		fakeLogByte=append(fakeLogByte,'l','o','g')
	}

	for i:=0;i<100000;i++  {
		RequestLog(fakeLogByte)
	}
	//time.Sleep(10*time.Hour)
}

func RequestLog(fakeLogByte []byte) {
	bb := bytebufferpool.Get()
	bb.Write(fakeLogByte)
	bytebufferpool.Put(bb)
}

go test -v bytebuffer_pool_test.go  -bench='.*' -cpuprofile cpu_put.out -memprofile mem_put.out
```

- 通过代码生成性能数据文件
```
func main() {
	cpuOut,_:=os.Create("cpu.out")
	defer cpuOut.Close()
	pprof.StartCPUProfile(cpuOut)
	defer pprof.StopCPUProfile()

	memOut,_:=os.Create("mem.out")
	defer memOut.Close()
	defer pprof.WriteHeapProfile(memOut)
	
	fakeLogByte:=[]byte{}
	for i:=0;i<10000;i++ {
		fakeLogByte=append(fakeLogByte,'l','o','g')
	}

	for i:=0;i<100000;i++  {
		RequestLog(fakeLogByte)
	}
	//time.Sleep(10*time.Hour)
}

go run bytebuffer_pool.go
```





## 后续

尝试跟踪调试 gc 的过程

