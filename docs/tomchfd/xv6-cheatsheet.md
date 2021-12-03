# xv6 速查表

## risc v 寄存器列表




## 常用命令

每次调试要 make clean 清除编译缓存

make grade 全部成绩

./grade-lab-util pingpong

make GRADEFLAGS=pingpong grade

查看汇编:

查看代码段,数据段:

## c 语法

神书: K&R The C Programming Language

char *argv[] , 可以通过 argv[下标] 来访问

int abc[2]  定义并开辟 size 为 2 的数组 , 可通过 abc[0]访问

struct stat st 定义名为 st 的 stat 结构体

static char buf[DIRSIZ+1] 静态变量, 生命周期为整个运行期 , buf 名可以作为数组指针



## xv6 function

stat 结构体

st_mode 文件权限和文件类型信息
st_ino   与该文件关联的inode
st_dev   保存文件的设备
st_uid   文件属主的UID号
st_gid   文件属主的GID号
st_atime 文件上一次被访问的时间
st_ctime 文件的权限、属主、组或内容上一次被修改的时间
st_mtime 文件的内容上一次被修改的时间。（和st_ctime的不同之处显而易见）
st_nlink  该文件上硬连接的个数

fstat(fd, &st) 通过描述符获取文件信息
stat(filename,&st)  通过文件名获取文件信息

fd = open(path,O_FLAGS) , fd<0异常

```
1）O_RDONLY：以只读方式打开；

2）O_WRONLY：以只写方式打开；

3）O_RDWR ：以读写方式打开。

　　另外，还有以下几种可选模式的组合（ 用按位或 || 来操作 ）：

4）O_APPEND：把写入数据追加在文件的末尾；

5）O_TRUNC：把文件长度设置为零，丢弃已有的内容；

6）O_CREAT：如果需要，就按照参数 mode 中给出的访问模式创建文件；
....
```

fprintf 打印 , 参数 (描述符, 字节数组)
atoi(int) 字节数组->int

int fd_arr[2]
pipe(fd_arr)  创建管道
close() 关闭描述符
fork()==0 子进程 , 负值错误 , 正值父进程
wait(整数指针) , 如 wait((int *)0) 等待所有子进程
sizeof(int) int 的字节

