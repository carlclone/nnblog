# network 排查的几个案例





## DNS 解析失败或解析慢



### DNS 介绍

DNS 服务器存储的记录类型有

- `A`  :  域名-> ip 的映射
- `CNAME`:  域名-> 域 的映射
- `NS`:  域名 -> 域名服务器 的映射

DNS 的查询方式

- 递归查询, 如果 DNS 服务器没有命中缓存, 会查询其他服务器再返回准确结果
- 迭代查询, DNS 服务器不返回结果, 而是返回其他服务器的地址, 让客户机自己去重新查询



### 排查

- `ping -c3 114.114.114.114` 检查 dns 服务器连通性 , 正常

- `time nslookup -debug time.geekbang.org` 发起一次 dns 请求, 并输出 debug 信息 , 并输出解析所用时间, 发现 DNS 服务器为 127.0.0.1

- `cat /etc/resolv.conf`  (存有 dns nameserver 配置) 为空 , 存入 114.114.114.114 

  

其他排查思路参考
- 检查本地hosts：`cat /etc/hosts`
- 检查resolv.conf文件：`cat /etc/resolv.conf`。在redhat7/centos7上修改resolv.conf里的DNS地址后，重启启网络服务发现DNS地址消失了，那么检查下网卡配置文件。
- 检查网卡配置文件： `cat /etc/sysconfig/network-scripts/ifcfg-<网卡名称>`，看下里头有没DNS配置信息，没有的话补上去。
  
  
  

### 优化工具

- (dns 缓存) `/etc/init.d/dnsmasq start ` , 并修改 `resolv.conf` 为 127.0.0.1 
- (dns 缓存) `nscd`

###其他工具

- `dig +trace +nodnssec time.geekbang.org` 跟踪 DNS 的递归查询过程 

    `+trace`表示开启跟踪查询 ,  `+nodnssec`表示禁止DNS安全扩展



### 参考资料

