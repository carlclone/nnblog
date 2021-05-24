# Symfony 的 Lock Component


今天在闲逛 Symfony 官网看组件文档的时候无意中看到了这个 Symfony 官方的锁组件, 以前在公司的项目里大部分都是一个满足最基本需求的实现, 现在有了这样一个规范, 好好学习一下, 以后就不用踩坑, 重复造轮子了

在大部分项目中看到对锁的封装, 都是使用的 redis 字符串类型, 资源的名称作为 key,随机值作为 value , 再加个最基本的锁超时时间设置 , 防止程序异常导致资源一直被锁住, 然后是加锁解锁

这样在大部分场景都是可用的 , 但在长时间执行的程序有可能会出现下面的问题 :

进程 1 抢锁,执行任务,锁超时释放
进程 2 拿到锁,执行任务,爆发第一个问题(线程安全)
进程 1 执行完毕,解锁,第二个问题(不是自己的锁)
进程 2 解锁时候出错(锁已经被进程 1 释放)



先解决问题 2 : 解锁的时候验证 value, 只有是自己的锁才能解开
解决问题 1 : 尽可能保证程序执行时间<锁超时时间,但又不能过长 , 将任务拆分成几个执行周期,对锁进行续租 . 但这个不能完全避免 , 问题 2 还是可能发生 , 此时当发现锁已经不在自己手上的时候,也回滚,确保数据一致性

官网给出了比较好的实现方式

  	**拆分成几个短的执行周期,续租**

```php
$lock = $factory->createLock('charts-generation', 30);

if (!$lock->acquire()) {
    return;
}
try {

    while (!$finished) {
        // perform a small part of the job.

        // renew the lock for 30 more seconds.
        $lock->refresh();
    }
} finally {
    $lock->release();
}	
```

发现锁不在自己手上了,回滚, 自此问题 1 和问题 2 都解决了

```php
// If we cannot acquire ourselves, it means some other process is already working on it
if (!$lock->acquire()) {
    return;
}

$this->beginTransaction();

// Perform a very long process that might exceed TTL of the lock


if ($lock->isAcquired()) { 
    //严格来说上下这俩句不是原子性的,还存在很小很小的概率在提交之前的很短的时间超时,锁被其他进程获取了
    // Still all good, no other instance has acquired the lock in the meantime, we're safe
    $this->commit();
} else {
    // Bummer! Our lock has apparently exceeded TTL and another process has started in
    // the meantime so it's not safe for us to commit.
    $this->rollback();
    throw new \Exception('Process failed');
}		
```





## 其他功能

序列化锁,  可以让同一个锁在不同进程之间传递, 这个使用场景好像比较少

阻塞取锁,  symfony lock 支持多种不同的底层存储实现, 有的底层存储原生支持阻塞取锁, 像 redis 不支持的话, 内部会有一个类似加了 for 循环, 直到取到锁才执行程序逻辑的实现

读写锁, reader/writer 问题, 读读可以同时得到锁, 写读, 写写,读写只能拿到一个, 并且还有读者写者优先级问题

锁拥有者检查, 上面说的问题 2

多个锁底层存储实现

Flock文件系统锁


## 结尾

Symfony 有很多优秀,规范的组件, 以后工作中遇到非业务的功能实现先参考[这里](https://symfony.com/doc/current/components/index.html)



## 参考资料

[Symfony Components](https://symfony.com/doc/current/components/index.html)

[Lock Component](https://symfony.com/doc/current/components/lock.html#blocking-locks)

左耳听风 - 锁

极客时间