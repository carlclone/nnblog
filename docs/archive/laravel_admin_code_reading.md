---
title: Laravel-Admin源码阅读--编程思想收获
tags: 归档
date: 2017-07-20 14:57:31
---

一个值得深入学习源码的laravel扩展

<!-- more -->


# 数据模型表格Grid的代码追踪

传入Eloquent模型和使用者Grid相关自定义设置的闭包

![Paste_Image.png](/img/ladmin/3830995-fe1641edd7516a18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Laravel的Facade实现
![Paste_Image.png](/img/ladmin/3830995-9f4ffa01a5a08ae7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Admin类的grid方法主体，将前面两个参数传入实例化的Grid类中（依赖注入）
![Paste_Image.png](/img/ladmin/3830995-dcb37b8268567b0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

初始化构造（依赖注入）
![Paste_Image.png](/img/ladmin/3830995-266b280d7c9fc334.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

追踪builder闭包的使用情况

将当前Grid类作为参数传入闭包，执行使用者在闭包中的自定义设置

# 导出表格方法
![Paste_Image.png](/img/ladmin/3830995-0fe28b8726a94fbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 表格过滤方法
![Paste_Image.png](/img/ladmin/3830995-4742a345d38a54e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

