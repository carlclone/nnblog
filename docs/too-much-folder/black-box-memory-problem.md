# 服务器内存检查手册

## 内存指标

### 系统内存指标
- 已用内存
- 剩余内存
- 共享内存 (tmpfs,特殊缓存,参考 linux 内核实战)
- 可用内存 (包括剩余内存和可回收的缓存)
- 缓存/缓冲区内存 (缓存优化磁盘读,缓冲优化磁盘写)

查看工具:free -h , vmstat , cachestat,cachetop,sar, /proc/meminfo

### 进程内存指标

- 虚拟内存
- 常驻内存 (实际使用的物理内存,不包括 swap 和共享内存,RESIDENT)
- 共享内存 (和其他进程共享的内存,包括动态链接库和程序代码段)
- Swap 内存

查看工具: top,ps,memleak,pmap,/proc/pid/status , valgrind,pcstat,pidstat

缺页异常(page fault) , 次缺页异常(直接物理内存分配),主缺页异常(如磁盘 IO 介入,swap,比次慢很多)

### swap 指标
已用空间
剩余空间
swap in
swap out

查看工具: free -h , vmstat

sysstat 软件包安装 `sudo apt-get install -y sysstat`
bcc 软件包安装
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4052245BD4284CDD
echo "deb https://repo.iovisor.org/apt/$(lsb_release -cs) $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/iovisor.list
sudo apt-get update
sudo apt-get install bcc-tools libbcc-examples linux-headers-$(uname -r)
```

## 常用工具
top 当前内存占用情况
vmstat 3 每三秒输出一组系统数据,可以查看变化趋势,包括内存 , man vmstat 查看更多参数
/usr/share/bcc/tools/memleak -a -p $(pidof app) 内存泄漏检查, 能够看到进程的内存分配请求的大小和地址, 能够看到持续在分配请求的调用栈
-a 表示显示每个内存分配请求的大小和地址
-p 指定 pid
容器里运行的程序会看不到调用栈,需要把程序的二进制文件复制到宿主机对应的目录上 (参考 linux 性能优化实战 18)

## 内存泄漏复现

`docker run --name=app -itd feisky/app:mem-leak`
`docker logs app`

## 问题描述

Swap 满了
Page Cache 引发的问题 
其他类型的内存泄漏,如内核,SHR,VIRT

## 参考资料

linux 内核实战 内存篇
linux 优化实战 内存篇 18 21