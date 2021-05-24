---
title: 知识点的reduce-问题排查
date: 2020-01-08
tags: 计算机基础
---

持续更新和记录

<!-- more -->

# 排查网络问题 支付超时问题 的流程

偶发超时,不能实时复现排查                                              

查看系统的网络统计数据                                     
使用 tcpdump 收集网络数据包,分析

                       
当前网络超时问题

当前网络缓慢问题



> linux性能优化实战 学习总结

- 善用 man page 查找命令和sys call 函数资料

# 学习方向指导图
![](/img/images/linux_perf_tools_1.png)
![](/img/images/linux_perf_tools_2.png)
![](/img/images/linux_perf_tools_3.png)

# ps命令

参考tldr常用

```
- List all running processes:
    ps aux

- List all running processes including the full command string:
    ps auxww

- Search for a process that matches a string:
    ps aux | grep string

- List all processes of the current user in extra full format:
    ps --user $(id -u) -F

- List all processes of the current user as a tree:
    ps --user $(id -u) f

- Get the parent pid of a process:
    ps -o ppid= -p pid
```



# 平均负载的理解

> 定义

1,5,15分钟时间内的平均活跃进程数

> 活跃进程 

处于`running/runnable` , `wait`,`iowait`的进程 , 在`ps`里看到的`STAT`是`R`和`D`的进程 ,`wait`也算入平均负载的原因可能是上下文切换的资源消耗

> 其他

1 当负载超过逻辑核心数百分之70就要小心

2 和cpu使用率不一样的地方是 , io 密集型程序的cpu使用率不一定高 

3 iowait高, 排查系统存储是否有问题

4 软中断和硬中断高,排查内核的中断服务程序



## 模拟cpu负载
`stress` 模拟平均负载升高工具
`stress —cpu 1 —timeout 600` 模拟cpu使用率,    
`stress -i 1 —timeout 600`  模拟io压力(不停执行sync) 
`stress -c 8 —timeout 600` , 模拟大量进程 , 8个在争抢2个cpu, 导致cpu过载 

## 工具

> 查看负载
`watch -d uptime`  

`top,htop,atop` 

`mpstat`用于看各个cpu的用户态,内核态,iowait等的占比 , pidstat查看进程的资源占用信息

> 查看cpu数  

`grep ‘model name’ /proc/cpuinfo | wc -l` (/proc是linux的一个虚拟文件系统,保存了系统的各种信息 `/proc/stat` `/proc/[pid]/stat`)
`mpstat -P ALL 5`  监控所有cpu , 每5秒输出一组数据
`pidstat -u 5`  1  5秒后输出一组


## 一些案例





# 上下文切换

> 定义

上下文=cpu寄存器+计数器
上下文切换=保存上一个任务的寄存器+计数器,读取下一个任务的信息到寄存器+计数器,跳到指定位置运行
任务的种类:进程,线程,硬件中断(会触发中断处理程序, 中断事件->取对应事件的处理程序,执行)



> 其他

一般不需要关注,但过多的上下文切换会缩短真正在运行的时间
上下文切换超过1万次就要注意了



## 模拟大量切换

`sysbench —threads=10 —max-time=300 threads run` 模拟多线程切换问题    10个线程 5分钟



## 查看工具

`vm stat 1 1`  查看上下文切换次数
`pidstat -w -u 1 `
`pidstat -wt 1` 看线程的指标
`watch -d cat /proc/interrupts`  查看中断次数 

## 一些案例



# cpu使用率高的排查步骤

> 定位进程

top , pidstat 

> 定位具体函数

perf top  实时定位
perf record , perf report  保存记录,离线分析
gdb  不适合线上定位, 会中断程序运行,用于线下进一步定位
strace 定位系统调用函数

推荐还是用perf

## 案例实验
`apt install docker.io sysstat linux-tools-common apache2-utils`

`docker run —name nginx -p 10000:80 -itd feisky/nginx`
`docker run —name phpfpm -itd —network container:nginx feiksy/php-fpm`

`curl 127.0.0.1:10000`
`ab -c 10 -n 100 https://127.0.0.1:10000/`

