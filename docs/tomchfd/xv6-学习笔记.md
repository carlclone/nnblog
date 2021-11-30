2020.4.3 + 4.4

今天开始了 6s081 的课程学习, 虽然感觉是一门远超出我水平的课, 但还是很想学习 , 在这记录一下每次的学习, 看看能坚持多久

看的是 2020fall 的版本 , 今天在 macos 上把 qemu 的环境搭好了, 然后开始的第一个 lab 的阅读, 读完了 xv6 ref 的第一章 , 基本上和之前 csapp 的 shell lab 内容差不多, 难点集中在管道和文件描述符上 , 如 exec 只覆盖内存, 但保留文件描述符 , 使用 dup 重定向子进程的标准输入到管道中 , 管道设计叫做 CSP模型, 也是 go 的 channel 的起源, 简化了进程/线程之间通信和同步的复杂度


还有 the c programming language 的 4 , 5 章 , 对 c 指针的一些人类不友好的语法有了一些印象 ,  然后过了一遍书上的硬核例子, c 对我来说有点太 low-level 了, 写代码的时候要关注特别多细节  

lab 看到 primes 并发素数算法这里, 卡了两个小时, 查阅了一番资料后发现就是动态地添加上过滤器 filter , 然后用管道把过滤器连起来,每个自然数都流过过滤器 , 如果是素数,就从管道一直往回流 (有两个方向的管道)




# lab1 utils

实现user space 的程序



# lab2 sys call

要求实现系统调用的跟踪

trace 掩码 执行的程序

如 trace 32 grep hello , 跟踪 grep hello 执行过程中的 read 系统调用

系统调用的过程:



系统调用的参数存在哪里 , 寄存器还是 trapframe , 后面可能就知道了 , 为什么是存在这里


trampoline page 是什么


先熟读:
kalloc.c
main.c
memlayout.h
param.h
proc.c
proc.h
riscv.h
syscall.c
syscall.h
sysproc.c
vm.c