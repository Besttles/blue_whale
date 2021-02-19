# ForkJoin Framework

在通常情况下，java1中我们实现一个简单的java，concurrent应用，我们实现了Runnable接口创建了线程类，然后创建相同的线程对象，我们控制了线程的创建，执行！

在Java5的时候，我们有了一种新的管理线程的技术—线程池，我们只需要创建线程类，并创建Executor对象，我们就可以使用Executor对象来创建，执行和管理线程！

在java7的时候，我们在ExecutorService接口下新增加可额外的实现，面向于解决许多特殊的情况！

------

## 简介：

这个框架主要是面向于解决那些可以被分割成许多小任务的任务

![](/Users/biwh/Downloads/IMG_0246.jpg)

​                                       

框架主要基于以下两步操作：

1.fork：将任务分割成若干小任务并使用该框架执行

2.Join：等待所有的任务执行结束                  

*线程可以充分利用执行的时间，因此可以改善应用的执行效率*

   forkJoin的执行主要使用下面的两个类：

- forkJoinPool： 实现于ExecutorService接口，用于管理工人线程的信息和执行！
- forkJoinTask： 一个基类执行在forkJoinPool，提供机制来执行fork（）和join（）在一个任务里，来控制任务的状态，实现两个类，RecurviceAction用于没有返回值，RecurviceTask用于有返回值的任务！​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          

## RecursiveAction 

### 目标：

- 创建一个forkJoinPool 对象来执行任务

- 创建一个forkJoinTask 对象在forkJoinPool里面执行

### 具体实现的方式：

1. 使用默认的构造方法来创建forkJoinPool

2. 在任务中，你可以使用算法来选择是否将任务拆分

3. 你会使用加锁的方式来执行任务，当一个任务被分割成两个或者是多个子任务，子任务会等待任务全部结束

4. 任务不会返回任何结果，所以你要去使用基于RecursiveAction来实现

   **创建物品类**

```java
public class Product {
	private String name;
	private double price;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public double getPrice() {
		return price;
	}
	public void setPrice(double price) {
		this.price = price;
	}
}
```

创建多个物品

```java
	public class ProductListGenerator {

	public List<Product> generator(int size){
		List<Product> ret = new ArrayList<Product>();
	    for(int i=0 ; i<size ; i++) {
	    	Product product = new Product();
	    	product.setName("product"+i);
	    	product.setPrice(10);
	    	ret.add(product);
	    }
	    return ret;
	}
}
```

创建任务类并测试结果

```java
public class Task extends RecursiveAction {
	private static final long serialVersionUID = 1L;
	private List<Product> products;
	private int first;
	private int last;
	private double increment;

	public Task(List<Product> products, int first, int last, double increment) {
		super();
		this.products = products;
		this.first = first;
		this.last = last;
		this.increment = increment;
	}

	@Override
	protected void compute() {
		if(last - first<10) {
			updatePrices();
		}else {
			int middle = (last + first)/2;
			System.out.printf("task:pading task %s\n", getQueuedTaskCount());
			Task t1 = new Task(products,first,middle,increment);
			Task t2 = new Task(products,middle+1,last,increment);
			invokeAll(t1,t2);
		}
	}

	private void updatePrices() {
		for(int i = first; i<last;i++) {
			Product product = products.get(i);
			product.setPrice(product.getPrice()*(1+increment));
		}
	}
	
	public static void main(String[] args) {
		ProductListGenerator generator = new ProductListGenerator();
		List<Product> products = generator.generator(10000);
		Task task = new Task(products, 0, products.size(), 0.2);
		ForkJoinPool forkJoinPool = new ForkJoinPool();
		forkJoinPool.execute(task);
		do {
			System.out.printf("mainThread: ThreadCount: %d \n", forkJoinPool.getActiveThreadCount());
			System.out.printf("main:stealCount: %d \n",forkJoinPool.getStealCount());
			System.out.printf("mian:parallesime:%d \n", forkJoinPool.getParallelism());
			try {
				TimeUnit.MILLISECONDS.sleep(5);
			} catch (Exception e) {
				e.printStackTrace();
			}
		} while (!task.isDone());
		
		forkJoinPool.shutdown();
		
		if(task.isCompletedNormally()) {
			System.out.println("the task is completlly normal");
		}
		for(int i = 0;i<products.size();i++) {
			Product product = products.get(i);
			if(product.getPrice() != 12) {
				//System.out.printf("product : %s %f \n",product.getName(),product.getPrice());
			}
		}
		System.out.println("the end of the task!");
	}
}

```

结果：

