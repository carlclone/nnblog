---
title: Laracasts学习记录
tags: 归档
date: 2018-01-07 14:57:31
---

一篇记录laracasts课程完成情况的文章,略微潦草,有时间记得重新整理

<!-- more -->

#### 贯穿所有课程的理念

(Jeff的讲课能力,能把一些十分复杂的东西讲的简单易懂,因为他我才喜欢上编程,也希望能和他一样保持热情)
Jeff传达给我的最重要的几个理念是:
先写出良好可读的抽象再实现具体细节;
一点小小的效率提升积少成多省下巨大的时间,对工具的熟练,灵活使用往往被忽略;
前期良好的设计和过程中合理的重构让项目不会陷入慢慢腐烂的状态;
对项目大小有正确的估计,不必死死遵循原则,灵活变通;
想办法简化复杂或者重复的操作（快捷键，Alias别名，VIM，脚本，代码片段如emmet，Alfred etc）

#### 重要的课程

`Lets build a form with Laravel and TDD `
学习了如何写出良好的测试,使得重构的过程中不再畏手畏脚,并通过Vue和Laravel的整合写出了一个功能齐全的论坛系统

内容分类: 
Laravel+PHPUnit的测试编写
重构,整理代码
Forum基础功能实现
Blade+Vue的使用方式
Laravel功能(Channel , 通知系统, 查询优化(N+1 , 复用) , Scout)
展望->下一个课程(How to Manage an opensource project)

`Learn Vue2 :Step By Step `
学习Vue2的一些基础概念,数据绑定,事件监听,组件.进阶学习了Webpack和vue-cli,路由构建SPA的过程,最后使用Vue+ES6的面向对象构建了一个可复用的前端Form组件

`Incremental APIs`
这是Laravel4时期的老教程,但内容并不过时,教会了我如何设计一个合理健壮的API,对照Twitter接口,对常用HTTP状态码进行封装,自定义状态码,对数据用Transformer类进行转换过滤,并展示了一个分页接口的设计(Laravel5的Paginate已经默认实现)

`SOLID Principles in PHP`
讲述面向对象设计的五大原则SOLID,看完之后我又去看了Taylor Otwell几年前写的书From Apprentice to Artisan , 写下了对他们的理解,[From Apprentice to Artisan 读后感](https://carlclone.github.io/Laravel/from-apprentice-to-artisan-review) 

`Design Patterns in PHP`
使用现实例子简单介绍了装饰器模式,适配器模式,模板方法模式,策略模式,责任链模式,Specification模式,观察者模式. 印象不深,我对设计模式的理解更多来自于PHP 对象,模式与实践一书.

`Advanced Eloquent`
第一遍没看明白,几个星期后的第二遍清晰了许多,Eloquent组件有相当复杂的分层和结构,这里是我的[Eloquent源码总结文](https://carlclone.github.io/Laravel/eloquent-source-code-summary)

`Simple Rule for Simple Code`
一些简单的原则提高代码的可读性,和[Clean Code PHP](https://github.com/jupeter/clean-code-php)的内容类似

`The PHP Practioner`
前半部分介绍了PHP的基础知识,后半部分实现了一个简单的框架,主要有:对PDO数据库常用操作的抽象(Connection类用于连接,QueryBuilder类用于执行查询操作),抽离配置(config.php),路由($_REQUEST匹配),分离视图Views.

`ES2015 Crash Course`
学习ES2015(ES6)的基本概念.

`Intermediate Laravel`
跟随者Jeff阅读了Laravel Scheduling , Command , Event/Listener , Illuminate Contracts , Container 部分的源码, 对内部精密的实现有了更深层次的了解, 课后阅读了 DI Container 反射解决依赖的部分的源码, 大部分核心概念得到了加强.

`Eloquent Techniques`
讲解了DB类transaction的源码执行过程(是个很好的学习闭包的例子)
设计一个跟踪文档修改记录的系统 (可以用版本区分)
记录字段的前后变化 
Filters类的设计,之前在Forum里看过

#### 还没懂的

`Whip Monstrous Code Into Shape`   
大量的复杂重构实践,难度略高,需要长期积累和不断实践

`Code Katas in PHP`    
偏简单算法和TDD规范,需要再看看
1.找Prime Factor 
2.转化为罗马数字
3.保龄球游戏(单元测试)
4.字符串计算器
5.网球计分
6.fizzbuzz游戏 

`Real-time with Socket.io`   
websocket长连接应用,Socket.io是websocket和其他许多通信机制的一层封装,提供给开发者一套简单调用的接口

#### 难度较低的课程

`Laravel Extra Credits`
大杂烩

`Laravel Auth Techniques`
Github 社会化登录的实践,OAuth的验证过程, 并阅读了Laravel Socialite包的部分简单底层封装.

`Testing Laravel`
完成

`PHP Extra Credits`
完成

`How Do I`
基本完成

`Object-Oriented Bootcamp`
完成

`PHP7 Up and Running`
需要复习

`Be Awesome in PHPStorm`
需要重新检查技巧
PHPStorm的使用技巧

`Git Me Some Version Control`
可能需要复习或者实践

`Learn Redis Through Examples`
完成

`Setup a Mac Dev Machine From Scratch`
完成

`Sublime Text Mastery`
忽略 换vscode编辑器了

`Understanding Regular Expressions`
正则匹配基础知识,https://regex101.com/   学习正则好网站.
一些必记基础回顾 : 
/ 表达式部分  /
[]匹配单个字符的所有可能性  ,在[] 里面的^ 代表除了这些之外的字符 
+ 一个或多个   
* 0或多个
{} 比如 \s{3} 表示匹配三个空格，\s[1,3]表示匹配一到三个空格
() 获取这个部分的匹配 , 使用$0 , $1 获取?
\w 与“[A-Za-z0-9_]”等效 大写和小写反过来
\s 指空白，包括空格、换行、tab缩进等所有的空白  \S 大写和小写反过来
\d 数字 
| 或
部分字符需要被转移\ 处理 如 .
^ 匹配的开始  $ 匹配的结束
? 懒惰匹配 尽可能短
. 匹配任何单个字符

`Vim Mastery`
Jeff说了一句对我影响深远的话,让编辑器和自己的手和身体融为一体,减少在操作上的无畏消耗

`FrontEnd Extra Credit`
完成

#### 一些应用型的课程:

全部完成
`Build Project Flyer with me`
`Hands on Community Contributions`
`Discover Symfony Components`
`How to Build Command-Line Apps`

