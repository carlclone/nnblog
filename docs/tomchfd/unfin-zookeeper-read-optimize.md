# zookeeper 对读的优化

之前写了一版 raft 和 raftkv, 被人问起读是怎么实现的, 读也是通过 append log 到 raft 里,保证一致性的, 接着问怎么优化, 我只说了一个分片的方案

常见的优化手段, 降低精确性 

zookeeper 也是通过降低一致性实现的高性能读 

通常多个 client 


和之前写的 raft kv 读做对比, 学习一下 zookeeper 的 read index