```
task:pading task 0
task:pading task 2
task:pading task 0
task:pading task 1
task:pading task 1
task:pading task 0
task:pading task 1
task:pading task 0
task:pading task 1
task:pading task 0
the task is completlly normal
the end of the task!
```

### 怎么实现的？

在上面的例子中，你创建了ForkJoinPool和任务类在池中执行，你是用的是ForkJoinPool的默认构造器，所以将会使用默认的配置！创建的线程池的数量是基于你电脑中processor的数量，当任务池创建完成，那么这些线程就已经创建完成，并等待任务的执行@

你使用的是继承RecursiveAction类，所以没有返回结果。你使用你创建的构造器去创建任务对象，当我们必须去更新超过10个物品的时候，将所有的元素分割成两部分，并创建两个任务，我们用两个子任务去执行任务，用的是一个物品类集合**$List<Product> products = generator.generator(10000);$** 而不需要创建两个集合！

你可以使用$invokeAll()$ 去执行，这是个同步的调用，任务将会等待所有子任务执行完成，当任务正在等待他的子任务，那么workThread会去执行它锁等待的子任务， 通过这种方式，Fork/Join FrameWork比Runnable和Callable更加高效！

任务执行时的主线程时同步（asynchronous）执行的,你可以使用ForkJoinPool的方法去检查任务执行的状态！

即使ForkJoinPool类是用于执行ForkJoinTaskTask，你也可以执行Runnable和Callable对象，你或许也可以使用adapt()方法去接收一个Runnable或是callable对象，返回一个ForkJoinTask去执行任务！

## RecursiveTask

### 目标：

任务执行返回结果！使用继承ForkJoinTask类和实现Executor框架的Future接口！

如果你需要去执行比预定规模要大任务，就可以把任务分成多个子任务，并使用Fork/Join框架去，当它们执行完成，原来的任务包含每个子任务的执行结果，分组，然后返回最终结果，当任务执行结束，整个任务的执行结果就已经返回了！

### 具体实现的方式:

定义document类

```java
public class Document {

	private String words[] = {"the","hello","goodbay","packet","java","thread","pool","random","class","main"};
	public String [][] generateDocument(int numlines,int numWords,String word){
		
		int counter = 0;
		String document[][] = new String [numlines][numWords];
		Random random = new Random();
		
		for (int i = 0; i < numlines; i++) {
			for(int j=0;j<numWords;j++) {
				int index = random.nextInt(words.length);
				document[i][j] = words[index];
				if(document[i][j].equals(word)) {
					counter++;
				}
				
			}
		}
		System.out.printf("documentMock: the wprd appends %d times in the document \n",counter);
		return document;
	}
}

```

定义DocumentTask

```java
public class DocumentTask extends RecursiveTask<Integer>{

	private String[][] document;
	private int start,end;
	private String word;
	
	public DocumentTask(String[][] document, int start, int end, String word) {
		super();
		this.document = document;
		this.start = start;
		this.end = end;
		this.word = word;
	}

	@Override
	protected Integer compute() {
		int result = 0;
		if(end - start < 10) {
			result = processWord(document,start,end,word);
		}else {
			int mid = (start+end)/2;
			DocumentTask task1 = new DocumentTask(document, start, mid, word);
			DocumentTask task2 = new DocumentTask(document, mid, end, word);
			invokeAll(task1,task2);
			try {
				result = groupResults(task1.get(),task2.get());
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		return result;
	}
	private Integer groupResults(Integer integer, Integer integer2) {
		Integer result = 0;
      	result = integer + integer2;
      	return result;
	}
	private int processWord(String[][] document, int start, int end, String word) {

		List<LineTask> tasks = new ArrayList<LineTask>();
		for(int i=start;i<end;i++) {
			LineTask task = new LineTask(document[i],0,document[i].length,word);
			tasks.add(task);
			invokeAll(tasks);
		}
		int result = 0;
		for (int i = 0; i < tasks.size(); i++) {
			LineTask lineTask = tasks.get(i);
			try {
				result += lineTask.get();
			} catch (Exception e) {
				// TODO: handle exception
			}
		}		
		return result;
	}
}
```

定义真正在执行比较字符串的lineTask

