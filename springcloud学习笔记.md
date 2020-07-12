## 1、Maven中的dependencyManagement和dependencies

dependencyManagement一般用于父工程从上往上找，好处是使全工程的jar包统一版本，便于管理
dependencyManagement只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖，不用声明版本号

## 2、SQL编写

create table payment(
	id int not null auto_increment comment 'ID',
	serial varchar(200) default '',
	primary key (id)
) engine=InnoDB auto_increment=1 default charset=utf8

insert into payment values(1, "aaaaa");

## 3、RestTemple

RestTemple提供了多种便捷访问远程Http服务的方法
是一种简单便捷的访问restful服务模板类，是spring提供的用于访问Rest服务
的客户端模板工具集

## 4、Eureka

服务注册：将服务信息注册进注册中心
服务发现：从注册中心上获取服务信息
实质：存key服务名 去value调用地址

1)、先启动Eureka注册中心
2)、启动服务提供者支付服务
3)、支付服务启动后会把自身的信息（比如服务地址以别名方式
注册进eureka）
4)、消费者服务在需要调用接口时，使用服务别名去注册中心获取
实际的RPC远程调用地址
5)、消费者获得调用地址后，底层实际是利用HttpClient技术实现
远程调用
6)、消费者获得服务地址后会缓存在本地JVM内存中，默认每间隔
30秒更新一次服务调用地址

7)、微服务RPC远程调用最核心的是什么？
高可用
解决办法：搭建Eureka注册中心集群，实现负载均衡+故障容错

8)、在RestTemplate上加@LoadBalanced进行负载均衡


Eureka自我保护：
保护模式主要用于一组客户端和Eureka Server之间的存在网络
分区场景下的保护。一旦进入保护模式，Eureka Server将会
尝试保护其服务注册表中的信息，不再删除服务注册表中
的数据，也就是不会注销任何微服务。

一句话：某时刻某一个微服务不可用了，Eureka不会立刻清理，
依旧会对该微服务的信息进行保存

## 5、Zookeeper

docker run -it --privileged=true --name zookeeper -p 2181:2181 -d zookeeper:latest

zookeeper上的服务节点是临时节点

## 6、CAP

C:强一致性
A：可用性
P：分区容错性
CAP理论关注粒度是数据，而不是整体系统设计的策略
CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，
可用性和分区容错性这三个需求。
CA:单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
CP：满足一致性，分区容忍性的系统，通常性能不是特别高
AP：满足可用性，分区容忍性的系统，通常可能对一致性要求低一些

## 7、Ribbon 负载均衡

Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上
获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术
一句话：负载均衡+RestTemplate调用

Ribbon在工作时分成两步：
第一步先选择EurekaServer，它优先选择在同一个区域内负载较少的server。
第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。
其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。

RestTemplate:常用方法
getForObject:返回对象为响应体中数据转化成的对象，基本上可以理解为JSON。
getForEntity:返回对象为ResponseEntity对象，包含了响应中的一些重要信息，
比如响应头，响应状态码，响应体等

postForObject
postForEntity


Ribbon核心组件IRule:
IRule接口主要的抽象类是AbstractLoadBalanceRule
根据特定算法中从服务列表中选取一个要访问的服务
方法有：
RoundRobinRule： 轮询（默认使用）
RandomRule：   随机
RetryRule：   先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
WeightedResponseTimeRule:对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
BestAvailableRule:会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
AvailabilityFilteringRule: 先过滤掉故障实例，再选择并发较小的实例
ZoneAvoidanceRule: 默认规则，复合判断server所在区域的性能和server的可用性选择服务器。

负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标，每次服务重启动
后rest接口计数从1开始。

从43集开始

## 8、OpenFeign（在消费端使用）

通过feign只需要定义服务绑定接口且以声明式的方式，优雅而简单的实现了服务调用。

OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务

OpenFeign整合了RestTemplate和Ribbon  所以具有负载均衡功能

## 9、Hystrix（一般在消费端）

当某个服务单元发生故障之后，通过断路器的故障监控，向调用方返回一个符合预期的，可处理的预备响应，而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间，不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

服务降级：

服务熔断：先服务降级->进而熔断->慢慢恢复调用链路

服务限流：

9.1、分布式系统面临的问题？

复杂分布式系统结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

9.2、@HystrixCommand报异常后如何处理？

一旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法

```java
@HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
```

系统设置超时峰值，超过峰值则调用fallbackMethod调用类中的指定方法

9.3、熔断机制是应对雪崩效应的一种微服务链路保护机制，当链路中的某个微服务出现异常时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后，恢复调用链路。

