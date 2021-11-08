lab3 query optimization

查询优化有两个大方向:
启发式, 基于经验的方式设定规则 , 如 predicate 为 1=0 , 可以直接返回而不需要继续往下执行
Cost-based model , 对所有表的数据进行扫描, 基于一些预估算法对 selectivity 快速计算, 从多个逻辑一致的不同执行顺序中选出开销最低的Physical Plan

Histogram 方式的数据预估:
一个柱子表示一个范围内 tuple 的数量 , 并且 tuple 在柱子内部是 uniformly distributed


调用顺序:
Parser 解析 SQL 生成 LogicalPlan
Parser 调用 TableStats 生成每个 Table 的统计预估数据
    TableStats 生成 IntHistogram 和每个表的 tuple 数
Parser 调用 estimateSelectivity 获得单个 predicate 的 selectivity
Parser 调用 JoinOptimizer 获得 join 的优化执行顺序
Parser 调用 xxx 生成可以执行的OPIterator (Physical Plan)

执行关系图:


如何获得不同数据库实现的执行计划和统计数据
MySQL :
EXPLAIN

MariaDB:


2.2.4 Join Cardinality
Join Exercise 3: Join Cost Estimation
使用公式计算
2.3 Join Ordering
Join Exercise 4: Join Ordering
遍历所有排列 , 找到best cost 的一个

在 lab3 要求实现 JoinOptimizer 的
estimateJoinCost
estimateJoinCardinality
orderJoins

estimateJoinCost 算出join大致的 cost (cpu+io cost)
estimateJoinCardinality 估算 join 产生的 tuple 数 The cardinality of a join is the number of tuples produced by the join.


Join Optimization视频资料:
cmu15-445 Query Planning & Opt  I , II  https://www.bilibili.com/video/BV1q741127SQ?p=15
cs186 https://www.bilibili.com/video/BV13a411c7Qo?p=131
mit6.830 https://www.bilibili.com/video/BV15i4y157F4?p=4

文字资料:
simpledb -lab3.md
