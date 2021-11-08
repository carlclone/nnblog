lab2 opiterators

实现剩余的 OPIterator

一条 SQL 可以parse 为一颗 AST (Logical plan) , 再 walk 一遍 AST 对每个节点进行 decorate 后, 

可以理解为一棵树 , 也可以理解为 stream of tuple

聚合类的 iterator 会遍历完所有 tuple 再返回
其他类型的则调用一次 next 返回一个