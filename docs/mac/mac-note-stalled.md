# mac Notes.app 卡死问题

## 问题描述
用了 3 年多的 10.13.6 系统 , notes.app 软件时长会出现卡住几秒不动的情况

## 排查
- 打开console.app (包含各种日志) 搜索关键词 `note`
- 日志显示是 sqlite 数据库文件 malformed 了 
- 删除后重新生成问题解决 , 文件路径`/Users/lin/Library/Group Containers/group.com.apple.notes`


![image-20191120140931373](/img/b.jpg)
![image-20191120140931373](/img/a.jpg)


