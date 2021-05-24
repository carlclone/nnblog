---
title: 第一份实习总结
tags: 归档
date: 2017-03-17 14:57:31
---

实习和待完善知识点的总结

<!-- more -->

---
#### 管理后台的完整搭建，功能更新，维护。

使用ThinkPHP5 +RBAC搭建的广东高校招协联盟后台管理系统。          

            
 #####  首页面板负责显示统计数据。

![Paste_Image.png](/img/xuegeng/3830995-4970b99c36f96d17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ##### 管理员模块添加和删除用户，分配权限（分学校管理员，最高管理员和编辑）

![Paste_Image.png](/img/xuegeng/3830995-d74d6963af2fa4c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####  BANNER，新闻，公告，商品信息，MENU，微信关键字回复 6个模块负责添加，编辑和软删除相关的描述，有回收站功能（从数据库中删除），Editor使用CKEditor插件，图片处理使用think-image ，日期使用datetimepicker 。

![Paste_Image.png](/img/xuegeng/3830995-9fed3a4b06b5f366.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Paste_Image.png](/img/xuegeng/3830995-9afc270d75f5bb10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 志愿者信息负责Excel表格导入导出（PHPExcel）和显示（bootstrap-table），并有相关的数据分析图表（echarts）

![Paste_Image.png](/img/xuegeng/3830995-71f70ee2766651f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Paste_Image.png](/img/xuegeng/3830995-95bd66bee1558f24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 问题审核模块对微信端网页提问功能进行后台审核。

![Paste_Image.png](/img/xuegeng/3830995-3f4886e2eb52987b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Paste_Image.png](/img/xuegeng/3830995-3bc1d805dc30954f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 用户分析模块对微信公众号用户按类型分类，可选时间段为横坐标绘制折线图（echarts），并有表格显示时间段内相关用户信息

![Paste_Image.png](/img/xuegeng/3830995-ba23489ad9537328.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

                 

#### 微信端接口编写

公众号类型：企业号
用户信息获取
[官方文档阅读](https://mp.weixin.qq.com/wiki/home/)
相关配置 
事件回复 
微信端网页，授权 JSSDK
开发，测试方法，工具  
微信网页接口签名验证：ksort参数，拼接字符串+key，md5加密
[微信支付](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=1_1)，[企业付款](https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_1) 
[EasyWechat SDK的使用](https://easywechat.org/zh-cn/docs/)

Q&A整理：



#### APP端接口编写

配置iOS的HTTPS认证
接口签名同上
登录注册接口：友盟短信验证码

#### 服务器搭建和配置

LAMP  阿里云Ubuntu14.04 + Apache2 + Mysql5.5 + PHP5.6
LNMP 腾讯云Ubuntu14.04+Nginx + Mysql5.7 + PHP7.0
SSL证书配置  阿里云Symantec证书+Apache ，StartSSL+Nginx
SVN服务器搭建：svn hook代码自动部署（相关配置要对应SVN大版本号）
计划任务脚本：数据库定时备份，post-commit
安全设置：SSH连接安全，账户权限

#### 数据库的学习
MySQL 语句 ， 优化 
数据库结构设计
索引，结构，查询优化
Redis
mysql-binlog

#### 版本控制学习

SVN
Git

#### 其他工具

Composer
NPM
正则表达式

#### 框架的掌握

ThinkPHP 5.0
Laravel 5.3

#### 理论方面（OOP，DP，TDD）


#### 遇到的重要问题和解决思路


#### 前端方面的知识

Javascript框架 ： Jquery ，vue
Javascript插件：
CSS ：SASS
前端框架 ： bootstrap ，bulma
工具： 打包工具和workflow工具

#### 产品功能和目的

广东高校招协联盟
今朝Pro
后台管理

#### 其他业余学习

NodeJS
Python
Ruby

#### 问题查找步骤
Google>Baidu>Stackoverflow

#### 测试相关

#### 重构的艺术

#### 书籍阅读

#### 视频阅读

Laracasts

#### 工作流

OS ： macOS sierra , ubuntu 14.04，ArchLinux
Editor：PhpStorm，Sublime Text，VIM
调试工具：XDebug，SocketLog
本地环境：MAMP，Laravel Valet

#### 需进一步学习

Docker

#### 总结

