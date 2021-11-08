# 使用 pprof 排查内存泄漏

## 问题描述

后台系统偶尔会出现 500 无法服务的问题然后自动重启,通过服务器查看日志得知被 OOM kill 掉了,内存是一定够用的,所以当做内存泄漏问题排查



## pprof 介绍

go 官方提供的 profiling 工具, 通过定期采集样本的方式获取 CPU, heap 内存, goroutine , thread , block , mutex 等信息提供分析能力 , 每份样本包含的信息有:

• 调用栈位置

• 当前函数，排除当前函数中调用其他函数，的指标值（例如只在当前函数中申请的堆内存）

• 当前函数，包括当前函数中调用其他函数，的指标值（例如在当前函数内部，包括调用的其他函数，申请的堆内存）



并提供可视化的工具展示上述样本集, 如表格 , call graph , 火焰图



## 排查过程

- 在代码中添加`import _ "net/http/pprof"` 开启样本采集, 部署到线上

- 由于出现的时机并不确定,写脚本每分钟判断,当内存使用率超过百分之 70, dump 一个 memory profile

```
#!/bin/sh

while [ 1 ]; do
     top -bn2 | grep "KiB Mem" | tail -1 | awk '{
         # $2 is total, $4 is free.
         if ( ($2-$4)/$2 * 100 >= 70 ) {
              
              while ("date" | getline date) {
                   split(date, str, " ");
                   prefix=sprintf("%s_%s_%s_%s", str[2],str[3], str[4], str[5]);
               }

              sys_usr_file=sprintf("/tmp/%s_info.highsys", prefix);
              print $2 > sys_usr_file;
              print $4 >> sys_usr_file;

              # run sysrq
              #system("echo t > /proc/sysrq-trigger");
              system("go tool pprof http://localhost:6060/debug/pprof/heap");
         }
     }'
     sleep 1m
done
```

- 将 profile 下载到本地, mac 打开 profile 需要安装 `brew install graphviz` , 或`sudo port install graphviz` 

- `go tool pprof -http=":8081" profile.out` 打开可视化界面, 查看 inuse_space 视图

- 通过 call graph 定位到一个函数,该函数接收两个参数,开始日期和结束日期,生成日期范围内的所有日期字符串数

```
func GetBetweenDates(sdate, edate string) []string {
	d := []string{}
	timeFormatTpl := "2006-01-02"
	if len(timeFormatTpl) != len(sdate) {
		timeFormatTpl = timeFormatTpl[0:len(sdate)]
	}
	date, err := time.Parse(timeFormatTpl, sdate)
	if err != nil {
		// 时间解析，异常
		return d
	}
	date2, err := time.Parse(timeFormatTpl, edate)
	if err != nil {
		// 时间解析，异常
		return d
	}
	if date2.Before(date) {
		// 如果结束时间小于开始时间，异常
		return d
	}
	// 输出日期格式固定
	timeFormatTpl = "2006-01-02"
	date2Str := date2.Format(timeFormatTpl)
	d = append(d, date.Format(timeFormatTpl))


	for {
		date = date.AddDate(0, 0, 1)
		dateStr := date.Format(timeFormatTpl)
		d = append(d, dateStr)
		if dateStr == date2Str {
			break
		}
	}
	return d
}
```

- 当开始日期和结束日期相等时,没有作判断,于是就造成了无限循环 append 数组,最终 OOM , 后续得知是从网上 copy 的代码, 从网上拷代码还是得再验证 , 写几个测试用例 , 也尽量找 star 多, 可信度高的开源项目


## 内存指标查阅

- top/free -h 查看整体内存,各进程内存情况
- vmstat 3 查看整体内存变化趋势 
- memleak 查看某进程内存分配请求的地址 , 大小 `/usr/share/bcc/tools/memleak -a -p $(pidof app)`
- pidstat -r -p 13084 查看进程内存情况
```
-a 表示显示每个内存分配请求的大小和地址
-p 指定 pid
容器里运行的程序会看不到调用栈,需要把程序的二进制文件复制到宿主机对应的目录上 (参考 linux 性能优化实战 18)
```

- sysstat 软件包安装 `sudo apt-get install -y sysstat`
- bcc 软件包安装
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4052245BD4284CDD
echo "deb https://repo.iovisor.org/apt/$(lsb_release -cs) $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/iovisor.list
sudo apt-get update
sudo apt-get install bcc-tools libbcc-examples linux-headers-$(uname -r)
```


## 参考资料
- [pkg.go.dev/net/http/pprof](https://pkg.go.dev/net/http/pprof)
- [linux 内核实战-进程的哪些内存类型容易引起内存泄漏](https://time.geekbang.org/column/article/280455?utm_source=related_read&utm_medium=article&utm_term=related_read)
- [call graph](https://github.com/google/pprof/blob/master/doc/README.md#interpreting-the-callgraph)
- [pprof 排查 Golang 服务内存问题](https://xie.infoq.cn/article/56c801b339241fd80c3b8f616)
- [linux 内核实战,业务是否需要透明大页](https://time.geekbang.org/column/article/292060)
- [linux 性能优化实战 18]()

## 后续

观察到 Go 程序的一个现象,VIRT 列(虚拟内存) 特别高,虽然不影响,但还是好奇查阅了一下,参考这篇文章 [go 虚拟内存为什么这么大](https://blog.csdn.net/EDDYCJY/article/details/109475941)

