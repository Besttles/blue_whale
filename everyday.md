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