9.4、有个疑问，什么注册在Eureka上面的服务端down后，Eureka会没响应呢？等一段时间看看

可以修改Eureka客户端向服务端发送心跳的时间间隔，会发现当Eureka客户端down后，一段时间没有向Eureka服务端发送心跳，则会被剔除服务。从页面上可以看出！

```java
#Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
lease-renewal-interval-in-seconds: 1
#Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
lease-expiration-duration-in-seconds: 2
```



hutool工具包很强大，学习使用！

三个重要参数：快照时间窗、请求总数阈值、错误百分比阈值

```java
//在10秒内请求10次，并且失败率达到60则断路
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 失败率达到多少后跳闸
})
```

9.5、熔断类型

熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间），当打开时长达到所设时钟则进入半熔断状态

熔断关闭：熔断关闭不会对服务进行熔断

熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

## 10、GateWay(网关--也可以实现负载均衡)

SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。（看视屏了解Netty  韩顺平）

zuul底层采用的是servlet阻塞架构，gateway底层采用的spring webflux和netty结合的异步非阻塞架构

三大核心：Route(路由)、Predicate(断言)、Filter(过滤)

Route:路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。

Predicate：开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由。

Filter:使用过滤器，可以在请求被路由前或者之后对请求进行修改

## 11、Git相关操作

1. 在GitHub上新建一个仓库地址： http://github.com/......git

2. 在需要上传的文件夹目录下，运行 git  init 初始化git；

3. 运行git add 命令，将需要上传的内容放到暂存区，

4. 运行 git commit  -m "上传说明" 将暂存区的的内容放入到仓储里面；

5. 运行 git remote add origin http://github.com/......git,将仓储与GitHub仓库联系起来；

6. 运行 git push origin master 将文件提交到GitHub仓库里面，这样就完成了文件夹的提交。

## 12、Config(配置中心)

Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置

application.yml 是用户级的资源配置项

bootstrap.yml 是系统级的，优先级更高

```java
@RefreshScope //刷新  需要配合 
//curl -X POST 
"http://localhost:3344/actuator/bus-refresh"
```

## 13、BUS消息总线（基本和config配合使用）

bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道。

在微服务架构中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。

Bus动态刷新全局广播：
第一种：利用消息总线触发一个客户端/bus/refresh，而刷新所有客户端的配置。

第二种：利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，而刷新所有客户端的配置。

第二种设计思想比第一种更加合适，原因有：1、破坏了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责。2、破坏了微服务各节点的对等性。3、有一定的局限性。

加入MQ的主要作用：ConfigClient实例都监听MQ中同一个topic（默认是springCloudBus）。当一个服务刷新数据的时候，它会把这个信息放入到topic中，这样其他监听同一个topic的服务就能得到通知，然后去更新自身的配置

Bus动态刷新定点通知：指定具体某一个实例生效而不是全部

公式：http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}，/bus/refresh请求不再发送到具体的服务实例上，而是发给config server并通过destination参数类指定需要更新配置的服务或实例

例如：curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"

## 14、Stream(消息驱动)

Stream 通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离。

Stream中的消息通信方式遵循了发布-订阅模式，Topic主题进行广播

相关API：

Binder: 方便连接中间件，屏蔽差异

Channel: 通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置

Source/Sink: 从Stream发布消息就是输出，接受消息就是输入。

集群的消息消费者在消费消息可能存在两个问题：

1、消息重复消费和持久化 ：使用分组和持久化属性group

在Stream中处于同一个group中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。

不同组是可以全面消费的（重复消费），

同一组内会发生竞争关系，只有其中一个可以消费。

group属性可以使消息持久化，使程序不会丢失数据。

## 15、Sleuth（分布式请求链路跟踪）

问题：在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的服务节点调用来协同产生最后的请求结果，每一个前端请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延迟或错误都会引起整个请求最后的失败。

zipkin为页面展示

## 16、SpringCloud Alibaba Nacos

Nacos就是注册中心 + 配置中心的组合，是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos与其他注册中心的区别？

Nacos支持AP和CP模式的切换：C是所有节点在同一时间看到的数据是一致的；而A的定义是所有的请求都会收到响应。

何时选择使用何种模式？：一般来说，如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式。当前主流的服务如SpringCloud和Dubbo服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

如果需要再服务级别编辑或者存储配置信息，那么CP是必须的，K8S服务和DNS服务则适用于CP模式，CP模式下则支持注册持久化实例，此时则是以Raft协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。

Nacos Config 分类设计思想

