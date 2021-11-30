# go http client 错误使用

## 未读取 body 就 close

连接不会释放 , 连接池不够用 , 会导致大量短连接 , 大量 TIME_WAIT , 有端口用尽风险

为什么一定要读取? 查看源码


## 未配置 PerHost


http://tleyden.github.io/blog/2016/11/21/tuning-the-go-http-client-library-for-load-testing/

cpu 排查的几个案例 - Log0c
site:github.io go http client - Google 搜索
golang-http-client-time-wait | 鱼的记忆
net/http | Go语言标准包解析
golang http timeout源码解析
golang net/http标准库学习
golang的net/http包的客户端简单科普 · zhangguanzhang's Blog
Go的HTTP tracing | Brantou的日常
Timeout in Go net/http client
Golang实现http请求及代理设置 | IT范儿
Go net/http包源码解析（一） | ph4ntom's blog
压力测试中的Go http Client 参数调优 | 日拱一卒，功不唐捐
Tuning the Go HTTP client settings for load testing - Seven Story Rabbit Hole
site:github.io linux 程序在后台运行 - Google 搜索
Linux 后台运行程序