```java
public class LineTask extends RecursiveTask<Integer>{

	private static final long serialVersionUID = 1L;

	private String line[];
	private int start,end;
	private String word;
	
	
	public LineTask(String[] line, int start, int end, String word) {
		super();
		this.line = line;
		this.start = start;
		this.end = end;
		this.word = word;
	}


	@Override
	protected Integer compute() {
		int result = 0;
		if(end - start < 100) {
			result = count(line,start,end,word);
		}else {
			int mid = (start+end)/2;
			LineTask task1 = new LineTask(line, start, mid, word);
			LineTask task2 = new LineTask(line, mid, end, word);
			invokeAll(task1,task2);
			try {
				result = groupResults(task1.get(),task2.get());
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		return result;
	}


	private int groupResults(Integer integer, Integer integer2) {
		Integer result = 0;
      	result = integer + integer2;
      	return result;
	}


	private int count(String[] line2, int start2, int end2, String word2) {
		
		int counter = 0;
		for(int i=start;i<end;i++) {
			if(line[i].equals(word2)) {
				counter++;
			}
		}
		try {
			//Thread.sleep(10);
		} catch (Exception e) {
			e.printStackTrace();
		}
  		return counter;
	}

	public static void main(String[] args) throws InterruptedException, ExecutionException {
		Date d = new Date();
		Document mock = new Document();
		String [][] document = mock.generateDocument(1000, 10000, "the");
		DocumentTask task = new DocumentTask(document, 0, 1000, "the");
		
		ForkJoinPool pool = new ForkJoinPool();
		pool.execute(task);
		Date d1 = new Date();
		long time1 = d1.getTime();
		long time = d.getTime();
		System.out.println("所需时间"+(time1-time)+"+++"+(task.get()));
//		do {
//			System.out.println("====================");
//			System.out.println("Main : parallelism %d \n"+pool.getParallelism());
//			System.out.println("Main : ActiveThreads %d \n"+pool.getActiveThreadCount());
//			System.out.println("Main : getStealCount %d \n"+ pool.getStealCount());
//			System.out.println("====================");
//		} while (!task.isDone());
		pool.shutdown();
		try {
			pool.awaitTermination(1, TimeUnit.DAYS);
		} catch (Exception e) {
			// TODO: handle exception
		}
		try {
			System.out.printf("Main : The word appears %s in the document" , task.get());
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

```

对比效率的方法

```java
public class Main {

	public static void main(String[] args) {
		Date d = new Date();
		Document mock = new Document();
		String [][] document = mock.generateDocument(1000, 10000, "the");
		
		int counter = 0;
		for (int i = 0; i < 1000; i++) {
			for(int j=0;j<10000;j++) {
				if(document[i][j].equals("the")) {
					counter++;
				}
			}
		}
		System.out.printf("documentMock: the wprd appends %d times in the document \n",counter);
		Date d1 = new Date();
		long time1 = d1.getTime();
		long time = d.getTime();
		System.out.println("所需时间"+(time1-time));
	}
}
```

结果

```
documentMock: the wprd appends 1001737 times in the document 
所需时间231+++1001737
Main : The word appears 1001737 in the document
```

### 怎么实现的？

DocumentTask：当传入的 $end-start>100$ ,就会把任务分割成两部分，这两部分使用递归的方式来继续判断使用哪种方式继续执行！

LineTask：$end-start>10$ 就会将任务分割成两部分，使用递归来实现！

当所有的任务执行完成，那么我们就可以使用$result.get()$ 来获取返回值！

### 异步地执行任务

当在线程池中执行任务的时候，我们是以同步的方式，知道任务返回结果的时候，程序才继续执行。但是我们也可以使用异步的方式去执行程序。当你使用异步的方式去执行任务，在任务执行的方法不会在任务执行完成后才会去返回结果，而是在任务会立刻返回结果，所以程序还可以继续执行

#### 具体的实现方式

