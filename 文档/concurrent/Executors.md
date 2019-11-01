# Executors

## ThreadFactory

> 研读过阿里巴巴的《阿里巴巴Java开发手册(终极版)》的同学应该都注意到，在其规范的“”并发”编程中，规定了不能显式地创建线程，只能通过创建线程池来管理线程，其目的是利用池化思想合理利用资源，但归根结底是通过实现ThradFactory接口创建，其中的方法使用了显式的构造器创建线程
> 

~~~java
        ExecutorService pool = Executors.newFixedThreadPool(3,Executors.defaultThreadFactory());
        Thread thread = new Thread(() -> {
        });
        pool.execute(thread);
        pool.shutdown();
~~~

线程工厂的接口

~~~java
public interface ThreadFactory {
    Thread newThread(Runnable r);
}
~~~

