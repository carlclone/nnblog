# 零散的工具使用技巧

## Datagrip

大的项目,数据库表一般都有成百上千张, 开发一个需求的时候翻来翻去相当麻烦, 个人不是很喜欢在工具的 Tab 上肉眼找打开的表

Datagrip 提供了一个`favorite list`的功能, 可以让你将相关的表放在一起, 比如说本次需求的表, 并且可以在列表任意一处直接输入表的名称进行搜索



```
然后是 transpose 视图 , sql 代码提示/错误提示(jetbrains 在这方面 top1)
```

## Fiddler



```
web 开发,http 调试利器

1.直接 mock 接口

2.查看 RAW HTTP Body (Chrome 没有这个功能,Firefox 有)

3.拦截请求或响应,修改

4.重放请求,有时候减少了重复造数据的时间 (有的接口安全要求没有那么高..没防重放)
```

## Navicat

navicat premium 的数据传输功能是其他数据库客户端没有的, 支持对数据库的数据和结构进行 diff , 同步, 复制, 如果要将测试环境的库拉到本地加快访问, 推荐使用这个功能



```
navicat 平时开发不会用到,但是一些特殊场景这工具非常契合 , 如同步或 DIFF两个数据库的数据表结构 , 数据 , 速度很快
```















用phpstorm condition过滤掉定时访问的请求       $action != getOfflineAction

额外分享：

数据库操作的客户端软件用过几个比如mac下的sequel pro ，navicat premium , datagrip

sequel pro 是界面和操作流程最简洁直接的，

navicat有许多高级功能，特别是同步和备份时速度很快 

datagrip 和navicat可以作为互补 ， datagrip更适合平日开发使用，有很方便的favorite list分组功能，把本次开发相关的表放在一次，省下找表的时间

并且有一个transpose视图功能，一行一行的数据列向显示，再也不用去拉滚动条看后面的列了

窗口切换是个问题

输入法中文的

使用fiddler的AutoResponder功能mock未实现的接口

fiddler的raw窗口可以查看原始http请求内容 , 复制粘贴到phpstorm的http client里可以快速测试 也支持加文件 和变量 , 方便版本管理

fiddler里也可以直接修改然后测试

