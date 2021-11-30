
Make this modification, boot xv6, and type echo hi to the shell. You should see something like this:

$ echo hi
usertrap(): unexpected scause 0x000000000000000f pid=3
sepc=0x00000000000012a6 stval=0x0000000000004008
panic: uvmunmap: not mapped
QEMU: TerminatedThe "usertrap(): ..." message is from the user trap handler in trap.c; it has caught an exception that it does not know how to handle. Make sure you understand why this page fault occurs. The "stval=0x0..04008" indicates that the virtual address that caused the page fault is 0x4008.



https://mit-public-courses-cn-translatio.gitbook.io/mit6-s081/lec08-page-faults-frans/8.2-lazy-page-allocation
sepc 是什么  , 发生错误时的用户代码 pc
stval 是什么 , 操作的虚拟地址
scause 有哪些值

为什么 echo 要用到堆

sh.asm

hp->s.size = nu;
12a6:	01652423          	sw	s6,8(a0)

shell 里调用了 malloc -> sbrk -> sys_sbrk  来执行命令 (存放 cmd 结构)






init: starting sh
usertrap(): unexpected scause 0x000000000000000c pid=17
sepc=0x0000000000000364 stval=0x0000000000000364
init: starting sh
usertrap(): unexpected scause 0x000000000000000c pid=18
sepc=0x0000000000000364 stval=0x0000000000000364
init: starting sh
usertrap(): unexpected scause 0x000000000QEMU: Terminated




test lazy unmap: OK
running test out of memory
panic: walk
QEMU: Terminated


用 gdb , 打日志 , 看 backtrace 查

addr 不是 uint64 导致 溢出3

// https://pdos.csail.mit.edu/6.S081/2020/labs/traps.html

running test out of memory
0x000000008000062c
0x0000000080001086
0x0000000080001198
0x0000000080001718
0x000000008000293a
0x0000000000000a94
panic: walk
QEMU: Terminated



if(va >= MAXVA) {
printf("%d:%d",va,MAXVA);
panic("walk");
}


running test out of memory
-2147471360:00x000000008000062c
0x000000008000109a
0x00000000800011ac
0x000000008000172c
0x000000008000294e
0x0000000000000a94
panic: walk
QEMU: Terminated

负数 , 什么东西溢出了
-7fffd000:00x000000008000062c


用之前 lab 写的 backtrace 程序 + 符号表 追踪栈
(base) ➜  xv6-labs-2020 git:(lazy) ✗ riscv64-unknown-elf-addr2line -e kernel/kernel
0x000000008000062c
0x0000000080001086
0x0000000080001198
0x0000000080001718
0x000000008000293a
0x0000000000000a94/Users/lin/s081/xv6-labs-2020/kernel/printf.c:123
/Users/lin/s081/xv6-labs-2020/kernel/vm.c:83
/Users/lin/s081/xv6-labs-2020/kernel/vm.c:166
/Users/lin/s081/xv6-labs-2020/kernel/vm.c:481
/Users/lin/s081/xv6-labs-2020/kernel/trap.c:72


0x00000000800010bc
0x00000000800011ce
0x000000008000174e
0x0000000080002970
0x0000000000000a94/Users/lin/s081/xv6-labs-2020/kernel/vm.c:86
/Users/lin/s081/xv6-labs-2020/kernel/vm.c:169
/Users/lin/s081/xv6-labs-2020/kernel/vm.c:484
/Users/lin/s081/xv6-labs-2020/kernel/trap.c:72


c 弱类型的坑 , xv6 的 bug

测试代码的逻辑:

panic("walk") ->
va >= MAXVA ->
发现 printf 没实现打印64位 , 照着改, 写了个printlong
打印 va 和 MAXVA 和 bt
这么说是 va 超过 MAXVA 了? 应该在 malloc -> sbrk 的地方拦截
加上了, 还是没用? 为啥 , 打印一下 addr+n

