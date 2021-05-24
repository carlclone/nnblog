# Clock Replacer 回顾

CMU 15445 课程的第一个 lab, 实现的是一个非常简单的缓存置换策略, Clock Replacer, 它的思路是:

replacer 是一个缓存置换策略类, 里面会存放将要被淘汰掉的 frame 的 ref , 和 buffer pool 里的 ref 一一对应

> frame 相当于数组里的一个位置, page 相当于数组里存放的元素. 实际上 frame 是一个 page 大小的内存空间,一般为 4kb , 多个 frame 组成一个 buffer pool

当一个 frame 的 page 被 unpin 掉时, 就会被加入到 replacer 里作为可选淘汰位置

当一个 frame 的 page 被 pin 时, 会从 replacer 里移除

当空间不足需要找一个位置淘汰时, clock replacer 的策略是淘汰最早被 unpin 的一个, 相当于先进先出 , 但这里的淘汰操作却不是 O(1)的, 最坏的情况下需要转一圈才能找到最早被 unpin的 frame

为什么不像循环队列一样, 使得淘汰操作复杂度为 O1 呢, 因为这里有更频繁的 unpin (入队) 和 pin 操作(中途出队), 为了保证两者的复杂度为 O1 , 牺牲了淘汰操作的复杂度

当然也可以像 LRU Cache 一样以空间换时间, 新增一个哈希表来存储每个 frame 的位置, 使得两个操作时间复杂度都为 O1

同时作业要求保证线程安全, 这里 unpin , pin 和淘汰(victim) 操作都会改动到 clock replacer 里的数据结构, 三者都加上 latch 互斥即可


## 未完待续

1 看了 2018 年的 Lab1 也是一个缓存置换的作业, 但要求实现 LRU

2 加上配图

## 资料

[Lab1](https://15445.courses.cs.cmu.edu/fall2019/project1/)

[2019 fall 视频](https://www.bilibili.com/video/BV1q741127SQ?from=search&seid=5317067461007025707)
