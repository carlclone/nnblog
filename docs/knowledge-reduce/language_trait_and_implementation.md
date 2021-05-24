---
title: 知识点的reduce-语言特性_底层实现
date: 2019-08-02
tags: 计算机基础
---

&nbsp;

<!-- more -->

# Py 的协程 / JS的协程




## Coroutine 模型对比

Go的是m:n的协程线程模型 , 并且支持多核并行 , 实现了自动调度 , 是相对完整的协程模型

Py的是m:1的单线程事件循环模型 ,  只支持并发 , 因此在cpu bound的场景下使用协程没有优势 , 没有实现自动调度 , 由用户决定调度位置

js的await / async也是协程 , 和py模型一样 , 但是使用方式不太一样, js可以直接执行异步函数 , py生成的是Coroutine object , 要配合其他才能执行



## Coroutine 概念

1.是用户态的并发调度

2.协程类似线程 , 但是线程的控制权是操作系统 , 用户没有控制权限 , 而协程把控制权交给了用户 

3.“协程就是可以人为暂停执行的函数”。如果你觉得，“这听起来像是生成器(generators)”，那么你是对的。



## 模型适用场景

多进程    cpu bound , 并行执行

多线程    io bound , 避免了进程上下文切换的消耗  , CPU空闲/阻塞在IO的时候切换到其他可运行线程 , 当前线程放入等待队列

事件循环 io bound best choice , 避免了线程维护的消耗 , 同上


## python 并发编程 demo代码+注释

https://github.com/carlclone/python-concurrent-programming-demo


## 参考资料