Nacos集群和持久化配置：默认Nacos使用嵌入式数据库实现数据的存储，所以如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题的。为了解决这个问题，Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySql的存储

Nacos默认自带的是嵌入式数据库derby

Nginx加Nacos加Mysql  组成Nacos集群高可用

## 17、Sentinel

Sentinel采用的是懒加载机制，需要访问一次服务，才会在Sentinel的客户端页面出现该服务的服务名。

页面相关功能有实时监控、簇点链路、流控规则、降级规则、热点规则、系统规则、授权规则、集群流控、机器列表

1、实时监控：实时监控调用链路的情况

2、簇点链路：展示各个链路的调用情况

3、流控规则：资源名（调用的方法）、阈值类型（QPS每秒请求数、线程数）、单机阈值（每秒调用的最大值）、是否集群、流控模式（直接、关联、链路）、流控效果（快速失败、Warm Up、排队等待）

直接--快速失败：表示1秒钟内查询一次就OK，若超过次数1，就直接快速失败，报默认错误。

关联：当关联资源/testB的QPS阈值超过1时，就限流/testA的REST访问地址，当关联资源到阈值后限制配置好的资源名。

Warm Up（预热）：默认的冷加载因子为3，即请求QPS从 设置的QPS数除以3开始，经预热时长逐渐升至设定的QPS阈值。场景有：秒杀系统在开启的瞬间，会有很多流量上来，很有可能使系统宕机，预热方式就是为了保护系统，慢慢的把流量放进来，再慢慢的把阈值增长到设置的阈值。

4、降级规则：资源名、降级策略（RT、异常比例、异常数）、RT

RT（平均响应时间，秒级）：平均响应时间超出阈值且在时间窗口内通过的请求>=5，两个条件同时满足后触发降级，时间窗口期过后关闭断路器。

异常比例（秒级）：QPS>=5且异常比例（秒级统计）超过阈值时，触发降级；时间窗口结束后，关闭降级。

异常数（分钟级）：异常数（分钟统计）超过阈值时，触发降级；时间窗口结束后，关闭降级。

RT简述：1秒钟进来10个线程调用testD，我们希望200毫秒处理完本次任务，如果超过200毫秒还没处理完，在未来1秒钟的时间窗口内，断路器打开，微服务不可用。

5、热点key限流：

```java
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
//如果违背了Sentinel配置规则，则调用blockHandler指定的兜底方法

//自定义兜底方法
@SentinelResource(value = "customerBlockHandler",
blockHandlerClass = CustomerBlockHandler.class, //定义兜底方法的类
blockHandler = "handlerException2") //类中的方法


//@SentinelResource(value = "fallback",fallback = "handlerFallback") //fallback只负责业务异常
//@SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler只负责sentinel控制台配置违规
@SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",
 exceptionsToIgnore = {IllegalArgumentException.class})
//exceptionsToIgnore将会直接抛出配置的异常，而不会被fallback捕捉到，从而不会调用fallback的兜底方法
```

6、系统规则：阈值类型（LOAD、RT、线程数、入口QPS、CPU使用率）、阈值

## 18、Sentinel数据持久化（需要在nacos上配置）

[
	{
		"resource": "/rateLimit/byUrl",
		"limitApp": "default",
		"grade": 1,
		"count": 1,
		"strategy": 0,
		"controlBehavior": 0,
		"clusterMode": false
	}
]

resource：资源名称；limitApp：来源应用；grade：阈值类型，0表示线程数，1表示QPS；count：单机阈值；strategy：流控模式，0表示直接，1表示关联，2表示链路；controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；clusterMode：是否集群。

基于分布式的微服务架构

## 19、Seata（一ID+三组件模型）

分布式事务问题：单体应用被拆分成微服务应用，原来的三个模板被拆分成三个独立的应用，分别使用三个独立的数据源，业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证，但是全局的数据一致性问题没法保证。

一ID：全局唯一的事务ID

三组件：

TC-事务协调者

维护全局和分支事务的状态，驱动全局事务提交或回滚

TM-事务管理器

定义全局事务的范围：开始全局事务、提交或回滚全局事务。

RM-资源管理器

管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

Seata分布式事务处理过程：1、TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；2、XID在微服务调用链路的上下文传播；3、RM向TC注册分支事务，将其纳入XID对应的全局事务的管辖；4、TM向TC发起针对XID的全局提交或回滚决议；5、TC调度XID下管辖的全部分支事务完成提交或回滚请求。

本地事务@Transactional；全局事务@GlobalTransactional

142 