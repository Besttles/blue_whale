# everyday

## Spring 的定时任务

@Scheduled(corn="  ")

首先启动一个线程池，默认实现是ThreadPoolTaskExecutor，初始化的时候先创建一个LinkedBlockingQueue阻塞队列，把需要执行的定时任务Runnable提交到线程池，由线程池来执行具体的操作！

**使用Quartz Scheduler**配置定时任务

```xml
<!--通知Spring容器通过定时任务来装配bean-->
<context:annotion-config. />
<!--扫描任务-->
<context:compent-scan base-package="com.stuff.task"   />
<!--启动注解的定时任务-->
<task:annotation-driven/>
<!--配置定时任务的线程池-->
<task:annotion-driven scheduler="myTask"/>
<task:scheduler id="myTask" pool-size="10">
```

执行一些每隔多长时间执行的任务时，会在上一次任务执行完之后隔一定时间会执行下一次任务！

------

## SpringBoot 的事务管理

事务的四要素：**ACID     原子性    一致性   隔离性    持久性**

[大佬讲事务](http://www.linkedkeeper.com/detail/blog.action?bid=1045)

关于事务管理器，不管是JPA还是JDBC等都实现自接口 PlatformTransactionManager 如果你添加的是 spring-boot-starter-jdbc 依赖，框架会默认注入 DataSourceTransactionManager 实例。如果你添加的是 spring-boot-starter-data-jpa 依赖，框架会默认注入 JpaTransactionManager 实例。

```java
@EnableTransactionManagement // 启注解事务管理，等同于xml配置方式的 <tx:annotation-driven />
@SpringBootApplication
public class ProfiledemoApplication {

    @Bean
    public Object testBean(PlatformTransactionManager platformTransactionManager){
        System.out.println(">>>>>>>>>>" + platformTransactionManager.getClass().getName());
        return new Object();
    }

    public static void main(String[] args) {
        SpringApplication.run(ProfiledemoApplication.class, args);
    }

```

注入Spring的事务管理器

```java
@EnableTransactionManagement
@SpringBootApplication
public class ProfiledemoApplication {

    // 其中 dataSource 框架会自动为我们注入
    @Bean
    public PlatformTransactionManager txManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean
    public Object testBean(PlatformTransactionManager platformTransactionManager) {
        System.out.println(">>>>>>>>>>" + platformTransactionManager.getClass().getName());
        return new Object();
    }

    public static void main(String[] args) {
        SpringApplication.run(ProfiledemoApplication.class, args);
    }
}
```

```java
@Component
public class DevSendMessage implements SendMessage {

    // 使用value具体指定使用哪个事务管理器
    @Transactional(value="txManager1")
    @Override
    public void send() {
        System.out.println(">>>>>>>>Dev Send()<<<<<<<<");
        send2();
    }

    // 在存在多个事务管理器的情况下，如果使用value具体指定
    // 则默认使用方法 annotationDrivenTransactionManager() 返回的事务管理器
    @Transactional
    public void send2() {
        System.out.println(">>>>>>>>Dev Send2()<<<<<<<<");
    }

}
```

指定特定的事务管理器，进行事务管理

*如果Spring容器中存在多个 PlatformTransactionManager 实例，并且没有实现接口 TransactionManagementConfigurer 指定默认值，在我们在方法上使用注解 @Transactional 的时候，就必须要用value指定，如果不指定，则会抛出异常。*

对于系统需要提供默认事务管理的情况下，实现接口 TransactionManagementConfigurer 指定。

对有的系统，为了避免不必要的问题，在业务中必须要明确指定 @Transactional 的 value 值的情况下。不建议实现接口 TransactionManagementConfigurer，这样控制台会明确抛出异常，开发人员就不会忘记主动指定。

```java
@Transactional(propagation = Propagation.REQUIRED)
```

- `REQUIRED` ：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- `SUPPORTS` ：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- `MANDATORY` ：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- `REQUIRES_NEW` ：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- `NOT_SUPPORTED` ：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- `NEVER` ：以非事务方式运行，如果当前存在事务，则抛出异常。
- `NESTED` ：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 `REQUIRED` 。

```tiki wiki
@Scope注解是什么
@Scope注解是springIoc容器中的一个作用域，在 Spring IoC 容器中具有以下几种作用域：基本作用域singleton（单例）、prototype(多例)，Web 作用域（reqeust、session、globalsession），自定义作用域
-------------------------
a.singleton单例模式 -- 全局有且仅有一个实例
b.prototype原型模式 -- 每次获取Bean的时候会有一个新的实例
c.request -- request表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效
d.session -- session作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效
e.globalsession -- global session作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义
--------------------------
@Scope注解的使用场景
几乎90%以上的业务使用singleton单实例就可以，所以spring默认的类型也是singleton，singleton虽然保证了全局是一个实例，对性能有所提高，但是如果实例中有非静态变量时，会导致线程安全问题，共享资源的竞争
当设置为prototype时：每次连接请求，都会生成一个bean实例，也会导致一个问题，当请求数越多，性能会降低，因为创建的实例，导致GC频繁，gc时长增加
```

 **3.3.3 spring事务实现机制**

   1 高层

​     比较好的方式有：1.基于持久层api的模板方法；2.使用具有事务工厂bean的本地ORM api；3使用代理管理本地资源工厂。

   2 底层

`      DataSourceUtils` (用作JDBC事务), `EntityManagerFactoryUtils` (用作JPA事务), `SessionFactoryUtils` (用作Hibernate事务),`PersistenceManagerFactoryUtils` (用作JDO事务)等等，

​    例如：在使用jdbc时，你可以不通过DataSource的getConnection()方法获取connection，而是使用以下方法获取：Connection conn = DataSourceUtils.getConnection(dataSource);
3 最底层
   TransactionAwareDataSourceProxy是事务的最底层，它代理了DataSource，并增加了spring管理事务功能。

------

## Spring AOP

[大佬讲AOP源码](http://www.linkedkeeper.com/1048.html)

AOP称为面向切面编程，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待，Struts2的拦截器设计就是基于AOP的思想，是个比较经典的例子。

一 AOP的基本概念

(1)Aspect(切面):通常是一个类，里面可以定义切入点和通知

(2)JointPoint(连接点):程序执行过程中明确的点，一般是方法的调用

(3)Advice(通知):AOP在特定的切入点上执行的增强处理，有before,after,afterReturning,afterThrowing,around

(4)Pointcut(切入点):就是带有通知的连接点，在程序中主要体现为书写切入点表达式

(5)AOP代理：AOP框架创建的对象，代理就是目标对象的加强。Spring中的AOP代理可以使JDK动态代理，也可以是CGLIB代理，前者基于接口，后者基于子类

二 Spring AOP

Spring中的AOP代理还是离不开Spring的IOC容器，代理的生成，管理及其依赖关系都是由IOC容器负责，Spring默认使用JDK动态代理，在需要代理类而不是代理接口的时候，Spring会自动切换为使用CGLIB代理，不过现在的项目都是面向接口编程，所以JDK动态代理相对来说用的还是多一些。

三 基于注解的AOP配置方式

1.启用@AsjectJ支持

在applicationContext.xml中配置下面一句:

```
<aop:aspectj-autoproxy />
```

2.通知类型介绍

(1)Before:在目标方法被调用之前做增强处理,@Before只需要指定切入点表达式即可

(2)AfterReturning:在目标方法正常完成后做增强,@AfterReturning除了指定切入点表达式后，还可以指定一个返回值形参名returning,代表目标方法的返回值

(3)AfterThrowing:主要用来处理程序中未处理的异常,@AfterThrowing除了指定切入点表达式后，还可以指定一个throwing的返回值形参名,可以通过该形参名

来访问目标方法中所抛出的异常对象

(4)After:在目标方法完成之后做增强，无论目标方法时候成功完成。@After可以指定一个切入点表达式

(5)Around:环绕通知,在目标方法完成前后做增强处理,环绕通知是最重要的通知类型,像事务,日志等都是环绕通知,注意编程中核心是一个ProceedingJoinPoint