[IO是否会一直占用CPU]( https://www.zhihu.com/question/27734728)

[Python 3.5中async/await的工作机制]( https://www.cnblogs.com/harelion/p/8496360.html)

https://en.wikipedia.org/wiki/Coroutine

https://en.wikipedia.org/wiki/Thread_safety

https://en.wikipedia.org/wiki/Race_condition#Computing

[多线程可以同时使用 CPU 的多个核心？](https://v2ex.com/t/285551)






## Py的并发接口

async 标识可以被调度/异步的操作 (返回Coroutine object)

await标识阻塞的位置 ( 用户的调度控制权)

task 是调度基本单位


## 异步编程风格的演化

以JS为典型 :  回调 , 观察者模式 , 发布/订阅 ,  

底层的实现方式 , 借助事件循环模型来理解


## 完

py的协程就是随处可见的事件循环模型 , 挺好理解的 , go的要复杂的多 , 以后再开一章总结



py的协程相比线程 , 没有线程安全问题 , 没有race condition(用户负责调度) , 没有线程维护开销


### 如何理解JS的作用域，作用域的底层实现

为什么这段代码输出为1 ？
```
let a = 1
function foo() {
    console.log(a)
}
function too() {
    let a = 2
    foo()
}
too() // 1  为什么这里输出为1 ？

```

答：Javascript有作用域链，在当前作用域找不到的时候会往上层作用域继续查找 , 并且作用域链在定义（编码阶段）的时候就已经固定，而不是运行时才确定，因此foo的上一层作用域是全局作用域 。于此相反的是this是在运行时才确定。




参考：

[讲清楚之javascript作用域](https://segmentfault.com/a/1190000014980841)

[javascript作用域底层作用分析](https://blog.csdn.net/weixin_39877717/article/details/80448975)



### Promise的出现背景，设计思想

回调函数实现异步 ， 有回调地狱的问题

观察者模式 / 发布订阅模式. VS   Promise

promise的reslove就是观察者模式中的 事件分发 ， then就是正常执行结果的Listener ，catch就是reject的Listener

由于只能触发一次，还增加了一个status状态字段来避免重发触发的情况(pending , fulfilled , rejected)

链式调用功能，则使用队列来保存不同阶段执行的结果 ， 分别有正常执行的promise结果队列，和错误执行的promise结果队列

每个promise中的data字段保存异步结果

rejected状态用于错误处理 ， 不同的状态相当于事件分类

参考：

[从设计模式角度分析Promise：手撕Promise并不难](https://juejin.im/post/5cc7d8f56fb9a031f61d838e)

[解读Promise内部实现原理](https://juejin.im/post/5a30193051882503dc53af3c)

### this的值到底是谁？

和作用域不同，this是运行时才确定的

在开启严格模式进行js编程时 ，当函数作为方法被对象拥有并调用时 this 指向该对象，否则 this 为 undefind

当函数是以箭头函数方式创建的，此时的 this 指向箭头函数执行时的宿主函数的上下文

```
function foo () {
    let that = this
    let too = () => {
        console.log(this === that) // true
    }
    too()
}
foo()
```



参考:

[讲清楚之 javascript中的this](https://segmentfault.com/a/1190000015038826)




### 扩展学习

 闭包可以理解为缓存了当前作用域的一个函数变量

 [讲清楚之 javascript 参数传值](https://segmentfault.com/a/1190000015105086)

 [讲清楚之 javascript 对象继承](https://segmentfault.com/a/1190000015171937)

 [讲清楚之 javascript 变量对象](https://segmentfault.com/a/1190000015077971)
 
 


# 用Go的 buffered channel 实现对象池

是否使用对象池的权衡: channel的线程安全实现,锁机制带来的开销 vs 对象创建开销

放回时满了抛出error
获取时引入超时机制

关于超时 , slow response 和quick failure , 惊群的危害
 
# 为什么php 的foreach比for快
 
 因为php数组的底层是哈希结构 , bucket之间是一条双向链表 , foreach的时候直接取next的值即可 , 同时foreach是根据插入顺序遍历的 , 如果需要有序遍历 , 只能用for循环
 
 for比foreach慢的原因 , 可能是多出的一个条件判断造成的 , 因此能用foreach就尽量用foreach
 
 
 
# php 的foreach小陷阱
 
 ```
 $arr = array('1','2','3');
 foreach($arr as &$v){
 }
 foreach($arr as $v){
 }
 var_dump($arr);
 
 //结果
 array
   0 => string '1' (length=1)
   1 => string '2' (length=1)
   2 => &string '2' (length=1)
 ```
 
 
 
 预期结果应该是1 2 3 , 为什么是1 2 2 呢 ?
 

 因为在第一个循环的最后一次执行中 , 由于$v为引用变量，所以$v 与 $arr[ 2 ] 指向了同一个地址空间 , 于是在第二次循环的时候 , 不断将值赋给arr[2]的zval结构体,  arr的变化如下:
 
 ```
 [ 1 , 2 , 1 ]
 
 [ 1 , 2 , 2 ]
 
 [ 1, 2 , 2 ]
 ```
 
 # unset
 
 unset只删除php层引用变量的映射关系 , 并不会清空底层zval变量的内存空间
 
 
 
 # 为什么要小心使用PHP的引用
 
 PHP采用的复制机制是“引用计数，写时复制”，也就是说，即便在PHP里复制一个变量，最初的形式从根本上说其实仍然是引用的形式，只有当变量的内容发生变化时，才会出现真正的复制。

# 引用和指针的区别



在C中指针是一个存储内存地址的变量

在PHP中引用则是在存储变量键值对的哈希表中增加一对键值对 , 值指向同一个结构体 , 因此几乎不产生额外空间 



写时复制

写时改变



https://blog.csdn.net/JathamJ/article/details/73189194

https://blog.csdn.net/chen1083376511/article/details/82721749



## [C语言里的三种参数传递 ](nowamagic.net/academy/detail/1205552) - 现代魔法学院

在开始之前，请务必看这篇文章： [漫谈C指针：参数传递的三道题目](http://www.nowamagic.net/librarys/veda/detail/2126)

我们都知道：C语言中函数参数的传递有：[值传递](http://www.nowamagic.net/academy/tag/值传递)、地址传递、引用传递这三种形式。题一为值传递，题二为地址传递，题三为引用传递。

值传递大家都应该很清楚，作为参数的变量，传递给函数执行后，自己的变量值是不变的。它（实参）仅仅只是把值赋给了形参，自己实际上是没有参与函数运算的，参与的是形参，这个就是参数的值传递。

地址传递呢？地址传递跟值传递也没有什么不同，值传递是把变量的值传递给形参去参与函数运算，而地址传递则把变量的地址传递给形参去参与函数运算。当然，如果函数改变了变量地址的值，实参的值也会变化的。

最后是[引用传递](http://www.nowamagic.net/academy/tag/引用传递)，也是我们本小节的重点。引用传递的调用方式与值传递一样的，而形参则引用了实参，在函数里头操作的是实参，而不是像上面两种操作形参那样。也就是函数是直接修改实参的值了。



# php变量和变量名是如何映射的

变量是一个zval结构体 , 通过哈希表键值对将变量名和变量映射起来 



参考: 

[如何获取一个变量的名字](http://www.laruence.com/2010/12/08/1716.html)
https://segmentfault.com/a/1190000018535960
[许铮的技术成长之路](https://segmentfault.com/blog/xuzheng_tech_growth)
https://www.jianshu.com/p/53fcf6128dcd


>每一个php变量都会由变量类型、value值、引用计数次数和是否是引用变量四部分组成

>而变量name是如何与变量容器关联起来的呢？其实也是使用了php的一个内部机制，即哈希表。每个变量的变量名和指向zval结构的指针被存储在哈希表内，以此实现了变量名到变量容器的映射

>其实不是的，变量存储也有作用域的概念。全局变量被存储到了全局符号表内，而局部变量也就是指函数或对象内的变量，则被存储到了活动符号表内（每个函数或对象都单独维护了自己的活动符号表。活动符号表的生命周期，从函数或对象被调用时开始，到调用完成时结束）

>我们这次主要讲一下手动销毁，即unset，每次销毁时都会将符号表内的变量名和对应的zval结构进行销毁，并将对应的内存归还到php所维护的内存池内（按内存大小划分到对应内存列表中）


# php 魔术方法 sleep wakeup

用在序列化和反序列化时, 对属性的操作

如文件资源 , 连接资源 , 在序列化时不需要,或没办法序列化

只需要保存文件的路径 , 连接的 ip,端口之类的 , 在反序列化时重新建立即可

## 参考
https://www.cnblogs.com/uduemc/p/4122156.html