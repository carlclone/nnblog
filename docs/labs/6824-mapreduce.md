# MapReduce

[github 地址](https://github.com/carlclone/MIT-6.824-LAB-2020/tree/lab3/src/mr)

## 前言

6.824课程的第一个 Lab, 目标是一个可以把大任务拆分成若干个可以并行运行的小任务, 自动分配到不同的机器上运行的系统, 将数据工程师的心智负担解放出来

有了这样一个系统, 负责编写 map 和 reduce 程序的数据工程师们就不再需要关注拆分后的文件如何存储, 如何分配到不同的机器上运行, 如何将运行结果最终 merge, 也不需要关心异常问题的处理, 例如某个 worker 宕掉, 某个 task 运行缓慢的处理, 可以专注在真正关心的程序的开发上



```
mapreduce

input is (already) split into M files
  Input1 -> Map -> a,1 b,1
  Input2 -> Map ->     b,1
  Input3 -> Map -> a,1     c,1
                    |   |   |
                    |   |   -> Reduce -> c,1
                    |   -----> Reduce -> b,2
                    ---------> Reduce -> a,2

解决的问题:
让用户只需要关心 map 和 reduce
隐藏:
sending app code to servers
  tracking which tasks are done
  moving data from Maps to Reduces
  balancing load over servers
  recovering from failures

MapReduce single-handedly made big cluster computation popular.
  - Not the most efficient or flexible.
  + Scales well.
  + Easy to program -- failures and data movement are hidden.
  These were good trade-offs in practice.
  We will see some more advanced successors later in the course.
  Have fun with the lab!
  
```

## mapreduce 例子

课程里的例子, 我看的时候有些细节不是很明白, 这次回顾时我尝试描述一个实际应用的例子

模型是函数式编程中的 map , reduce 函数

对应到单词统计这个例子就是

```
完整的文章 : 你好,张三,很高兴遇见你. 你好,李四.

对文章进行切割, 让文章的 map 操作可以在多个机器并行执行

文章块1 : 你好啊,张三,很高兴遇见你啊.

文章块2: 你好,李四.

map 函数的过程把文章转换成了 k,v 键值对
map 任务 1 结果:

单词=>计数
你好 => 1
啊=>1
张三=>1
很=>1
高兴=>1
....
啊=>1

map 任务 2 结果:
单词=>计数
你好=>1
李四=>1

为了 reduce 任务能独立,并行地执行,每个 reduce 之间没有关联关系, 将哈希值相同的 key 被放到同一个 reduce 任务中

reduce 任务 1:
你好=>1
你好=>1
张三=>1
...

reduce 任务 2:
啊=>1
啊=>1
高兴=>1
李四=>1
...

reduce 任务 1 结果:
你好=>2
张三=>1
...

reduce 任务 2 结果:
啊=>2
高兴=>1
李四=>1
...

merge + sort 任务结果:
你好=>2
啊=>2
张三=>1
李四=>1
...

```


## 系统开发

主要的模型是单 master 多 worker , 我这里使用了worker 主动去 master 拉任务的模式, 任务的类型有 map 任务和 reduce 任务, reduce 任务只有等所有 map 任务完成后才会生成和分配

这里定义了一个规范, map 的输入是(filename,contents) 所有任务都可以被map 转换成 key,value 形式 , reduce 的规范是 (key,values) , values 是所有 key 相同的 value 聚合后的数组


## 功能列表

**已实现的功能:**

- master 初始化map 任务列表

- worker 向 master 取 map 任务执行, 通知 master 任务执行完成

- worker向 master 取 reduce 任务, 通知执行完成

- master 定期检查任务超时情况,重新分配

**待实现的功能:**

- 新增一项 merge 任务, 将 reduce 任务的输出合并

**Fault tolerance 相关**:

- worker 心跳检查
- master 宕机 (多 master, 选举 , 状态持久化和recovery) 

Master崩溃的解决方案 , 会在Lab2和3 接触到 , 要对重要的状态实时持久化 , 并对维护的task列表实行checkpoint+从这个点开始的log 

- 论文里提到的其他点





## 问题记录

**多个 worker 同时向一个中间文件写入, 产生了输入错乱的情况**

- 一个 worker 的中间输出应该写入自己单独的文件 (mr-map 任务 id-reduce 任务 id, 而不是 mr-reduce 任务 id), 然后在 reduce 的时候读取这些文件, 虽然说这个问题没有触发过, 是在后面阅读代码的时候发现的, 但想到 web 框架中的写日志也是单例 , 加文件锁写的, 问题应该还是存在的

- 在不要求性能的情况下, 加个文件锁也能解决问题....

 

**worker 取任务的时候未加锁, 导致一个任务重复运行两次**

- worker 去 master 取任务的时候要加锁, 避免一个任务被分配给了多个 worker

**超时任务重分配问题记录** 

- lab 要求实现 worker 执行超时后重新分配的机制, 原 worker 可能还在执行, 所以在写入中间文件的时候文件如果已存在, 就直接跳过

- 全部worker都在等某个很慢的 worker 执行的情况, 增加超时, 分配给其他人做

**worker 异常崩溃,中间文件不完整** 

- 文件的写入要先写入临时文件,完成后再重命名, 防止各种异常情况造成文件结果不完整

**写入异常慢** 

- 写入文件之前先把要写的数据按文件聚合好后, 再写入, 减少write 函数的系统调用

还有许多未记录忘记了的...很伤, 还有些现在看起来很低级的错误




## 结尾

在 v2ex 逛的时候发现了这门课程, 这是第一个 Lab, 主要的收获是编码能力得到了提升

fault-tolerance 是本课程一个重要的点





## 参考资料

[关于锁的文章](https://pdos.csail.mit.edu/6.824/labs/raft-locking.txt)

[lab1](https://pdos.csail.mit.edu/6.824/labs/lab-mr.html)

[lec1](https://pdos.csail.mit.edu/6.824/notes/l01.txt)

[lec1 paper](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)

[go tour faq](https://pdos.csail.mit.edu/6.824/papers/tour-faq.txt)


## 后续扩展学习

golang so 后缀文件的插件形式?

阅读对比其他人的实现

[同时 append 一个文件 会有问题吗](https://www.oschina.net/question/142976_70051)






## reading 摘抄


The worker should put intermediate Map output in files in the current directory, where your worker can later read them as input to Reduce tasks.    

The worker implementation should put the output of the X'th reduce task in the file mr-out-X.



* The master, as an RPC server, will be concurrent; don't forget to lock shared data.
* Use Go's race detector, with go build -race and go run -race. test-mr.sh has a comment that shows you how to enable the race detector for the tests.
* Workers will sometimes need to wait, e.g. reduces can't start until the last map has finished. One possibility is for workers to periodically ask the master for work, sleeping with time.Sleep() between each request. Another possibility is for the relevant RPC handler in the master to have a loop that waits, either with time.Sleep() or sync.Cond. Go runs the handler for each RPC in its own thread, so the fact that one handler is waiting won't prevent the master from processing other RPCs.
* The master can't reliably distinguish between crashed workers, workers that are alive but have stalled for some reason, and workers that are executing but too slowly to be useful. The best you can do is have the master wait for some amount of time, and then give up and re-issue the task to a different worker. For this lab, have the master wait for ten seconds; after that the master should assume the worker has died (of course, it might not have).
* To test crash recovery, you can use the mrapps/crash.go application plugin. It randomly exits in the Map and Reduce functions.
* To ensure that nobody observes partially written files in the presence of crashes, the MapReduce paper mentions the trick of using a temporary file and atomically renaming it once it is completely written. You can use ioutil.TempFile to create a temporary file and os.Rename to atomically rename it.
* test-mr.sh runs all the processes in the sub-directory mr-tmp, so if something goes wrong and you want to look at intermediate or output files, look there.



## 命令记录

```
go build -buildmode=plugin ../mrapps/wc.go

$ rm mr-out*
$ go run mrmaster.go pg-*.txt

$ go run mrworker.go wc.so   运行多个worker
```


