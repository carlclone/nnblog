---
title: 博客服务器环境搭建记录
tags: 归档
date: 2016-12-10 14:57:31
---

无

<!-- more -->

> 用了差不多半天的时间把博客部署了起来，做一下过程的记录

# 服务器

Ubuntu 14.04+PHP7.1+MySQL5.7+Nginx
大部分参照[**此文**](https://lufficc.com/blog/step-by-step-teach-you-to-deploy-your-laravel-application-to-server)

## 修改Root密码 sudo passwd root

## 配置MySQL远端访问和root登录

### 启动和登录MySQL

查看mysql安装的路径和依赖：`whereis mysql`

```
mysql: /usr/bin/mysql /etc/mysql /usr/lib/mysql /usr/include/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz
```

这时你会发现 mysql以服务形式自启动
这时候在本机可以通过 mysql -r root -p你的密码  登录了

### 开启远程访问

默认情况下远程访问会出现 

```
Can’t connect to MySQL server on ‘192.168.10.18′ (10061)
```

错误，是因为mysql的默认配置为了增强安全性，禁止了非本机的访问，在ubuntu中是这样，debian中也是。禁止的方式以前是在my.cnf中有一句 `skip-network` 现在则变成了：

```
# Instead of skip-networking the default is now to listen only on # localhost which is more compatible and is not less secure. bind-address = 127.0.0.1
```

就是只监听本机地址。所以，如果要开放局域网内的访问，需要注释掉这一句，或者改成：`bind-address = 0.0.0.0` 就行了

`# vi /etc/mysql/my.cnf`
修改 `bind-address = 127.0.0.1`  为  `bind-address = 0.0.0.0`
修改完成后重启MySQL服务`# sudo /etc/init.d/mysql restart`

这样改完3306端口就开始监听了,但是MYSQL的权限还没有打开ROOT远程登陆
Mysql默认是不允许远程连接的，因为有很大的安全隐患。
需要手动增加可以远程访问数据库的用户。
方法一、本地登入mysql，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，将"localhost"改为"%"

```
#mysql -u root -proot
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
mysql>FLUSH PRIVILEGES;
```

修改完这些就可以远程访问了。

### mysql数据库重启命令

```
启动：sudo /etc/init.d/mysql start
停止：sudo /etc/init.d/mysql stop
重启：sudo /etc/init.d/mysql restart
```

# XBlog(Laravel5.3)

大部分参照[**此文**](https://lufficc.com/blog/how-to-install-my-blog)

## 七牛云存储配置

[AccessKey和SecretKey的获取](http://jingyan.baidu.com/article/5552ef47d98291518ffbc9bc.html)
Bucket 就是空间名
域名有三种：默认domain，HTTPS domain，自定义domain
默认域名在七牛云个人中心首页

# 总结

## 需要深入学习的点

* Nginx+fpm
* VIM快捷键
* Linux命令
* Laravel核心概念
* 设计模式
* Redis
* and so on....