- [重启后 resolv.conf 还原问题](https://blog.csdn.net/lishuan182/article/details/81383448) 
- [nslookup 正确,ping 却unknown host](https://plantegg.github.io/2019/01/09/nslookup-OK-but-ping-fail/)  不该用 resolv.conf 同时做内外部 DNS 解析, 应该配 upstream dns , 但分析过程值得学习



## ping 反向解析导致(PTR) 总耗时高



### 排查

- `ping -c3 geektime.org` , 每次耗时不高, 但总耗时高
- 猜测是 dns 导致慢, `time nslookup geektime.org` , 结果正常
- `tcpdump -nn udp port 53 or host 35.190.27.188`  抓取 dns 解析的数据包 (-nn 表示不反向解析 )
- 执行 `ping -c3 geektime.org`
- 查看 tcpdump 结果,有两个 PTR 查询, 每个耗时 5 秒(超时了)
- PTR 是反向解析, 在这里的作用只是人类友好(  ip -> 域名)
- `man ping` , 查得 -n 可以禁止 PTR



### 使用 wireshark 分析 pcap

- `tcpdump -nn udp port 53 or host 35.190.27.188 -w ping.pcap` 导出抓取的数据
- `scp host-ip/path/ping.pcap .` 或 ftp 下载到本地
- `Statistics-> Flow Graph , Flow type 选 TCP Flows` , 可视化 TCP 执行过程



### 复现

```
# 禁止接收从DNS服务器发送过来并包含googleusercontent的包  (使 PTR 超时)
$ iptables -I INPUT -p udp --sport 53 -m string --string googleusercontent --algo bm -j DROP
```



### 其他

- tshark (wireshark 命令行版)
- wireshark 学习书籍 《Wireshark网络分析就这么简单》和《Wireshark网络分析的艺术》
- 在使用 mysql 的时候也遇到过反向解析导致的超时





## SYN_FLOOD 攻击导致的服务缓慢或不可用

### DOS 介绍

拒绝服务攻击, 通过大量合理请求耗尽服务端某个资源使得正常用户的请求无法响应 (可能连 ssh 都上不去 ), 通常类型有

- 消耗`带宽`
- `操作系统资源` CPU, 内存, 连接表, 各种队列(如半连接队列)
- 消耗`应用程序资源`, 如数据库 (大量 cache miss, 请求打到数据库)

### 模拟复现

-  `curl -s -w 'Http code: %{http_code}\nTotal time:%{time_total}s\n' -o /dev/null http://192.168.0.30/` 

获取正常情况下的响应时间 , -w表示只输出HTTP状态码及总时间，-o表示将响应重定向到/dev/null

- 正常情况只需要 0.002s , 2 ms
- `hping3 -S -p 80 -i u10 192.168.0.30` 模拟 SYN 攻击

-S 表示 TCP 协议的 SYN , -p 表示目的端口 ,` -i u10` 表示每隔10 微秒发一个网络帧

- 再执行 curl , 正常请求超时了
- `sar -n DEV 1` 查看网络指标 PPS (packet per second) 和 BPS (byte per second)
- `tcpdump -i eth0 -n tcp port 80` 抓取 eth0 网卡的数据包
- 有大量`Flags [S] `, SYN 包的请求
- 使用 wireshark 观察 TCP 流, 客户端收到 SYN+ACK 后不再 ACK , 导致服务端消耗资源直到握手超时
- 此时这类连接的状态为 SYN_RECEIVED 
- `netstat -n -p | grep SYN_REC`  查询为 SYN_REC 的连接情况 , -n 表示不执行反向解析, -p 显示所属进程
- `netstat -n -p | grep SYN_REC | wc -l `统计 SYN_REC 的数量
- 用户不能访问的原因是, 连接表被占满
- `iptables -I INPUT -s 192.168.0.2 -p tcp -j REJECT` drop 掉单点 DOS 攻击, 缓解连接表问题
- 用户可以访问了, 但耗时比较高 1.5s , 因为 SYN 包还在触发大量软中断消耗 CPU  (参考 CPU 案例第一个 )



### 分布式 DOS 攻击的服务端缓解手段

- 限制 SYN 并发数每秒 1 次 (杀敌一千自损八百)

`iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT`

- 限制单个 IP 在 60 秒内新建连接总数为 10

```
iptables -I INPUT -p tcp --dport 80 --syn -m recent --name SYN_FLOOD --update --seconds 60 --hitcount 10 -j REJECT
```

注: 模拟分布式攻击 : hping3 的 `--rand-souce`

- tcp 优化手段

  - 提高半连接队列大小

  ```
  sysctl net.ipv4.tcp_max_syn_backlog 
  net.ipv4.tcp_max_syn_backlog = 256    # 当前为 256
  
  sysctl -w net.ipv4.tcp_max_syn_backlog=1024 # 修改为 1024
  
  ```

  - 减少 SYN_REC 状态时的重试次数

    ```
    $ sysctl -w net.ipv4.tcp_synack_retries=1 
    ```

  - 使用 syn cookie

  ```
  # 开启后半连接限制会失效 ( 因为 syn cookie 没有半连接了)
  # 最后一次握手 ACK 使用 cookie 确认后才建立连接
  
  $ sysctl -w net.ipv4.tcp_syncookies=1 
  ```

  

  - 配置持久化生效

  ```
  $ cat /etc/sysctl.conf net.ipv4.tcp_syncookies = 1 net.ipv4.tcp_synack_retries = 1 net.ipv4.tcp_max_syn_backlog = 1024
  
  sysctl -p 命令后，才会动态生效
  
  ```



### 分布式 DOS 攻击进阶

- 为什么不太好在服务器端做攻防?

因为 pps 大时, linux 的协议栈是个巨大的负担 , 包括软中断, 各种队列 , 连接表 , 缓冲区

- 更好的解决方案 ?

在 linux 协议栈前拦截 (购置专业的入侵检测和防御设备，配置流量清洗设备阻断恶意流量等)

绕过 linux 协议栈 (XDP 或者 DPDK)

- 慢速 DDOS 

请求不大, 响应很大 , 且耗系统资源  , 比如 ssp 的 stat 接口

缓解: 应用程序识别 , 如合理利用缓存、增加 WAF（Web Application Firewall）、使用 CDN







## 并发高时请求延迟变大问题

- 如何测单次网络延迟

  `ping` 

  `hping3 -c 3 -S -p 80 baidu.com` icmp 被禁时, 使用

  `traceroute --tcp -p 80 -n baidu.com`  原理是向每一跳发送限制 TTL 的三个请求, 获得响应计算往返时间 , 星号表示超时或无响应  [traceroute 原理](https://www.jianshu.com/p/75a5822d0eec)

- 如何测并发时的延迟

wrk 安装 (类似 ab 的压测工具)

```
$ https://github.com/wg/wrk
$ cd wrk
$ apt-get install build-essential -y
$ make
$ sudo cp wrk /usr/local/bin/
```



### 排查

- `hping3 -c 3 -S -p 80 192.168.0.30` 
- `hping3 -c 3 -S -p 8080 192.168.0.30` 
- 对比普通 nginx 和案例 nginx 的单次请求延迟, 都正常
- `wrk --latency -c 100 -t 2 --timeout 2 http://192.168.0.30/`
-  `wrk --latency -c 100 -t 2 --timeout 2 http://192.168.0.30:8080/` 
- 并发普通 nginx , 90 分位点有 9ms , 并发案例 nginx, 75 分位只有45ms
-  `nginx tcpdump -nn tcp port 8080 -w nginx.pcap`  
-  `wrk --latency -c 100 -t 2 --timeout 2 http://192.168.0.30:8080/`
- 抓包使用 wireshark 查看, 数据包太多怎么办? `Follow -> TCP Stream  , 可视化 Statics -> Flow Graph，选中 “Limit to display filter” 并设置 Flow type 为 “TCP Flows"`
- 发现TCP 连接的第二次 HTTP 请求的客户端 ACK 延迟了 40ms (TCP 延迟确认（Delayed ACK）的最小超时时间 )
- 猜测客户端(wrk)开启了延迟确认 
- `man tcp` 查阅文档, 发现默认就开启延迟确认的 , 只有在设置了TCP_QUICKACK 才关闭
- `strace -f wrk --latency -c 100 -t 2 --timeout 2 http://192.168.0.30:8080/ `查看 wrk 系统调用, 看看给套接字设置了什么参数
- `setsockopt(52, SOL_TCP, TCP_NODELAY, [1], 4) = 0`
- 确实没有设置 QUICKACK , 但是还有一个问题, 在客户端并发那么高的时候, 同一时间应该有很多小包需要发送 , 足够凑成一个大包 , 从而不需要等待延迟确认超时后单独发送
- 继续往下看, 发现 1175 号的 ACK Seq和 697 号都是 85 , 构成同一个 HTTP 响应 , 但1175 却等到客户端对 697ACK 后才发送
- 查资料确认是 NAGLE 算法, 减少小包发送数量提升宽带利用率的算法 , 只有等到前面一个大分组 ACK 之后, 才会继续发后面的分组 , 和客户端的延迟确认一起用就造成了延迟明显
- NAGLE : 一个 TCP 连接上，最多只能有一个未被确认的未完成分组 , 所以不会在这之前不会发送其他分组

- 带宽利用率低的例子 , 大部分被 header 占用了 ,  当有效负载只有 1 字节时，再加上 TCP 头部和 IP 头部分别占用的 20 字节，整个网络包就是 41 字节，这样实际带宽的利用率只有 2.4%（1/41）

- `man tcp , TCP_NODELAY` 才会关闭 nagle
- 查看 nginx 配置

```
$ docker exec nginx cat /etc/nginx/nginx.conf | grep tcp_nodelay
    tcp_nodelay    off;
```

- 重新压测 , 这次 90 分位有 9ms 了
- 重新抓包, 这次同一个 http 的一起发了 , 不再等客户端的 ack 才发了

![123](/img/http-tcp-packet-at-same-time.jpg)

