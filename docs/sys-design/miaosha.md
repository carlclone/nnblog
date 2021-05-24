# 秒杀系统设计

秒杀系统 和 抢红包系统 , 抢票系统等类似

之前参加一个面试,有些问题引起了我的思考,比如秒杀用 redis 做预扣的时候,redis 挂了,数据没有持久化,返回给用户的情况却是秒杀成功了,造成了超卖的问题,这个之前在那篇系统设计里好像没有提到
https://github.com/GuoZhaoran/spikeSystem
光这一篇的解决方案还是不太够啊


## 资料
grokking the system design interview

github 秒杀系统