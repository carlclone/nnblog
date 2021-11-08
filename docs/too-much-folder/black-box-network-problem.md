# network 排查的几个案例

## DNS 解析失败或解析慢

### 排查
- ping -c3 114.114.114.114 检查 dns 服务器连通性 , 正常
- time nslookup -debug time.geekbang.org 执行 dns 解析,输出 debug 信息 , 并输出解析所用时间, 发现 DNS 服务器为 127.0.0.1
- cat /etc/resolv.conf  (存有 dns nameserver 配置) 为空 , 存入 114.114.114.114 

- dig

  

优化工具

- (dns 缓存) /etc/init.d/dnsmasq start  , 并修改 resolv.conf 为 127.0.0.1 
- (dns 缓存) nscd

## ping 反向解析导致(PTR) 总耗时高

- ping -c3 geektime.org , 每次耗时不高, 但总耗时高
- 猜测是 dns 导致慢, time nslookup geektime.org , 结果正常
- tcpdump 
- traceroute



## SYN_FLOOD 攻击导致的服务缓慢或不可用



## 并发高时请求延迟变大问题

