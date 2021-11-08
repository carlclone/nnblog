# 数据库选错索引问题一例

## 问题描述
线上一个报表预览页面出现加载缓慢的情况

## 排查

- 检查 sql 日志,发现执行需要 10-20 秒
- explain 语句,可选索引有两个,campaign 和 date,命中索引 campaign , 但实际上使用 date 索引筛选数据量会更小
- 临时修改语句为 force index (date) , 耗时降到 1 秒以下
- 查看优化器执行决策 , 更详细的执行计划(physical plan),statistic ?
- 排查索引选错问题, campaign 的 不重复数/总数 要比date 大
- 优化器是 cost-based 的,统计方式为抽样统计,出现了不准确的情况
- 在业务空闲时间执行重新统计,修正不准确问题

## 参考资料
mysql 45 讲 - 选错索引
cmu 15445  , histogram 抽样统计法
cs186 2018
simpledb