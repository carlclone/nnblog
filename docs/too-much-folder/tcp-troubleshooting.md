# tcp 异常几例

各个异常如何复现 ,  hping3

tcp 异常造成的危害一般是占用如下 linux 系统资源

- 内核软中断(占用 CPU)
- 连接跟踪表
- 半连接队列
- 文件描述符限制数
- 端口  
- ...

## SYN FLOOD 攻击

- 危害

大量软中断,占用 CPU,正常用户 SYN 请求无法响应 ,占用连接跟踪表和半连接队列

- 含义

SYN 包属于合法请求,利用大量的只发 SYN 包不 ACK, 使系统有大量的半连接

- 异常指标

```
top 查看各个 cpu 的软中断(si) 占用
sar 查看 PPS (packet per second)
```

- 优化方案

增大连接跟踪表  ,增大半连接队列 ,使用 syn_cookie

参考[linux 内核参数速查表](/too-much-folder/troubleshooting-cheatsheet/#linux)

参考 [network 排查几例](/too-much-folder/black-box-network-problem/#syn_flood)

## FIN_WAIT_2 , CLOSE_WAIT 多

- 危害

占用端口,耗费内存,导致连接跟踪表变大,其他连接响应也变慢

- 含义

FIN_WAIT_ 2: 主动关闭一方发出 FIN 后处于 FIN_WAIT_1 , 收到对方 ACK 处于 FIN_WAIT_2 , 意味着等待对方的FIN 包 ,收到 FIN 包发出 ACK 后则处于 TIME_WAIT

CLOSE_WAIT: 被动关闭一方收到对方 FIN 被返回 ACK 后处于 CLOSE_WAIT,等待应用层调用 close()发出 FIN 包,发出则处于 LAST_ACK 等待对方最后一个 ACK

- 异常指标

处于各个状态的连接数统计
```
netstat -alepn | grep FIN_WAIT | wc -l
netstat -alepn | grep CLOSE_WAIT | wc -l
ss -s
```

- 优化方案

找到出问题的代码,添加连接关闭

## TIME_WAIT 多

- 危害

可能造成端口不够用或内存耗尽问题
  
- 含义

主动关闭一方收到对方的 FIN ,并发出 ACK 后所处的状态,默认等待 60s 回收所有遗留在链路上的包 , 避免旧包被新连接当做正常包接收,从而拒绝正常的数据包
  
- 异常指标

处于 TIME_WAIT 状态的连接数统计
```
netstat -alepn | grep TIME_WAIT | wc -l
ss -s
```

- 优化方案

增大端口范围

重用端口

增大连接跟踪表

增大 time_wait 最大数

减小 time_wait 超时时间

使用 timestamp=1

参考[linux 内核参数速查表](/too-much-folder/troubleshooting-cheatsheet/#linux)

## 参考资料

- RFC1337
- linux 性能优化实战
- linux 内核实战