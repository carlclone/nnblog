# go http client 错误使用

## 未读取 body 就 close

连接不会释放 , 连接池不够用 , 会导致大量短连接 , 大量 TIME_WAIT , 有端口用尽风险

为什么一定要读取? 查看源码


## 未配置 PerHost

