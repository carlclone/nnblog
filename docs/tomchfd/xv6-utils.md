# xv6 添加用户态程序


## sleep

描述符 0 1 2 分别是标准输入,标准输出,错误输出

支持命令行调用 sleep n , 简单判断输入参数 , 并调用 sleep syscall

- exit syscall

exit 也是个系统调用, 接收一个 int 参数,
https://mit-public-courses-cn-translatio.gitbook.io/mit6-s081/lec13-sleep-and-wakeup-robert/13.6-exit-systemcall
传入-1时 , 一般表示错误, 父进程据此判断
传入 0时 , 表示正常退出

```
$ make qemu
      ...
      init: starting sh
      $ sleep 10
      (nothing happens for a little while)
      $
```

## ping pong

使用管道实现 parent 和 child 互传

- linux管道的特性

pipe 的底层实现 , in-memory file ( buffer )

管道由两个文件描述符组成 , 0 表示 read 端,1 表示 write 端
子进程会继承管道, 但和父进程独立变化(close 不影响父进程)

- 管道 close 某个方向是必须的吗

父进程 close 写端, 子进程 close 读端, 形成单向的数据流动
同时也是关闭没必要的文件描述符,减少进程描述符表占用
并且如果父进程不 close , 无法检测到子进程已经关闭管道的描述符 , 造成继续写数据到管道并且阻塞 (因为还有一个读端 , 所以往管道里写数据不会报错 , 也不会触发 SIGPIPE)

https://www.youtube.com/watch?v=Mqb2dVRe0uo
https://stackoverflow.com/questions/24766013/is-it-really-necessary-to-close-the-unused-end-of-the-pipe-in-a-process/24768846

https://stackoverflow.com/questions/19265191/why-should-you-close-a-pipe-in-linux

https://www.cnblogs.com/nufangrensheng/p/3560130.html

```
$ make qemu
    ...
    init: starting sh
    $ pingpong
    4: received ping
    3: received pong
    $
```


## primes

用管道实现素数筛 prime sieve

- 什么是素数筛

快速筛出 2~n的所有素数的方法 , 复杂度是 O(nlognlogn) , 要比一个一个判断快得多

- 朴素实现 (埃氏筛)

2~16
每次看第一个,如果是素数,就划掉它的倍数,如 2, 划掉4,6,8,10,12,14,16

下一轮第一个是 3, 同上 

直到遍历结束


- 伪代码
使用一个 bool 数组表示划掉的数  isRemoved
  
遍历 i =  2 到 n
    如果 isRemoved , 跳过
    如果 isPrime
        遍历 i 到 n
            如果是 i 的倍数,划掉
定义 res 数组
 遍历i=2到 n
    如果划掉了,跳过
    否则加入 res 数组
返回 res
    
- 如何快速判断n是否素数
遍历 i = 2到 根号 n 的数
  如果n 能被整除,则不是  n%i==0
  否则是
- 为什么是根号 n  

    
参考资料 https://blog.csdn.net/linraise/article/details/16992817

- 用管道实现的思路
父进程作为一个 master,把要处理的素数集通过管道给到第一个子进程, 并等待接收子进程处理的结果,
第一个子进程筛第一遍,然后把剩余的交给子子进程继续筛,并等待结果 , 这样一直筛下去直到遍历结束,最后一个子进程一层一层返回给父进程
  
每个进程只负责判断自己的倍数, 如果不是则传给下一个


- 为什么要用管道实现

提高了并行度吗? 在这个例子里 , 没有 , 文中写的是提高了程序的清晰度 , 代码更简洁易懂

go 中的 channel , CSP 模型

CSP 提供了自然的抽象,使得某些程序更简单

https://swtch.com/~rsc/thread/

参考6824 russ cox 的演讲

```
 $ make qemu
    ...
    init: starting sh
    $ primes
    prime 2
    prime 3
    prime 5
    prime 7
    prime 11
    prime 13
    prime 17
    prime 19
    prime 23
    prime 29
    prime 31
    $
```

## find

实际上只需要实现完全匹配 , 但是在 grep 中看到了一个很好的 match 实现,直接用了

- 要实现的功能列表,案例

参考 ls.c 和 grep.c

任务: 看
- ls.c(X) 
- grep.c (rob pike大神级代码,参考这篇 https://www.cs.princeton.edu/courses/archive/spr09/cos333/beautiful.html)
- find.c X

- 文字梳理 grep.c 的逻辑 (待完成)

网上找代码实现讲解 , 看不懂

```
$ make qemu
    ...
    init: starting sh
    $ echo > b
    $ mkdir a
    $ echo > a/b
    $ find . b
    ./b
    ./a/b
    $ 
```

## xargs

- xargs 介绍

从标准输入读入每一行, 并对每一行执行命令 , 比如
echo "1\n2" | xargs echo line
输出
```
echo line 1
echo lin 2
```

任务: 读 xargs.c

网上找代码实现讲解 , 看不懂
测试: make qemu , sh < xargstest.sh

char 数组的指针是不可变的
char * 另外存

C K&R 书里有写 , 勉强记起来了

现在写法是一个一个字节读的, 会产生多次系统调用 , 优化成一次读多个字节的



## BUG汇总

BUG1:
```

init: starting sh
$ sleep 10
usertrap(): unexpected scause 0x000000000000000d pid=3
            sepc=0x0000000000000114 stval=0x0000000000003008
```

指令和寄存器列表:

寄存器:
- sepc

指令:
- scaue


sleep.asm
```
    return 0;
  24:	4501                	li	a0,0
  26:	60a2                	ld	ra,8(sp)
  28:	6402                	ld	s0,0(sp)
  2a:	0141                	addi	sp,sp,16
  2c:	8082                	ret
```


```
exit(0);
  40:	4501                	li	a0,0
  42:	00000097          	auipc	ra,0x0
  46:	292080e7          	jalr	658(ra) # 2d4 <exit>
```

BUG2:

primes 忘记关 pipe 导致阻塞





BUG3:
字符数组末尾没加 0 , 读取越界了