```java
public class FolderProcessor extends RecursiveTask<List<String>>{
	private static final long serialVersionUID = 1L;
	private String path;
	private String extension;
	public FolderProcessor(String path, String extension) {
		super();
		this.path = path;
		this.extension = extension;
	}
	@Override
	protected List<String> compute() {
		List<String> list = new ArrayList<String>();
		List<FolderProcessor> tasks = new ArrayList<>();
		File file = new File(path);
		File[] contends = file.listFiles();
		if(contends != null) {
			for (int i = 0; i < contends.length; i++) {
				if(contends[i].isDirectory()) {
					FolderProcessor task = new FolderProcessor(contends[i].getAbsolutePath(), extension);
					task.fork();
					tasks.add(task);
					
				}else {
					if(checkFile(contends[i].getName())) {
						list.add(contends[i].getAbsolutePath());
						System.out.println(contends[i].getAbsolutePath());
					}
				}
			}
			if(tasks.size() > 50) {
				System.out.println("task"+tasks.size()+"run");
			}
			addResultFromTask(list,tasks);
		}
		return list;
	}
	private boolean checkFile(String name) {
		return name.endsWith(extension);
	}
	private void addResultFromTask(List<String> list, List<FolderProcessor> tasks) {
		for (FolderProcessor folderProcessor : tasks) {
			list.addAll(folderProcessor.join());
		}
	}
	public static void main(String[] args) {
	    ForkJoinPool pool = new ForkJoinPool();
	    FolderProcessor system = new FolderProcessor("/Users/biwh/Desktop/blue_whale", "md");
	    FolderProcessor apps = new FolderProcessor("/Users/biwh/Desktop/blue_whale/文档", "md");
	    FolderProcessor document = new FolderProcessor("/Users/biwh/Desktop/blue_whale/文档/concurrent", "md");
	    pool.execute(system);
	    pool.execute(apps);
	    pool.execute(document);
	    do {
	    	System.out.println("++++++++++++++++++++++++++");
			System.out.println("Main:paralleism:"+pool.getParallelism());
			System.out.println("Main:ActiveCount"+pool.getActiveThreadCount());
			System.out.println("Main:steal count"+pool.getStealCount());
			System.out.println("++++++++++++++++++++++++++");
		    try {
				TimeUnit.SECONDS.sleep(1);
			} catch (Exception e) {
				// TODO: handle exception
			}
	    } while ((!system.isDone())||(!apps.isDone())||(!document.isDone()));
	    pool.shutdown();
	    List<String> result;
	    result = system.join();
	    System.out.println("System size" + result.size());
	    result = apps.join();
	    System.out.println("System size" + result.size());
	    result = document.join();
	    System.out.println("System size" + result.size()); 
	}
}
```

结果：

```
/Users/biwh/Desktop/blue_whale/文档/Spring源码/spring-boot.md
/Users/biwh/Desktop/blue_whale/文档/Spring源码/spring-aop.md
/Users/biwh/Desktop/blue_whale/文档/Spring源码/spring-task.md
/Users/biwh/Desktop/blue_whale/文档/Spring源码/spring-transaction.md
/Users/biwh/Desktop/blue_whale/文档/Spring源码/spring-context.md
/Users/biwh/Desktop/blue_whale/文档/Spring源码/spring-mvc.md
task170run
/Users/biwh/Desktop/blue_whale/文档/有趣的文章/redis集锦/Redis持久性.md
System size21
System size20
System size3

```



#### 怎么实现的

这个例子可以对路径下的文件进行扫描，使用异步方式去执行任务，每一个任务都会去扫描一个路径，你知道的，扫描的文件只有两种方式：文件，或是文件夹，当扫描到文件夹的时候，我们会新创建一个task去执行，使用folk方法！

直到任务遍历了全部的目录，它会等待所有的任务使用jion() 方法去返回结果，这个方法在任务执行完成后会进行调用，并将所有的compute()方法返回的结果使用一个list集合来存储！

ForkJoinPool同样也可以使用同步的额方式执行，你是用execute()方法去执行已经创建好的任务，在调用程序中，你同样也是用shotdown()方法，和获取执行的状态！

### get()与join()的区别

**get()：**方法可以返回compute()方法执行完后的结果，或者等待直到任务完成

**get(long TIMEOUT,TimeUtil util)：**这个方法可以设置等待结果的时间，在时间到达后仍未返回结果，则返回null

join()：不可以被打断，如果你打断了join()方法的执行，会抛出InterruptorException

当get() 方法被打断，会抛出ExecutionException，如果程序中抛出了UncheckException，那么join()方法会抛出RuntimeException

### 在任务执行时抛出异常

checkedException：检查时异常

UncheckException：非检查时异常

#### 示例

```java
public class Task extends RecursiveTask<Integer>{

	private static final long serialVersionUID = 1L;
	private int array[];
	private int start,end;
	public Task(int[] array, int start, int end) {
		super();
		this.array = array;
		this.start = start;
		this.end = end;
	}
	@Override
	protected Integer compute() {
		
		System.out.println("the task is start");
		
		if(end-start < 10) {
			if((3>start) && (3<end)) {
				throw new RuntimeException("the task throw "+start+"to"+end);
			}
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}else {
			int mid = (end+start) /2;
			Task t1 = new Task(array, start, mid);
			Task t2 = new Task(array, mid, end);
			invokeAll(t1,t2);
		}
		System.out.println("the task to the end");
		return 0;
	}
}

```

调用方法：

