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

```xml
<aop:aspectj-autoproxy />
```

2.通知类型介绍

(1)Before:在目标方法被调用之前做增强处理,@Before只需要指定切入点表达式即可

(2)AfterReturning:在目标方法正常完成后做增强,@AfterReturning除了指定切入点表达式后，还可以指定一个返回值形参名returning,代表目标方法的返回值

(3)AfterThrowing:主要用来处理程序中未处理的异常,@AfterThrowing除了指定切入点表达式后，还可以指定一个throwing的返回值形参名,可以通过该形参名

来访问目标方法中所抛出的异常对象

(4)After:在目标方法完成之后做增强，无论目标方法时候成功完成。@After可以指定一个切入点表达式

(5)Around:环绕通知,在目标方法完成前后做增强处理,环绕通知是最重要的通知类型,像事务,日志等都是环绕通知,注意编程中核心是一个ProceedingJoinPoint

## Spring AOP 2

AOP常用的实现方式有两种，**一种是采用声明的方式来实现（基于XML），一种是采用注解的方式来实现（基于AspectJ）**。

首先复习下AOP中一些比较重要的概念：

**Joinpoint（连接点）：**程序执行时的某个特定的点，在Spring中就是某一个方法的执行 。
**Pointcut（切点）：**说的通俗点，spring中AOP的切点就是指一些方法的集合，而这些方法是需要被增强、被代理的。一般都是按照一定的约定规则来表示的，如正则表达式等。切点是由一类连接点组成。 
**Advice（通知)：**还是说的通俗点，就是在指定切点上要干些什么。 
**Advisor（通知器)：**其实就是切点和通知的结合 。

**一、基于XML配置的Spring AOP**

采用声明的方式实现（在XML文件中配置），大致步骤为：配置文件中配置pointcut, 在java中用编写实际的aspect 类, 针对对切入点进行相关的业务处理。

业务接口：

```java
package com.spring.service;

public interface IUserManagerService {
    //查找用户
    public String findUser();
    
    //添加用户
    public void addUser();
}
```

业务实现：

```java
package com.spring.service.impl;

import com.spring.service.IUserManagerService;

public class UserManagerServiceImpl implements IUserManagerService{
    
private String name;
    
    public void setName(String name){
        this.name=name;
    }
    
    public String getName(){
        return this.name;
    }
    
    public String findUser(){
        System.out.println("============执行业务方法findUser,查找的用户是："+name+"=============");
        return name;
    }
    
    public void addUser(){
        System.out.println("============执行业务方法addUser=============");
        //throw new RuntimeException();
    }
}
```

切面类：

```java
package com.spring.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;

public class AopAspect {
    
    /**
     * 前置通知：目标方法调用之前执行的代码
      * @param jp
     */
    public void doBefore(JoinPoint jp){
        System.out.println("===========执行前置通知============");
    }
    
    /**
     * 后置返回通知：目标方法正常结束后执行的代码
      * 返回通知是可以访问到目标方法的返回值的
      * @param jp
     * @param result
     */
    public void doAfterReturning(JoinPoint jp,String result){
        System.out.println("===========执行后置通知============");
        System.out.println("返回值result==================="+result);
    }
    
    /**
     * 最终通知：目标方法调用之后执行的代码（无论目标方法是否出现异常均执行）
      * 因为方法可能会出现异常，所以不能返回方法的返回值
      * @param jp
     */
    public void doAfter(JoinPoint jp){
        System.out.println("===========执行最终通知============");
    }
    
    /**
     * 
     * 异常通知：目标方法抛出异常时执行的代码
      * 可以访问到异常对象
      * @param jp
     * @param ex
     */
    public void doAfterThrowing(JoinPoint jp,Exception ex){
        System.out.println("===========执行异常通知============");
    }
    
    /**
     * 环绕通知：目标方法调用前后执行的代码，可以在方法调用前后完成自定义的行为。
      * 包围一个连接点（join point）的通知。它会在切入点方法执行前执行同时方法结束也会执行对应的部分。
      * 主要是调用proceed()方法来执行切入点方法，来作为环绕通知前后方法的分水岭。
      * 
     * 环绕通知类似于动态代理的全过程：ProceedingJoinPoint类型的参数可以决定是否执行目标方法。
      * 而且环绕通知必须有返回值，返回值即为目标方法的返回值
      * @param pjp
     * @return
     * @throws Throwable
     */
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable{
        System.out.println("======执行环绕通知开始=========");
         // 调用方法的参数
        Object[] args = pjp.getArgs();
        // 调用的方法名
        String method = pjp.getSignature().getName();
        // 获取目标对象
        Object target = pjp.getTarget();
        // 执行完方法的返回值
        // 调用proceed()方法，就会触发切入点方法执行
        Object result=pjp.proceed();
        System.out.println("输出,方法名：" + method + ";目标对象：" + target + ";返回值：" + result);
        System.out.println("======执行环绕通知结束=========");
        return result;
    }
}
```

