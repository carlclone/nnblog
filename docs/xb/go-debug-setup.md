# 配置 go 的 debug



delve

dlv debug main.go --wd /Users/lin/go/src/xiaobu/biz_server/output/game_biz/bin -- -config /Users/lin/go/src/xiaobu/biz_server/output/game_biz/conf/application.yml


goland


自定义 debug 配置 

output dir 配上运行目录 /Users/lin/go/src/xiaobu/biz_server/output/game_biz/bin

program arg 加程序运行参数 -config /Users/lin/go/src/xiaobu/biz_server/output/game_biz/conf/application.yml

all done !





https://www.jetbrains.com/help/go/creating-and-editing-run-debug-configurations.html#create-go-build-configuration

https://github.com/go-delve/delve/issues/562

https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_debug.md

https://github.com/go-delve/delve/blob/master/Documentation/installation/README.md

https://www.bilibili.com/video/BV1pb411i7nw?from=search&seid=7748383152437991388