# atomic LoadInt64 / LoadInt32



在阅读 bytebufferpool 和 workerpool 代码的时候经常看到`atomic.LoadUInt64` , `atomic.LoadInt32` , `atomic.StoreInt32` 等函数调用，产生了一个疑问 : 为什么整数操作要用到原子包里的东西



一番搜寻后，得到一个答案是64位的操作在32位上并不是由一个指令完成的，而是分成前32位和后32位两部分操作完成，因此整个操作没有原子性保证


**那要如何在32位机器上实现64位的原子操作呢？ 先分析一下可能出现的情况**

```
1. 执行完第一部分后，上下文切换到其他线程，其他线程读取该值，导致问题
2. 多cpu，多核，有cpu cache的情况下，单个指令有硬件的缓存一致性协议保证，多个指令的话无法保证 ， 也导致问题
3. 编译器对代码进行优化，打乱顺序，导致问题
```

问题1是可见性的问题，读取同样也要加内存屏障， 这样即使发生了线程切换， 另一个线程也不会看到赋值到一半的int64

问题2是原子性问题， 使用硬件提供的内存屏障指令，被内存屏障指令包围的操作只有在全部完成后才写入内存

问题3是有序性问题， 也由内存屏障提供， 阻止编译器,cpu的执行优化


## 如何查看 go 汇编代码

- 样例代码
```
package main

const x int64 = 1 + 1<<33
func main() {
	var i=x
	_ = i
}
```

- gdb

```
go build -gcflags "-N -l" -o gdb_sandbox main.go
gdb gdb_sandbox

进入 gdb 后
layout split
b main
run
```

- go tool compile / objdump
```
// objdump -gnu 好像没这个选项
//GOARCH=386 go tool compile -N -l main.go
//GOARCH=386 go tool objdump  main.o

//GOARCH=amd64 go tool compile -N -l main.go
//GOARCH=amd64 go tool objdump  main.o
```

- 64位和 32 位 int64 汇编代码对比 (macos-darwin)

```
 32 位:
 
 main.go:5             0x2ec                   c7042401000000          MOVL $0x1, 0(SP)        
 main.go:5             0x2f3                   c744240402000000        MOVL $0x2, 0x4(SP)
 
 64 位:
 main.go:5             0x2d4                   48b80100000002000000    MOVQ $0x200000001, AX

```


- LoadInt64 源码

runtime/internal/atomic/atomic_amd64.go
```
func Load64(ptr *uint64) uint64 {
	return *ptr
}

64位下直接取值

```

runtime/internal/atomic/asm_386.s
```
// uint64 atomicload64(uint64 volatile* addr);
TEXT runtime∕internal∕atomic·Load64(SB), NOSPLIT, $0-12
	MOVL	ptr+0(FP), AX
	TESTL	$7, AX
	JZ	2(PC)
	MOVL	0, AX // crash with nil ptr deref
	MOVQ	(AX), M0
	MOVQ	M0, ret+4(FP)
	EMMS
	RET

猜测: volatile 告知编译器不做优化 , NOSPLIT 和 EMMS 应该是内存屏障?    
```




## 为什么有32位的原子操作

猜测是给 16 位架构使用的, 开始尝试验证


GOARCH可以指定机器架构交叉编译，先看看go提供了哪些平台编译支持 , 并没有看到16位的参数

```
darwin/386
darwin/amd64
darwin/arm
darwin/arm64
```

- 测试代码
```
package main


const x int32 = 2 + 1 << 30
func main() {
	var i  = x
	_ = i
}
```

- 64 位和 32 位 LoadInt32 源码 对比
```
runtime/internal/atomic/atomic_386.go
func Load(ptr *uint32) uint32 {
	return *ptr
}

runtime/internal/atomic/atomic_amd64.go
func Load(ptr *uint32) uint32 {
	return *ptr
}

```
和直接取值的编译结果没有区别


- 再看看arm64 的汇编

```

arm64 直接取值的编译结果
main.go:6             0x2f5                   d2800040                MOVD $2, R0             
main.go:6             0x2f9                   f2a80000                MOVK $(16384<<16), R0

Loadint32 源码:
// uint32 runtime∕internal∕atomic·Load(uint32 volatile* addr)
TEXT ·Load(SB),NOSPLIT,$0-12
	MOVD	ptr+0(FP), R0
	LDARW	(R0), R0
	MOVW	R0, ret+8(FP)
	RET

```
结论: arm64 平台的 32 位操作是不是原子操作,先操作前 16 位,再操作后 16 位





## 资料

- [内存屏障](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C)
- [GOARCH 参数](https://liqiang.io/post/view-all-GOOS-and-GOARCH-for-go-3c2beacc)
- [go 并发-12丨 atomic：要保证原子操作，一定要使用这几种方法]()
- [使用GDB调试Go语言](https://www.cnblogs.com/beyondblog/p/4423173.html)
- [为什么会有atomic.LoadInt32](https://blog.huoding.com/2021/10/08/958)



## 后续

- 既然 32 位执行 64 位操作不是原子的, 为什么不在编译的时候自动加内存屏障呢 , 而是要自己调用 atomic , 猜测是因为内存屏障开销大?

- 所以这些流行的库为了提供给更多不同平台的机器使用，使用了 int 原子操作 ?

- atomic 包 ,主要提供了三种接口, RMW (read-modify-write ), 加载 , 存储 , 记住传的是指针