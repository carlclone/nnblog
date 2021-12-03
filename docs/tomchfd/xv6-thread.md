(base) ➜  notxv6 git:(thread) ✗ ./ph 1            
100000 puts, 6.422 seconds, 15572 puts/second
0: 0 keys missing
100000 gets, 6.352 seconds, 15742 gets/second
(base) ➜  notxv6 git:(thread) ✗ ./ph q
[1]    2921 floating point exception  ./ph q
(base) ➜  notxv6 git:(thread) ✗ ./ph 2
100000 puts, 3.120 seconds, 32053 puts/second
1: 214 keys missing
0: 214 keys missing
200000 gets, 6.216 seconds, 32174 gets/second
(base) ➜  notxv6 git:(thread) ✗ ./ph 2
100000 puts, 3.035 seconds, 32948 puts/second
0: 154 keys missing
1: 154 keys missing
200000 gets, 5.970 seconds, 33502 gets/second
(base) ➜  notxv6 git:(thread) ✗ make ph
cc     ph.c   -o ph
(base) ➜  notxv6 git:(thread) ✗ ./ph 2
100000 puts, 7.484 seconds, 13362 puts/second
1: 0 keys missing
0: 0 keys missing
200000 gets, 6.244 seconds, 32033 gets/second

(base) ➜  notxv6 git:(thread) ✗ ./ph 2
100000 puts, 3.587 seconds, 27877 puts/second
1: 0 keys missing
0: 0 keys missing
200000 gets, 6.092 seconds, 32831 gets/second




后续问题 , 关于 lost wakeup , 还有为什么要在 sleep wakeup 之间 hold 住锁

看那篇 go 并发的课

xv6 chapter 7 有很多没看明白的地方 , 虽然这个 lab 很简单 , 
但是 thread 内核调度的实现很复杂 , 
还有很多同步问题 , 不变式如何保证

如何理解同步原语这个词? 我的理解是在并发,并行执行的情况下, 保证程序的执行结果和同步执行一样的工具

site:stackoverflow.com {keyword}
site:github.io {keyword}

原子型的
lock
atomic


协同型的原语
多个等一个
一个等多个

条件等待 , sleep/wake , waitgroup , 或组合多个同步原语


xv6 里实现的

spinlock (spin + 原子操作)
sleeplock (sleep / wake 实现)

xv6 和 linux 里 sleep wake 各自的实现 和区别