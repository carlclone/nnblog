# 关于业务代码的框架

对于我参与过的大部分项目来说,业务代码的分层如下基本就够用了 (可测试和易于修改)

```
Controller
|
Service        <-   Third Party Library
|
Repository
```

在 Java 的 Spring boot 框架中, Service 和 Repository 的注入是通过注解的形式实现的, 在 PHP 中也有通过类反射的方式自动注入的实现, 所以只需要在 Controller 的构造函数中, 或者 Controller 的方法中注入 Service 即可 (注入由HTTP Kernel 在负责)

## 一些问题

但是在实际开发中会有一些问题

- 一个 Service 可能会依赖许多的 Repository, 通过构造函数的方式注入会有额外的开销, 因为可能这一次调用只使用了一部分的 Repository
- ServiceA 依赖 ServiceB, ServiceB 又依赖 ServiceA, 造成循环依赖, 自动反射注入的时候出现问题 

第一个问题

- 可能不是问题,实例化的开销并没有想的那么严重

- 可以在容器中定义实例化描述, Service 依赖于容器, 在使用到该 Repository 的时候才从容器中取

- 可以使用 RepositoryManager/Factory 的形式, 在使用到该 Repository 的时候由Manager 实例化

两种形式差不多, 都是想办法延迟加载, 但对开发者不友好, IDE 无法自动提示了

第二个问题
 
- 两个 Service 相互依赖, 就得考虑是不是职责的划分有问题了, 这时候也许可以引入第三个Service, 类似微服务中的"聚合服务"的概念 
- 和 Repository 一样引入一个 Manger 的角色, 类比微服务中的注册中心

## 小想法记录

一些第三方的库,通常是通过 ServiceProvider 注入到容器中, 然后我们使用的时候从容器中去取,如果一定要在容器中显式定义实例化过程, 我们可以参考第三方库, 将 Repository 的实例化写入到一个 RepositoryServiceProvider 中, Service 的实例化写入到 ServiceServiceProvider 中, 起到一个分类的作用

## 结尾

之前在构建 PHP 框架的时候想到了这个问题, 参与过的项目基本都没有在业务这层遵循好依赖注入的原则, 这次好好总结了一下

并且在之前学习杨波老师的微服务课程的过程中, 学习了一些架构设计的理念, 在单体应用内部的设计中也是通用的, 每个 Service 类的设计就类似微服务中的一个服务, 做好服务的拆分很重要, 这要求开发人员非常了解业务, 我个人对业务知识这块一直是比较无感的, 希望通过这个案例让自己重视起来

## 参考资料

[Spring Boot与Kubernetes云原生微服务实践](https://time.geekbang.org/course/intro/100031401)

[Product service and repository](https://www.youtube.com/watch?v=GCl8nhEMSv8)

之前看到的企业级框架 java 的 [tiny framework](http://www.tinygroup.org/docs/04b2cd7aa22b427fa3dc8fd0fea52f5f) , 有很多值得参考的地方,如数据级权限, 分库分表, 太多了

Symfony 有一套 Service/Repository 的 Bundles, 有空看看怎么设计的