# 面试记录

笔试题和网上的题目一样
技术面问的也不深,而且才一轮技术面

印象深刻的问题,引起我进一步思考的问题


# 面试准备

二叉树排序 X

不会死机遍历一亿个节点的二叉树  (模拟栈)  X

设计组织架构的数据表 , 伪代码打印组织架构 X

员工关系表, 打印员工关系表 , 遍历某个员工的下属 X

SQL 反模式一书 X

ID  NAME  PARENT_ID  , 每行就是一个节点

打印员工关系表就是 DFS , 带层级的 , 打印一颗树 , 多维数组

findChild()

优化方法的话就是冗余+缓存了

打印某个员工的关系表 (树状图)

sql 统计某个领导下有多少小弟 X

dfs bfs 遍历二叉树 X

sql exist  , 返回 true 或 false , 什么场景用? youtube 那个视频
https://www.youtube.com/watch?v=zfgJ3ZmAgNw&list=PLLchAlP_W0GfYWjv6Off6lfk4xNe_l-QB&index=1

https://stackoverflow.com/questions/52083057/why-do-we-need-where-exists-operator-in-sql

一对多关系,想判断是否存在,用 exist 查的话存在一条就停止了,当然也可以用 limit 1

Well, in your example, EXISTS is more efficient. The subquery needs to read all matching rows in order to do the count.

EXISTS, by contract, can (and does!) stop at the first matching row.

You could argue that the SQL engine could identify this situation. However, that would be very, very difficult for more complex queries.

In contrast to your viewpoint, I find EXISTS to be more useful than IN or correlated subqueries with aggregations. It is the preferred method for expressing this logic.

缓存过期,缓存雪崩,缓存穿透 x

重新叙述问题: 是什么 , 会产生什么问题, 如何解决 

问设计模式  X



问框架 X

