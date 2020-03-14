# SpringCould学习

### SpringCould自我保护机制

某时刻某一个微服务不可用，Eureka不会立刻清理，依旧会对该微服务的信息进行保存！

> 自我保护机制是应对网络异常的安全保护措施，他的架构哲学就是保留所有的微服务，不盲目注销任何健康的微服务，使用自我保护机制，可以使Eureka集群更加的健壮！

禁用自我保护机制：`eureka.server.enable-self-preservation = false`

### 服务发现

使用DiscoveryClient来获取微服务列表

在主启动类中添加`@EnableDiscoveryClient`注解 

### 集群配置

修改defaultZone来配置集群环境

### CAP

C：强一致性 A：可用性 P：分区容错性   三者之中只能选两个

### zookeeper和SpringCould的区别

zookeeper是CP

SpringCould是AP

### Ribbon负载均衡

LoadBalance负载均衡，集中式负载均衡（偏硬件F5），进程式负载均衡（ngnix集成到消费方）

从eureka集群中发现可用的服务列表，消费者根据负载均衡算法来进行负载均衡

在provider中暴露的实例名相同，做负载均衡

Ribbon的负载均衡算法

Ribbon自定义算法

自定义的算法类不能在@CompontScan的路径上

### Feign

`@FeignClient`注解是调用微服务放在interface类上

feign默认集成了ribbon，使用轮训实现客户端的负载均衡

### Hystrix

熔断器：服务降级

> 多个微服务调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C也调用其他的微服务，这就是所谓的“扇出”。如果扇出的某个微服务响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统的崩溃，也就是所谓的雪崩！

熔断器本身是一种开关装置。