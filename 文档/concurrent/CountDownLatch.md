# CountDownLatch

CountDownLatch**这个类能够使一个线程等待其他线程完成各自的工作后再执行**。

​	例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。

CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

## 代码示例

定义抽象类，进行组件的加载的父类！

```java
package com.xiaour.spring.boot.startUpService;
import java.util.concurrent.CountDownLatch;
public abstract class HealthyCheck implements Runnable{

	private CountDownLatch countDownLatch;
	private boolean _serviceUP;
	private String serviceName;
	
	public HealthyCheck(CountDownLatch countDownLatch, String serviceName) {
		super();
		this.countDownLatch = countDownLatch;
		this.serviceName = serviceName;
	}
	@Override
	public void run() {
		try {
			if(vertifyService()) {
				_serviceUP = true;
			}
						
		} catch (Exception e) {
			// TODO: handle exception
			_serviceUP = false;
		} finally {
            if(countDownLatch != null) {
                //等待其他的线程结束
            	countDownLatch.countDown();
            }
		}
	}
	public boolean isStartUP() {
		return _serviceUP;
	}
	//定义丑行方法，子类去重写这个方法
	abstract boolean vertifyService();

}

```

继承父类，重写vertifyService的方法，判断服务是否启动

```java
package com.xiaour.spring.boot.startUpService;

import java.util.concurrent.CountDownLatch;

public class DatabaseCheck extends HealthyCheck{
	public DatabaseCheck(CountDownLatch countDownLatch) {
		super(countDownLatch, "Database sevice");
		// TODO Auto-generated constructor stub
	}
	@Override
	boolean vertifyService() {
		if(1>0) {
			try {
				Thread.currentThread().sleep(1000);
				return true;
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		return false;
	}
}
```

```java
package com.xiaour.spring.boot.startUpService;

import java.util.concurrent.CountDownLatch;

public class NetWorkService extends HealthyCheck{
	public NetWorkService(CountDownLatch countDownLatch) {
		super(countDownLatch, "network Service");
	}
	@Override
	boolean vertifyService() {
		if(1>0) {
			try {
				Thread.currentThread().sleep(3000);
				return true;
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		return false;
	}
}

```

定义工具类，来调用其服务启动的方法，来使用countdownlatch来进行加载全部的心线程类

```java
package com.xiaour.spring.boot.startUpService;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import javassist.bytecode.analysis.Executor;
public class StartUpService {

	private final static StartUpService service = new StartUpService();
	private static CountDownLatch latch;
	private boolean isStartUP;
	
	private static List<HealthyCheck> check;
	public boolean startUP() {
		return isStartUP;
	}
	
	public static boolean chechService() throws InterruptedException {
		
		latch = new CountDownLatch(2);
		check = new ArrayList<>();
		check.add(new NetWorkService(latch));
		check.add(new DatabaseCheck(latch));
		//创建线程池
		ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(check.size());
		for (final HealthyCheck healthyCheck : check) {
			newFixedThreadPool.execute(healthyCheck);
		}
        //等待全部线程结束
		latch.await();
		for (HealthyCheck healthyCheck : check) {
			if(healthyCheck.isStartUP()) {
				continue;
			}else {
				return false;
			}
		}
		return true;
	}	
}

```

测试方法

```java
   public static void main(String ... args){
        //预先加载组件
    	try {
    		boolean isStart = StartUpService.chechService();
    		if(isStart) {
    			System.out.println("已经加载组件");
    		}else {
    			System.out.println("未加载组件");
    		}
		} catch (Exception e) {
			e.getMessage();
		}
        SpringApplication.run(Application.class, args);
    }
```

## 源码分析

构造函数：

```java
        Sync(int count) {
            setState(count);
        }
```

CountDown方法

```java
    public void countDown() {
        sync.releaseShared(1);
    }
```

```java
    private void doReleaseShared() {
        /*
         * Ensure that a release propagates, even if there are other
         * in-progress acquires/releases.  This proceeds in the usual
         * way of trying to unparkSuccessor of head if it needs
         * signal. But if it does not, status is set to PROPAGATE to
         * ensure that upon release, propagation continues.
         * Additionally, we must loop in case a new node is added
         * while we are doing this. Also, unlike other uses of
         * unparkSuccessor, we need to know if CAS to reset status
         * fails, if so rechecking.
         */
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

await方法

```java
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

```

```java
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }

```

```java
    /**
     * Acquires in shared interruptible mode.
     * @param arg the acquire argument
     */
    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

```

```java
    private void cancelAcquire(Node node) {
        // Ignore if node doesn't exist
        if (node == null)
            return;

        node.thread = null;

        // Skip cancelled predecessors
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;

        // predNext is the apparent node to unsplice. CASes below will
        // fail if not, in which case, we lost race vs another cancel
        // or signal, so no further action is necessary.
        Node predNext = pred.next;

        // Can use unconditional write instead of CAS here.
        // After this atomic step, other Nodes can skip past us.
        // Before, we are free of interference from other threads.
        node.waitStatus = Node.CANCELLED;

        // If we are the tail, remove ourselves.
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            // If successor needs signal, try to set pred's next-link
            // so it will get one. Otherwise wake it up to propagate.
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }

```