发现前面 va 都是 uint64 的, 这里传进去的 addr 是 int , 而且刚开始打印%d的时候是负数
想起 c 是弱类型的, 被强制转换了!
查看代码提交记录, 发现 06 年 rsc 都是用 int 的,因为那时候64 位还不普及 , 然后 19 年 f 把返回值改成 uint64 , 但是这个 addr 没改!

c 弱类型好坑
用着 go 的思维在排查 , 忘记了 c 是弱类型

78003780
79003790
7a0037a0
7b0037b0
7c0037c0
7d0037d0
7e0037e0
7f0037f0
80003800
81003810
80003000:00x000000008000062c
0x00000000800010bc
0x00000000800011ce
0x000000008000174e
0x0000000080002970
0x0000000000000a78
panic: walk
QEMU 5.2.0 monitor - type 'help' for more information
这时候怎么看也是超过 MAXVA 了啊, 很奇怪, 这时候还没有怀疑 xv6 代码错了 ,  写了个demo 测试了一下, 还是一样, 突然发现 addr 和 n 的类型是 int
32位有符号啊 , 最后一位为 1 的话为负数了! 把 int 改成 uint64 , 泪目了,一天就这么过去了 

2046834592
2063611824
2080389056
2097166288
2113943520
2130720752
-2147469312
-2130692080
80003000:00x000000008000062c
0x00000000800010bc
0x00000000800011ce
0x000000008000174e
0x0000000080002970
0x0000000000000a78
panic: walk
QEMU: Terminated



1



总结:

有很多坏习惯 , 养成了就难改

每个函数都要加定义注释 , 说明函数是做什么用的, 参数, 返回什么

代码设计
函数接口设计 , 参数设计 , 定义 什么的

代码里的 if else 代码块 含义 设计 , 变量定义



xv6 int addr 的 bug , 地址应该是 uint64 ,

stval 是 被调用方不会还原的寄存器 , 要在一开始就存好

一个函数别做太多事, 还把 p->killed 写里面了,  内核里的 walkaddr 根本不需要 kill 进程 , 而且内核里是软件 walk 页表,根本没有触发 pgfault , 也就没有 stval , 还傻傻的去 stval 取


这两个地方还是不知道 < 和 >= 是怎么判断出来的

PGROUNDUP 是往高地址 ROUND 的

if (va < PGROUNDUP(ustack)) {
return 0;

    } else if (va >= p->sz) {
        return 0;
    } else {


    }


    使用某个函数的时候看其他地方是怎么调用的, 看看函数接口定义 , 要做好错误处理



    不要用 ide 的 refactor 功能随便把某块代码封装成函数 , 如果要封装得先定义好函数接口 , 清楚函数是做什么用的


    lazy alloc , walk 页表的可能情况 , 没有 pte ( pte 为 0) , 或 pte_v 为 0, pte_u 为 0

    修改某个函数时也要清楚函数的定义, 参数 , 返回值定义 , 理解函数的作用 , 避免把函数语义给篡改了

    找本书或课程看看 , 关于代码设计, 维护的 , 坏习惯

    列出所有的坏习惯, 写代码前看看

    学习 lazy-debug 的写法 : 函数说明清除 , debug 日志清楚

crucial check
t is easy for a kernel developer to omit a crucial check, and real-world kernels have a long
history of missing checks whose absence can be exploited by user programs to obtain kernel priv-
ileges. It is likely that xv6 doesn’t do a complete job of validating user-level data supplied to the
kernel, which a malicious user program might be able to exploit to circumvent xv6’s isolation

增量式地改进是一种管理复杂度的强大工具。正如Polya在数学问题求解中所建议的那样——理解问题、形成计划、执行计划，而后再回顾你的做法。

找bug, 调试技巧 (问题排查) , 有固定的套路 或检查顺序吗 , 如何节省时间

xv6 vmprint , backtrace , gdb , printf , 







panic 后还会继续跑 其他进程

0x0000000080002966
0x0000000000000a78
panic: walk
asd!0x0000000080002d94
0x0000000080002c44
0x0000000080002906
0x0000000000000030
asd!0x0000000080002d94
0x0000000080002c44
0x0000000080002906
0x0000000000000030
asd!0x0000000080002d94
0x0000000080002c44