```java
public class Main {
	public static void main(String[] args) {
		int array[] = new int[100];

		Task t = new Task(array, 0, 100);

		ForkJoinPool pool = new ForkJoinPool();

		pool.execute(t);

		pool.shutdown();

		try {
			pool.awaitTermination(1, TimeUnit.DAYS);
		} catch (Exception e) {
			e.printStackTrace();
		}

		if (t.isCompletedNormally()) {
			System.out.println(t.getException());
		}
		System.out.println(t.join());
	}
}

```

结果：

```yaml
the task is start
the task to the end
Exception in thread "main" the task to the end
the task to the end
the task to the end
the task to the end
the task to the end
the task to the end
the task to the end
the task to the end
the task to the end
java.lang.RuntimeException: java.lang.RuntimeException: the task throw 0to6
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)	
```

这个是在join等待输出的时候，抛出异常，程序停止执行！

## 取消任务

当你在forkJoinPool中执行ForkJoinTask对象时，你可以取消他们在他们执行之前，为了这个目的提供了cancel（）方法，当你想要取消一个任务，你需要遵循以下的原则：

1.当任务执行的时候，ForkJoinTask不提供任何方法去取消任务！

### 怎么实现取消任务：

定义数据生产方法

```java
public class ArrayGeneration {

	public int [] generationArray(int size) {
		int [] array = new int[size];
		Random random = new Random();
		for (int i = 0; i < size; i++) {
			array[i] = random.nextInt(10);
		}
		return array;
	}
}

```

定义任务管理类

```java
public class TaskManager {

	private List<ForkJoinTask<Integer>> tasks;

	public TaskManager(List<ForkJoinTask<Integer>> tasks) {
		super();
		this.tasks = tasks;
	}
	
	public TaskManager() {
		// TODO Auto-generated constructor stub
	}

	public void addTask(ForkJoinTask<Integer> task) {
		tasks.add(task);
	}
	
	public void cancleTasks(ForkJoinTask<Integer> cancleTask) {
		for (ForkJoinTask<Integer> forkJoinTask : tasks) {
			if(forkJoinTask != cancleTask) {
				forkJoinTask.cancel(true);
				((SearchNumberTask)forkJoinTask).writeCancelMessge();
			}
		}
	}
}

```

定义任务执行类

```java
public class SearchNumberTask extends RecursiveTask<Integer>{

	private int numbers[];
	private int start , end;
	private int number;
	private TaskManager manager;
	private final static int NOT_FOUND = -1;
	
	
	
	public SearchNumberTask(int[] numbers, int start, int end, int number, TaskManager manager) {
		super();
		this.numbers = numbers;
		this.start = start;
		this.end = end;
		this.number = number;
		this.manager = manager;
	}



	@Override
	protected Integer compute() {
		System.out.println("Task:"+start+"to"+end);
		int ret = 0;
		if(end-start>10) {
			ret = latchTask();
		}else {
			ret = lookForNum();
		}
		return ret;
	}
	private int lookForNum() {
		for(int i=start; i<end;i++) {
			if(numbers[i] == number) {
				manager.cancleTasks(this);
			}
			return i;
		}
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return NOT_FOUND;
	}
	private int latchTask() {
		int middle = (start+end)/2;
		SearchNumberTask task1 = new SearchNumberTask(numbers, start, middle, number, manager);
		SearchNumberTask task2 = new SearchNumberTask(numbers, middle, end, number, manager);
		//添加任务到任务列表中
		manager.addTask(task1);
		manager.addTask(task2);
		//执行任务
		task1.fork();
		task2.fork();
		int returnValue;
		returnValue = task1.join();
		if(returnValue == -1) {
			return returnValue;
		}
		
		returnValue = task2.join();
		return returnValue;
	}
	public void writeCancelMessge() {
		System.out.println("Task cancel from :=="+start+"==TO=="+end);
	}
}
```

测试类

```java
public class Main {
	public static void main(String[] args) {
		ArrayGeneration generator = new ArrayGeneration();
		int array[] = generator.generationArray(1000);
		List<ForkJoinTask<Integer>> tasks = new ArrayList<ForkJoinTask<Integer>>();
		TaskManager task = new TaskManager(tasks);
		ForkJoinPool pool = new ForkJoinPool();
		SearchNumberTask search = new SearchNumberTask(array, 0, 1000, 5, task);
		pool.execute(search);
		pool.shutdown();
		try {
			pool.awaitTermination(1, TimeUnit.DAYS);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("the task is finashed");
	}
}

```

我们取消的任务是任务还未执行，就是在我们分割任务的时候，当我们的任务还未执行fork（）条件的时候，我们就可以用cancel（）方法来取消任务的执行！