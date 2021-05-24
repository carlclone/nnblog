---
title: 知识点的reduce-积攒下来的编程技巧/方法
date: 2019-12-01
tags: 计算机基础
---

&nbsp;
<!-- more -->



# 尽早错误

> 已经被我频繁地使用于检查变量为空的情况

```
//嵌套写法
if (a == true) {
	if (b == true) {
		doSomething()
	}
}

//尽早错误写法
if (a!=true) {
	return
}
if (b!=true) {
	return
}
doSomething()
```

## 其他尽早错误的例子

> 10x程序员工作法27 尽早暴露问题

1.重构的作者Martin flower写的文章 - [fail fast](https://www.martinfowler.com/ieeeSoftware/failFast.pdf)

2.尽早检查参数错误,而不是让已经错误的参数继续往后流入数据库,导致难以分析错误来源

3.数据库配置,缺少重要配置的时候直接报错,而不是设立默认值,比如最大连接数默认值10,配置项拼写错误,读取不到配置,然后在线上造成异常的缓慢,并可能耗费几天的时间来排查

4.做需求,做项目,有问题尽早提出来,越拖越难解决





# 避免使用否定条件

> 这条不是硬性要求,但能提高代码可读性

```
if (!exist) {
 doSomething()
}


if (notExist) {
	doSomething()
}
```





# 二进制位运算标志位表示状态

> 用二进制位运算,而不是数组遍历,在资源紧缺的时候性能优化还是很重要的,然后其实可读性也非常好

参考Redis源码中的例子: Client结构体里的flags属性

```
if (server.maxidletime &&
        // 不检查作为从服务器的客户端
        !(c->flags & REDIS_SLAVE) &&    /* no timeout for slaves */
        // 不检查作为主服务器的客户端
        !(c->flags & REDIS_MASTER) &&   /* no timeout for masters */
        // 不检查被阻塞的客户端
        !(c->flags & REDIS_BLOCKED) &&  /* no timeout for BLPOP */
        // 不检查订阅了频道的客户端
        dictSize(c->pubsub_channels) == 0 && /* no timeout for pubsub */
        // 不检查订阅了模式的客户端
        listLength(c->pubsub_patterns) == 0 &&
        // 客户端最后一次与服务器通讯的时间已经超过了 maxidletime 时间
        (now - c->lastinteraction > server.maxidletime))
    {
        redisLog(REDIS_VERBOSE,"Closing idle client");
        // 关闭超时客户端
        freeClient(c);
        return 1;
    }
```



参考[MDN]([https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#%E7%A4%BA%E4%BE%8B](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#示例))的例子

```
var FLAG_A = 1; // 0001
var FLAG_B = 2; // 0010
var FLAG_C = 4; // 0100
var FLAG_D = 8; // 1000

var mask = FLAG_A | FLAG_B | FLAG_D; // 0001 | 0010 | 1000 => 1011

// 如果我们有 cat
if (flags & FLAG_C) { // 0101 & 0100 => 0100 => true
   // do stuff
}

// 如果我们有 bat 或者 cat 至少一个
// (0101 & 0010) || (0101 & 0100) => 0000 || 0100 => true
if ((flags & FLAG_B) || (flags & FLAG_C)) {
   // do stuff
}

// 如果我们有 bat 或者 cat 至少一个
var mask = FLAG_B | FLAG_C; // 0010 | 0100 => 0110
if (flags & mask) { // 0101 & 0110 => 0100 => true
   // do stuff
}

# 或运算可以设置标志位
// 我们有 cat 和 duck
var mask = FLAG_C | FLAG_D; // 0100 | 1000 => 1100
flags |= mask;   // 0101 | 1100 => 1101

# 与运算可以清楚标志位
// 我们没有 ant 也没有 cat
var mask = ~(FLAG_A | FLAG_C); // ~0101 => 1010
flags &= mask;   // 1101 & 1010 => 1000

德摩根定律：
~(FLAG_A | FLAG_C)  euqals ~FLAG_A & ~FLAG_C


# 异或运算，相同的置为0 ，不同的为1 ， 这个例子中用于切换标志位

// 如果我们以前没有 bat ，那么我们现在有 bat
// 但是如果我们已经有了一个，那么现在没有了
// 对 cat 也是相同的情况
var mask = FLAG_B | FLAG_C;
flags = flags ^ mask;   // 1100 ^ 0110 => 1010

# 非运算翻转
// entering parallel universe...
flags = ~flags;    // ~1010 => 0101

```



## 延伸

创建掩码，转换为二进制的算法实现：



```
function createMask () {
  var nMask = 0, nFlag = 0, nLen = arguments.length > 32 ? 32 : arguments.length;
  for (nFlag; nFlag < nLen; nMask |= arguments[nFlag] << nFlag++);
  return nMask;
}
var mask1 = createMask(true, true, false, true); // 11, i.e.: 1011
var mask2 = createMask(false, false, true); // 4, i.e.: 0100
var mask3 = createMask(true); // 1, i.e.: 0001
// etc.

alert(mask1); // 打印 11
```



```
function arrayFromMask (nMask) {
  // nMask 必须介于 -2147483648 和 2147483647 之间
  if (nMask > 0x7fffffff || nMask < -0x80000000) { 
    throw new TypeError("arrayFromMask - out of range"); 
  }
  for (var nShifted = nMask, aFromMask = []; nShifted; 
       aFromMask.push(Boolean(nShifted & 1)), nShifted >>>= 1);
  return aFromMask;
}

var array1 = arrayFromMask(11);
var array2 = arrayFromMask(4);
var array3 = arrayFromMask(1);

alert("[" + array1.join(", ") + "]");
// 打印 "[true, true, false, true]", i.e.: 11, i.e.: 1011
```



```
function createBinaryString (nMask) {
  // nMask must be between -2147483648 and 2147483647
  for (var nFlag = 0, nShifted = nMask, sMask = ""; nFlag < 32;
       nFlag++, sMask += String(nShifted >>> 31), nShifted <<= 1);
  return sMask;
}

var string1 = createBinaryString(11);
var string2 = createBinaryString(4);
var string3 = createBinaryString(1);

alert(string1);
// 打印 00000000000000000000000000001011, i.e. 11
```



# 不要在一开始就过度抽象，先思考模型的最小状态

> 用现实世界的模型创建类 ,容易创造出OOP 怪兽

eloquent js ，OOP一章



#  学习开源代码该如何入手？



> 消息队列课 09

## 先看文档, 而且确保是最新的 , 中文可能不是最新的

1.quick start 先搭建起来, 跑一些例子

2.Introduction 看基本概念 , 名词术语 ,可能会有专门的basic concepts 文档 ,然后是使用场景 , use cases , ecosystem  , 在ecosystem中查看是否有和当前需求匹配的案例 , 如果有 , 可以少走很多弯路



## 然后问自己这些问题

这个项目是干什么的？
能解决哪些问题？
适合在哪些场景使用？
有哪些功能？
如何使用？



> 并不是说之后就能直接看源码 , 因为程序代码是一个网状结构, 不适合顺序阅读 , 容易被网缠住



> 大部分开源项目诞生的方式 , 某个科学家发表某篇论文被大厂看到了 , 实现了出来 , 这篇论文就是这个项目的灵魂 , 比如kafka的



## 之后, 带着问题去读源码 ,  最好是带着问题的**答案**去读



kafka的官网有 design和implementation章节 , 可以寻找实现原理/答案 , 但是一般的项目好像没有吧 ...

可以把更多的精力放在实现细节上, 而不是被全局绕晕 , 形成一个阅读单元 , 点线面最后串成网



## 最后,最重要的是，把主要的流程用流程图或者时序图画出来，把重点的算法、原理用文字写出来。



## 作业

课后，建议你找一个你熟悉的开源项目，可以是消息相关的，也可以是无关的开源项目，确定一个问题，用这节课中讲到的“带着问题和答案去读源码”的方法，去读一点源码。然后，最重要的是，把主要的流程用流程图或者时序图画出来，把重点的算法、原理用文字写出来。



## 下面的评论

实在不行你还可以跟着别人的源码解析文章去读源码。 (我就是这样....)


# push 代码前的Checklist

1.可能出错的代码加try catch避免扩大影响范围
2.检查副作用
3.影响范围估算
4.幂等性检查

