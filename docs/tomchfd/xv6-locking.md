有哪些可能会交错执行 , 访问的数据

proc 列表
kmem 


并发一词指的是由于多处理器并行、线程切换或中断，导致多个指令流交错的情况


竞争条件是一种内存位置被并发访问的情况，并且至少有一次访问是写操作





lab hint 里有一些可能的 bug , 可以写到文章里


锁优化 , 比如之前那个kfree 的 memset 拆分出来

还有 freelist 分 bucket 锁

xv6 chapter 8 很重要

不变式,  原子 , 序列化

使用锁的一个困难部分是决定使用多少个锁，以及每个锁应该保护哪些数据和不变量。有几个基本原则。首先，当一个CPU可以写一个变量的同时，另一个CPU可以读或写它的时候，应该使用锁来防止这两个操作重叠。第二，记住锁是保护不变性的：如果一个不变性涉及多个内存位置，通常所有这些位置都需要由一个锁来保护，以确保不变性得到维护

锁的颗粒度决定需要由性能测量和复杂性考虑来驱动。在随后的章节中解释xv6的每个部分时，他们将提到xv6使用锁来处理并发的例子。

make lookup in the buffer cache lock-free. Hint: use gcc's __sync_* functions. How do you convince yourself that your implementation is correct?

如何convince yourself 实现的正确性


锁和中断

一些xv6自旋锁保护线程和中断处理程序都使用的数据。例如，clockintr定时器中断处理程序可能在内核线程读取sys_sleep（kernel/sysproc.c:64）中的ticks（kernel/trap.c:163）的同时增加ticks。锁tickslock将这两个访问序列化。自旋锁和中断的交互引起了一个潜在的危险。假设sys_sleep持有tickslock，而它的CPU被一个定时器中断所打断，clockintr将试图获取tickslock，看到它被持有，并等待它被释放。在这种情况下，tickslock将永远不会被释放：只有sys_sleep可以释放它，但是sys_sleep不会继续运行，直到clockintr返回。因此，CPU会死锁，任何需要这两个锁的代码也会被冻结。为了避免这种情况，如果一个自旋锁被一个中断处理程序使用，CPU决不能在启用中断的情况下持有该锁。Xv6更保守：当一个CPU获得任何锁时，xv6总是禁用该CPU上的中断。中断仍然可以在其他CPU上发生，所以中断的获取可以等待线程释放自旋锁；只是不能在同一个CPU上发生。

CPU的排序规则被称为内存模型。例如，在这段推送的代码中，如果编译器或CPU将第4行对应的存储移到第4行的释放后的某一点，那将是一场灾难

为了告诉硬件和编译器不要进行这样的重新排序，xv6 在获取（kernel/spinlock.c:22）和释放（kernel/spinlock.c:47）中都使用了 __sync_synchronize() 。__sync_synchronize()是一个内存屏障：它告诉编译器和CPU不要在屏障上重新排序加载或存储。xv6的获取和释放屏障在几乎所有重要的情况下都会强制排序，因为xv6使用锁来访问共享数据。第9章讨论了一些例外情况

管多年来对并发原语和并行性的研究，用锁进行编程仍然具有挑战性。通常最好的办法是将锁隐藏在更高层次的结构中，如同步队列，尽管xv6没有这样做。如果你用锁编程，使用一个试图识别竞赛条件的工具是明智的，因为很容易错过一个需要锁的不变量。

sleeplock 不能用在哪些地方 , 看 chapter 8 


bcache

为什么使用时间戳之后就不用 bcache.lock 了呢

换成小粒度锁了? 为啥不继续用movetoHead 的方法


新思路: 可以用时间戳优化 get (不是这里的 bget) 时候的链表操作消耗 , 用原子的时间戳更新方式 , 使得 get 的速度更快