top 定位进程
perf top -g -p 21515 定位函数

docker rm -f nginx phpfpm
docker run —name nginx -p 10000:80 -itd feisky/nginx:cpu-fix
docker run —name phpfpm -itd —network container:nginx feiksy/php-fpm:cpu-fix



# 案例:cpu使用率高却找不到对应进程

docker run —name nginx -p 10000:80 -itd feisky/nginx:sp
docker run —name phpfpm -itd —network container:nginx feiksy/php-fpm:sp

top
pidstat 1      ps aux|grep 24344 交叉确认
pstree | grep stress 获取进程树 找到父进程
docker cp phpfpm:/app . 拷贝源码到本地

grep stress -r app 在代码里找stress
pref record -g  记录15秒性能报告 函数调用情况
perf report 分析报告

没有权限创建临时文件 , 导致频繁失败

execsnoop , 动态追踪短时进程 , 输出短时进程的基本信息 , 还可用于分析linux内核的运行时行为

短时进程,关键是要找到父进程

## 可能出现这种问题的情况

1.短时进程 , 应用调用了其他二进制程序,运行时间短,top什么不容易发现
2.应用在不停崩溃重启


# 僵尸进程

top    按下1查看全部cpu ,   zombie属性查看僵尸进程数
ps aux | grep /app  
dstat 1 10 查看cpu with io , 读写数变化
pidstat -d -p 4344 1 3        -d显示io统计
strace 跟踪进程系统调用   strace

R Running/Runnable
I Idle
D Disk Sleep 不可中断睡眠,如大量iowait时就可能有大量D状态的进程
S Sleep 可中断睡眠
Z Zombie 子进程已经退出,但父进程没有回收子进程的资源 , 重点关注长时间处于Z的进程
T

## 案例



# 网络问题排查/性能优化

## netstat 

tldr 常用
```
netstat

Displays network-related information such as open connections, open socket ports, etc.

- List all ports:
    netstat -a

- List all listening ports:
    netstat -l

- List listening TCP ports:
    netstat -t

- Display PID and program names for a specific protocol:
    netstat -p protocol

- List information continuously:
    netstat -c

- Print the routing table:
    netstat -nr

```

## 大量处于TIME_WAIT状态的连接

Linux性能优化 40,43,44,52

占用大量内存和接口资源,无法建立新连接

场景 用wrk模拟高并发负载下 , nginx作为客户端,连接php-fpm, 大量fpm执行超时,nginx主动断开连接
造成的问题 , TIME_WAIT占用住端口,无法建立新连接

优化方案
增大处于TIME_WAIT的连接数 net.ipv4.tcp_max_tw_buckets  (避免TIME_WAIT超出时被系统直接关闭连接,违反TCP)
增大连接跟踪表的大小 net.netfilter.nf_conntrack_max (这个案例里Docker用到了NAT,NAT和防火墙用到了连接跟踪表)
减小 net.ipv4.tcp_fin_timeout (尽快释放资源)
减小 net.netfilter.nf_conntrack_tcp_timeout_time_wait 尽快释放资源
开启端口复用 net.ipv4.tcp_tw_reuse  , 被占用的端口也能用于新连接
增大端口范围 net.ipv4.ip_local_port_range
增大系统文件描述符数量 fs.nr_open 增加最大连接数
应用程序的 systemd 配置文件中，配置 LimitNOFILE ，设置应用程序的最大文件 描述符数。

## SYN FLOOD 攻击

特征: sar -n DEV 1 , PPS特别多,BPS特别小

发送大量SYN包, 服务端发送SYN+ACK包后等待客户端的ACK包,直到超时,期间占用大量资源,无法服务正常客户请求
这种状态的连接称为半开连接 , 连接表被占满后无法建立新连接
此时的服务端会有大量的SYN_RECEIVED状态的连接 , 用netstat -n -p | grep SYN_REC | wc -l 查看 , or ss -s

单ip攻击 用iptable拒绝
多ip , hping3 -randip模拟 , 增大SYN_REC的最大数, 减少连接重试次数 , 使用TCP syn Cookies
