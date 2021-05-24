---
title: K480N安装黑苹果记录
tags: 归档
date: 2016-12-18 14:57:31
updated: 2016-12-18 14:57:31
---

--

<!-- more -->


## 未解决

电池 √

CPU 睿频（用MBP9.2的SMBIOS） √

[睡眠](http://bbs.pcbeta.com/viewthread-951913-1-1.html)  [睡眠2](http://bbs.pcbeta.com/viewthread-1480332-1-1.html)

麦克风（可能要原生驱动）

开机有点慢（加载kext慢一点点，主要是-v过程慢很多）



## 总结

+ HWMonitor的安装  pkg安装 记得自定义选。 要安装上传感器才有显示
+ 别装太多插件 Texura NTFS之类的，或者干脆别装，输入法也用原生的
+ 自启动太多，主要瓶颈 硬盘读写
+ 有问题搜远景
+ 变色龙，启动到界面前的配置在wowpc里，点击mac启动的在系统的Extra的plist里
+ sudo passwd root 修改默认root密码



## 整理的文件

+ [hwsensor](http://bbs.pcbeta.com/viewthread-1561279-1-1.html)
+ [小度wifi驱动+bearextender](http://bbs.pcbeta.com/viewthread-1571321-1-1.html)
+ 本地文件夹【11.26 升级Sierra整理】
  - Extra整个备份
  - 小度wifi+bearextender
  - 装到SLE里的第三方kext
+ [MBR补丁+内核](http://bbs.pcbeta.com/viewthread-1695930-1-3.html)



## 系统个人设定