返回的字节数 = read(描述符,字节数组,要读的字节数) , 如果描述符是个目录(目录也是文件,指向一块包含其中文件信息的字节block) , 可以这样遍历读 while(read(fd, &de, sizeof(de)) == sizeof(de)){
write(描述符,字节数组,要写的字节数)

"sadas" 也是字节数组

printf(字符串) fprintf()的封装,写到标准输出
getpid() 获取进程 id
dup()
strcpm() , ==不能对比字符串,而是对比指针值
strlen() 获取字符串(以\0 结尾的)长度  (sizeof 求数据类型所占空间)
memmove(dest,src,bytesize) 复制内容 http://c.biancheng.net/cpp/html/156.html

memset(dst,'字节',前几个)
```
memset(str,'$',7);
This is string.h library function
$$$$$$$ string.h library function

```

strcpy(dst,src) 顾名思义

strchr(str,ch) 返回第一次出现 ch 的指针 (指向 ch)


# gdb

最近做 xv6 和看 go 汇编用到 gdb 调试, 整理一下用到的资料

## gdb 原理

## 案例-调试 atomic 包


## 案例-调试 xv6

make clean && make qemu-gdb
新开窗口
riscv64-unknown-elf-gdb kernel/kernel
进入 gdb
target remote localhost:25501
add-auto-load-safe-path /Users/lin/s081/xv6-labs-2020/.gdbinit 加到/User/lin/.gdbinit
加到 Makefile
```
gdb:
	riscv64-unknown-elf-gdb kernel/kernel
```
b exec
注意看另一个窗口输出
c
c
c

## 调试 xv6 用户态程序

gdb 下:
file user/_ls
b main
c (要多 c 几下)

直接回车可以快速使用上一个命令

b *0x0 在某个地址下断点?

`系统调用detected，编号为7，查看kerne/syscall.h可知，编号为7的系统调用是SYS_EXEC。我们先把断点1删掉避免gdb因为断点崩溃掉，然后再exec上打断点：` 为什么会崩溃
delete 1  删除断点 1
b exec
查看所有断点 info breakpoints

系统调用 ecall

用户态没办法访问内核区的代码 , 只能 si 单步调试过 ecall , 进去内核态再看看能不能打断点?
```
0x0000000000000010 in ?? ()
=> 0x0000000000000010:  9d 48   li      a7,7
(gdb) 
0x0000000000000012 in ?? ()
=> 0x0000000000000012:  73 00 00 00     ecall
```

csrw    satp,t1 是什么

这个诡异的函数指针和函数调用，我们不能用n，因为很可能找不到对应的C代码，我们用si苟过去： 0x0000000080002814      130       ((void (*)(uint64,uint64))fn)(TRAPFRAME, satp);

那篇里的小 tips , 比如
使用*(array)@10，可以将指针array解释为数组，并打印后面的10个元素。
bt查看堆栈
后面会慢慢开始更新blog，主要更新自己上的一些公开课(已经完成了的6.824，正在肝的6.828和15-445)的一些笔记。后面会有开题和小论文，下学期留着刷leetcode和背面试八股，能发育的时间已经所剩不多了，加油吧。

(gdb) p cpus[0].proc
$1 = (struct proc *) 0x80325600
(gdb) p *cpus[0].proc
$2 = {lock = {locked = 1, name = 0x80008478 "process lock", cpu = 0x80012928 <cpus>, n = 267, nts = 0},
state = RUNNING, parent = 0x80325400, chan = 0x0, killed = 0, xstate = 0, pid = 2, kstack = 274877886464,
sz = 16384, pagetable = 0x87fa9000, tf = 0x80356000, context = {ra = 2147492184, sp = 274877890272,
s0 = 274877890320, s1 = 2150782464, s2 = 2147559640, s3 = 1, s4 = 2150782464, s5 = 4, s6 = 1, s7 = 0,
s8 = 0, s9 = 0, s10 = 0, s11 = 0}, ofile = {0x803253c0, 0x803253c0, 0x803253c0, 0x0 <repeats 13 times>},
cwd = 0x8001b548 <icache+32>, name = "sh", '\000' <repeats 13 times>}
(gdb)

gdb 是怎么知道在哪里下断点的? 有符号表
而代码编译后的结果一般是一个ELF(Executable Linkable Format)文件。ELF文件记录了我们代码中每个函数的虚拟地址，此外还会有一些其他有助于我们的信息。我们可以使用指令查看一下user/_sleep这个ELF文件的格式。新开一个终端，输入命令readelf -a user/_sleep

用 file 可以在 gdb 里加载符号表



## 案例-go 的函数调用栈

## 汇编语法
$  立即数 , 如$0x1 , 表示 16 进制的数字 1
() 访问内存

## 常见寄存器
sp stack pointer  永远指向 top of stack
bp base pointer  当前 stack frame 的基准地址
rax 通用寄存器

## 常用命令
- 打印 16 进制 p /x
- 打印变量 p
- 断点 b main.go:1 或函数名
- 开始调试 run
- 按汇编代码步进 si
- 下一条语句 n(next)
- 进入函数 s(step)
- 查看寄存器值 info reg pc

## 资料
[手把手带你写一门编程语言-15｜汇编语言学习（二）：熟悉X86汇编代码](https://time.geekbang.org/column/article/416809)

## 后续

观察 go 的函数调用栈汇编



gdb等调试手段要熟， 没法调试等着两眼抓瞎吧

熟悉链接库和编译那一套，Makefile什么的会用就行了。不熟悉编译的去编译个LFS（Linux from Scratch）就差不多了。

指针是进程内存地址空间上的一个地址
引用是高级语言里的一种语法糖 , 有点像 linux 的硬链接, 两个名词指向同一个容器,只有两个引用都没了,容器才会删除