Spring配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    
    <!-- 声明一个业务类 -->
    <bean id="userManager" class="com.spring.service.impl.UserManagerServiceImpl">
        <property name="name" value="lixiaoxi"></property>
    </bean>  
    
     <!-- 声明通知类 -->
    <bean id="aspectBean" class="com.spring.aop.AopAspect" />

    <aop:config>
     <aop:aspect ref="aspectBean">
        <aop:pointcut id="pointcut" expression="execution(* com.spring.service.impl.UserManagerServiceImpl..*(..))"/>
        
        <aop:before method="doBefore" pointcut-ref="pointcut"/> 
        <aop:after-returning method="doAfterReturning" pointcut-ref="pointcut" returning="result"/>
        <aop:after method="doAfter" pointcut-ref="pointcut" /> 
        <aop:around method="doAround" pointcut-ref="pointcut"/> 
        <aop:after-throwing method="doAfterThrowing" pointcut-ref="pointcut" throwing="ex"/>
      </aop:aspect>
   </aop:config>
</beans>
```

测试类：

```java
package com.spring.test;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.spring.service.IUserManagerService;

public class TestAop {

    public static void main(String[] args) throws Exception{
        
        ApplicationContext act =  new ClassPathXmlApplicationContext("applicationContext3.xml");
         IUserManagerService userManager = (IUserManagerService)act.getBean("userManager");
         userManager.findUser();
         System.out.println("\n");
         userManager.addUser();
    }
}
```

 **二、使用注解配置AOP**

采用注解来做aop, 主要是将写在spring 配置文件中的连接点写到注解里面。

业务接口和业务实现与上边一样，具体切面类如下：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```java
package com.spring.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class AopAspectJ {
    
    /**  
     * 必须为final String类型的,注解里要使用的变量只能是静态常量类型的  
     */  
    public static final String EDP="execution(* com.spring.service.impl.UserManagerServiceImpl..*(..))";
    
    /**
     * 切面的前置方法 即方法执行前拦截到的方法
      * 在目标方法执行之前的通知
      * @param jp
     */
    @Before(EDP)
    public void doBefore(JoinPoint jp){
        
        System.out.println("=========执行前置通知==========");
    }
    
    
    /**
     * 在方法正常执行通过之后执行的通知叫做返回通知
      * 可以返回到方法的返回值 在注解后加入returning 
     * @param jp
     * @param result
     */
    @AfterReturning(value=EDP,returning="result")
    public void doAfterReturning(JoinPoint jp,String result){
        System.out.println("===========执行后置通知============");
    }
    
    /**
     * 最终通知：目标方法调用之后执行的通知（无论目标方法是否出现异常均执行）
      * @param jp
     */
    @After(value=EDP)
    public void doAfter(JoinPoint jp){
        System.out.println("===========执行最终通知============");
    }
    
    /**
     * 环绕通知：目标方法调用前后执行的通知，可以在方法调用前后完成自定义的行为。
      * @param pjp
     * @return
     * @throws Throwable
     */
    @Around(EDP)
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable{

        System.out.println("======执行环绕通知开始=========");
        // 调用方法的参数
        Object[] args = pjp.getArgs();
        // 调用的方法名
        String method = pjp.getSignature().getName();
        // 获取目标对象
        Object target = pjp.getTarget();
        // 执行完方法的返回值
        // 调用proceed()方法，就会触发切入点方法执行
        Object result=pjp.proceed();
        System.out.println("输出,方法名：" + method + ";目标对象：" + target + ";返回值：" + result);
        System.out.println("======执行环绕通知结束=========");
        return result;
    }
    
    /**
     * 在目标方法非正常执行完成, 抛出异常的时候会走此方法
      * @param jp
     * @param ex
     */
    @AfterThrowing(value=EDP,throwing="ex")
    public void doAfterThrowing(JoinPoint jp,Exception ex) {
        System.out.println("===========执行异常通知============");
    }
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

spring的配置:

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    
    <!-- 声明spring对@AspectJ的支持 -->
    <aop:aspectj-autoproxy/>    
    
    <!-- 声明一个业务类 -->
    <bean id="userManager" class="com.spring.service.impl.UserManagerServiceImpl">
        <property name="name" value="lixiaoxi"></property>
    </bean>   
    
     <!-- 声明通知类 -->
    <bean id="aspectBean" class="com.spring.aop.AopAspectJ" />

</beans>
```

测试类：

```java
package com.spring.test;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.spring.service.IUserManagerService;

public class TestAop1 {
    public static void main(String[] args) throws Exception{
        
        ApplicationContext act =  new ClassPathXmlApplicationContext("applicationContext4.xml");
         IUserManagerService userManager = (IUserManagerService)act.getBean("userManager");
         userManager.findUser();
         System.out.println("\n");
         userManager.addUser();
    }
}
```

测试结果与上面相同。

**注意：**
1.环绕方法通知，环绕方法通知要注意必须给出调用之后的返回值，否则被代理的方法会停止调用并返回null，除非你真的打算这么做。           
2.只有环绕通知才可以使用JoinPoint的子类ProceedingJoinPoint，各连接点类型可以调用代理的方法，并获取、改变返回值。

**补充：**
1.<aop:pointcut>如果位于<aop:aspect>元素中，则命名切点只能被当前<aop:aspect>内定义的元素访问到，为了能被整个<aop:config>元素中定义的所有增强访问，则必须在<aop:config>下定义切点。
2.如果在<aop:config>元素下直接定义<aop:pointcut>，必须保证<aop:pointcut>在<aop:aspect>之前定义。<aop:config>下还可以定义<aop:advisor>，三者在<aop:config>中的配置有先后顺序的要求：首先必须是<aop:pointcut>，然后是<aop:advisor>，最后是<aop:aspect>。而在<aop:aspect>中定义的<aop:pointcut>则没有先后顺序的要求，可以在任何位置定义。
.<aop:pointcut>：用来定义切入点，该切入点可以重用；
.<aop:advisor>：用来定义只有一个通知和一个切入点的切面；
.<aop:aspect>：用来定义切面，该切面可以包含多个切入点和通知，而且标签内部的通知和切入点定义是无序的；和advisor的区别就在此，advisor只包含一个通知和一个切入点。
3.在使用spring框架配置AOP的时候，不管是通过XML配置文件还是注解的方式都需要定义pointcut"切入点"
例如定义切入点表达式 execution(* com.sample.service.impl..*.*(..))
execution()是最常用的切点函数，其语法如下所示：
整个表达式可以分为五个部分：
(1)、execution(): 表达式主体。
(2)、第一个*号：表示返回类型，*号表示所有的类型。
(3)、包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包，com.sample.service.impl包、子孙包下所有类的方法。
(4)、第二个*号：表示类名，*号表示所有的类。
(5)、*(..):最后这个星号表示方法名，*号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数。

## ImmutableList

ImmutableList是一个不可变、线程安全的列表集合，它只会获取传入对象的一个副本，而不会影响到原来的变量或者对象，如下代码：

```java
    int a = 23;
    ImmutableList<Integer> list = ImmutableList.of(a, 12);
    System.out.println(list);
```

ImmutableList创建不可变对象有两种方法，一种是使用静态of方法，另外一种是使用静态内部类Builder。

```java
    //获取一个空的不可变集合对象
    ImmutableList<String> list1 = ImmutableList .<String>of();
    //获取一个有一个元素的不可变集合对象
    ImmutableList<String> list2 = ImmutableList .<String>of("12");
    //获取一个有两个元素的不可变集合对象
    ImmutableList<String> list3 = ImmutableList .<String>of("12","23");
```
```java
    List<String> list4 = new ArrayList<String>();
    list4.add("1");
    list4.add("2");
    list4.add("3");
    //copy数组list4的一个副本
    List<String> list5 = ImmutableList .<String>copyOf(list4);
```
```java
    //使用内部类的方式
    ImmutableList<Integer> list = ImmutableList .<Integer>builder()
                                                    .add(12)
                                                    .add(23)
                                                    .add(34)
                                                    .build();
```

## 使用@RequestBody来接受Json类型的数据

数据接收时会自动转型

~~~json
{
	"a":"a",
	"b":[{
		"name":"c"
	},{
		"name":"d"
	}]
	}
}
~~~

这样我们使用Map来接收时会在b里面存储LinkedList

~~~json
{
	"a":"a",
	"b":{
		"c":"c",
		"d":"d"
	}
}
~~~

这种情况就会在b下面存储LinkedHashMap

也可以使用对象类型进行嵌套存储

## 注意：mysql时间的表示法

```java
Date date=new Date(System.currentTimeMillis()/1000L*1000L);
```

这段代码为什么这么写？

原来，MySQL存储datetime时会将毫秒四舍五入到秒级。

举例：1572858619499和1572858619500存入MYSQL的值分别为`2019-11-04 17:10:19`和`2019-11-04 17:10:20`，500是临界点，大于等于500入一秒，小于500舍。

