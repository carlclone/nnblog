# cpu 排查的几个案例

## 使用 direct_io 导致 io 持续高位

### 问题描述

iowait 很高,且有大量不可中断进程和僵尸进程

### 排查

- `top` 发现平均负载很高, iowait 也很高 ,  且有许多僵尸进程 , 有 D 状态的进程
- 没有 cpu 占用高的, 为什么负载会高呢
- 查阅资料, 平均负载包含R 状态的和 D 状态的进程 , D 状态表示不可中断睡眠, 大概率在等待 IO
- 查看 D 状态的进程有两个 , 查看 IO 情况
- 使用 `dstat 1 10` 同时查看 cpu 和 io 详细情况 , iowait 升高时磁盘读也很大
- 使用 `pidstat -d -p xxx` 查看进程 io 情况 , 发现两个D 状态进程都没读数据
- 使用 `pidstat -d` 看所有进程 io , 发现确实是 D 状态进程 , 但进程号变了
- 使用 `strace -p 进程 id` 报错 permission denied , 发现进程很快消失 , `ps aux|grep 6082` 验证
- `perf record -g` / `perf report `
- sys_read 和 direct_io 函数 , 直接 IO 造成
- 改为使用普通 IO , 享受系统缓存读优化


- 还有僵尸进程问题 , 找出父进程然后让父进程回收
- `pstree -aps 3084` ( a 表示输出命令行选项 , p 表示 pid , s 表示指定进程的父进程)

### 复现
TODO


### 疑问

- 为什么 iowait 高会导致负载变高 , cpu 不是空闲着吗   (磁盘 IO 也是耗能操作 , 同样算做电力负载)

参考电路基础第六版一书 : 负载是接收能量的元件(能量吸收器) , 它与提供能量的发电机(能量源)相反 , 参见 4.9.1 节

- 不可中断睡眠状态(TASK_UNINTERRUPTIBLE)的定义

不可中断睡眠的意思是该进程在执行系统调用后睡眠期间不能被类似 SIGKILL 之类的信号中断唤醒, 只能在该系统调用完成后被唤醒, 参考[Uninterruptible Sleep](https://blog.csdn.net/Rong_Toa/article/details/108226571)



## SYN_FLOOD 攻击造成软中断升高

### 问题描述

正常服务无法响应或超时

### 问题排查

- 服务响应缓慢, 先排除客户端和服务端的问题, 再检查链路 , 造成服务端缓慢的原因 : CPU,内存,磁盘 IO,网络     , 一个个排查
- 客户端访问其他服务正常
- 检查服务端
- top cpu 占用不高, 也没有 iowait , 平均负载很低
- softireq 某个核心很高  , ksoftirqd 进程占最多
- 查看软终端次数 `watch -d cat /proc/softirqs`  , 网络接收软中断次数变得很快
- `sar -n DEV 1` 查看每秒接收的包 PPS , 和每秒接收的字节数 , 发现都是小包 (664 * 1024) / 12607 = 54 字节
- `tcpdump -i eth0 -n tcp port 80` 检查包 , 发现都是Flag(S ) SYN 包, 确认是 SYN FLOOD 攻击
- 从交换机 or 硬件防火墙封原 IP
- 开启 SYN_COOKIE ,增大半连接队列, 不能用 iptable 封,因为同样走软中断 , 导致其他网络包延迟(如 ssh)  (参考 linux 内核实战 11, linux 性能优化 43,44)

### 疑问

- 为什么软中断只占用一个核心

因为默认情况下网卡都是单队列,绑定某个核心,所以只有一个 CPU 能处理

- 可以使用 iptable 缓解 SYN_FLOOD 吗

可以,但是效果不大,因为包已经进入网络协议栈的处理阶段,还是会造成软中断问题,但至少可以缓解 tcp 半连接队列被堆满


### 复现
TODO


## 程序循环执行 CPU 密集函数

### 问题描述

使用ab 压测 web 服务, QPS 只有两位数, 且 cpu 占用已经满了

### 问题排查

- 使用 top 查看各项指标, CPU us 项偏高, 接近百分百 , 分别为几个 php-fpm 进程
- 使用 `perf top -g -p 进程 id` , 查看某一 php-fpm 进程的 cpu 采样结果 (-g 表示显示调用关系)
- 方向键切换到 php-fpm , 回车展开调用关系 , 定位到 `sqrt` 和`add_function`
- `grep 'sqrt' -r app/`  代码查找调用
- 修复代码

### 问题复现

- `docker run --name nginx -p 10000:80 -itd feisky/nginx`
- `docker run --name phpfpm -itd --network container:nginx feisky/php-fpm`
- `docker exec nginx ip addr  `
- `curl http://[nginx 的 ip]:10000`确认服务启动
- `ab -c 10 -n 10000 http://[nginx 的 ip]:10000`
  
- `docker cp phpfpm:/app .`  将容器中 php 源码拷贝出来

- `docker rm -f nginx phpfpm`
- `docker run --name nginx -p 10000:80 -itd feisky/nginx:cpu-fix`
- `docker run --name phpfpm -itd --network container:nginx feisky/php-fpm:cpu-fix`
- 再次验证 QPS





## 进程不停崩溃重启导致 cpu 高

### 问题描述

cpu 占用高却找不到占用高的进程 

### 排查

-  top 发现 cpu us占用高, 却没有占用高的进程 ( 短时进程, 很快就退出了 , 或是进程在不断崩溃重启 )
- `pidstat 1` 也是一样看不到占用高的情况
- top 的 running 队列有 6 个之多 (双核,竞争激烈) , 且 running 的进程都是 stress 进程, php-fpm 都处于 sleep
- `pidstat -p 24344` 检查其中一个,  找不到 , `ps aux|grep 24344` , 也一样
- 查看启动这些进程的父进程 `pstree |grep stress`
- php-fpm 调用的, 查看 stress 输出, 发现是php 调用外部 stress 命令没有权限创建临时文件
- 大量的 stress 命令在初始化阶段就失败了,导致 cpu 升高, 但是没有看到大量 stress 进程 , 如何进一步确认
- `perf record -g`   大概 15 秒  / `perf report`
- 发现 stress 占了百分之 77
- 其他工具 , `execsnoop` , 可以查看系统的所有 exec 调用进程 id,父进程 id,命令, 如果有大量相同命令, 就可以怀疑这里有问题了


### 复现
TODO



