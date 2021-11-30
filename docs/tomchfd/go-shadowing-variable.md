# go shadowing variable 造成的歧义



## 问题描述

如下面代码的一个场景,先从缓存中尝试获取文章 ID,如果缓存失效了,则从数据库中获取,预期的结果应该是在缓存失效的情况下,依旧能获取到正确的文章 ID

```
articleID,cacheMiss:=getArticleIDFromCache()
if cacheMiss {
    articleID,err:= getArticleID()
}
fmt.Println(articleID)
```

问题就出现在了 `articleID,err:= getArticleID()` , 由于 err 变量不存在, 需要使用`:=`符号进行声明和赋值,此时就会产生歧义:这里的 articleID 到底是在内层声明了新的变量,还是绑定着外层作用域的同名变量呢? 答案是内层作用域声明了一个新的同名变量,于是就造成了缓存失效时articleID 永远为 0 的 bug



## 解决和避免方法

- Go 的显式设计哲学 , C 的风格, 先声明后使用,缺点是无法享受 `:=` 的便利性

```
var article int = 2

if true {
    var err error
    article,err = getArticleId()
}
```

- IDE 和 Lint 代码静态检查

Goland : 开启probable bug inspection , 并修改为 error 级别警告 , 修改路径 `Settings->Inspections->Go->Probable bugs` , 右键选择`error` , 其中的一条 inspection 检查了上述的歧义问题 , `Reports declarations of variables that overlap with the declarations in the outer scope.`



其他编辑器: 使用 golangci-lint 检查 , 安装命令 `go get github.com/golangci/golangci-lint/cmd/golangci-lint@v1.41.1` , 运行 `golangci-lint run` , 与 Goland 集成方式 `settings->plugins` 搜索golangci 插件



# 参考资料

- [lox : resolving-and-binding](https://craftinginterpreters.com/resolving-and-binding.html)
- [Go 语言项目开发实战-代码检查：如何进行静态代码检查？](https://time.geekbang.org/column/article/390401)



## 后续
其他代码工具

- go run -race 并发检查
- go test
- go vet 静态检查
- gofmt

https://goreportcard.com/report/github.com/gammazero/workerpool
