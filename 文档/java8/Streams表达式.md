# Streams

## 什么是Streams

Streams是一种用来操作集合中数据以一种声明的方式！你可以把它想象成一个集合的迭代器，除此之外，Streams可以用平行的方式进行，你不需要去写一些线程并发的代码！

```java
public class StreamOpration {
	public static void main(String[] args) {
	  List<Dishes> menu = Arrays.asList(new Dishes(100, "红烧排骨","food"),
	    		new Dishes(20, "可乐","drink"),
	    		new Dishes(40, "米饭","food"),
	    		new Dishes(120, "鸡柳","food"),
	    		new Dishes(200, "满汉全席","food"));
		List<String> collect = menu.stream()
				.filter(d -> d.getCalorious() > 40)
				.sorted(Comparator.comparing(Dishes::getCalorious))
				.map(Dishes::getName)
				.collect(Collectors.toList());
		collect.forEach(System.out::println);
	}
}
```

如果想用并行的方式来执行这段代码，只需要将***stream()*** 换成***parallelStream()*** 

```java
List<String> collect2 = menu.parallelStream()
				.filter(d -> d.getCalorious() > 40)
				.sorted(Comparator.comparing(Dishes::getCalorious))
				.map(Dishes::getName)
				.collect(Collectors.toList());
		collect2.forEach(System.out::println);
```

你可能在想当我们使用parallelStream()的时候，发生了什么呢？我们会使用多少线程来啊完成任务呢？这种方式的好处是什么呢？

这种方式我们用声明的方式来写代码，我们根据自己的需要去实现一个操作

**Chaining stream operations forming a stream pipeline**

![IMG_0269](/Users/biwh/Desktop/blue_whale/文档/java8/assets/IMG_0269.jpg)

因为这些操作例如filter都是high-level building block，所以他们不依赖于特殊的线程模型，他们的核心实现可以是单线程的或者是结构中的最多线程数，这意味着我们在实际应用的时候不需要去考虑线程和锁的问题，streams已经帮我们做好了！

```java
Map<String, List<Dishes>> collect3 = menu.stream().collect(Collectors.groupingBy(Dishes::getType));
		collect3.forEach((k,v) -> {System.out.println(k);
			v.forEach(g -> System.out.println(g.getName()));
		});
```

这个例子就是我们根据Dishes的类型类进行分组！

使用java8的streams我们可以

1. **声明式编程**：可读性和简洁
2. **组合式的架构**：更加灵活
3. **并行**：更好的性能

优化案例

```java

public class Dishes{
	private int calorious;
	
	private String name;
	
	
	private Type type;
	
	public Type getType() {
		return type;
	}
	public void setType(Type type) {
		this.type = type;
	}
	public Dishes(int calorious, String name, Type type) {
		super();
		this.calorious = calorious;
		this.name = name;
		this.type = type;
	}
	public int getCalorious() {
		return calorious;
	}
	public void setCalorious(int calorious) {
		this.calorious = calorious;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}

	public Dishes() {
		super();
	}
	public Dishes(int calorious, String name) {
		super();
		this.calorious = calorious;
		this.name = name;
	}
	public int compareing(Dishes d1,Dishes d2) {
		return d1.getCalorious() - d2.getCalorious();
	}
}
    enum Type{
	DRINK,
	FOOD
}

```

```java
public class StreamOpration {
	public static void main(String[] args) {
//		List<Dishes> menu = new ArrayList<Dishes>();
//		menu.add(new Dishes(100, "红烧排骨","food"));
//		menu.add(new Dishes(20, "可乐","drink"));
//		menu.add(new Dishes(40, "米饭","food"));
//		menu.add(new Dishes(120, "鸡柳","food"));
//		menu.add(new Dishes(200, "满汉全席","food"));
	    List<Dishes> menu = Arrays.asList(new Dishes(100, "红烧排骨",Type.FOOD),
	    		new Dishes(20, "可乐",Type.DRINK),
	    		new Dishes(40, "米饭",Type.FOOD),
	    		new Dishes(120, "鸡柳",Type.FOOD),
	    		new Dishes(200, "满汉全席",Type.FOOD));
		List<String> collect = menu.stream()
				.filter(d -> d.getCalorious() > 40)
				.sorted(Comparator.comparing(Dishes::getCalorious))
				.map(Dishes::getName)
				.collect(Collectors.toList());
		collect.forEach(System.out::println);
		List<String> collect2 = menu.parallelStream()
				.filter(d -> d.getCalorious() > 40)
				.sorted(Comparator.comparing(Dishes::getCalorious))
				.map(Dishes::getName)
				.collect(Collectors.toList());
		collect2.forEach(System.out::println);
		Map<Type, List<Dishes>> collect3 = menu.stream().collect(Collectors.groupingBy(Dishes::getType));
		collect3.forEach((k,v) -> {System.out.println(k);
			v.forEach(g -> System.out.println(g.getName()));
		});
	}
}

```

## Getting start with streams

​	我们开始来讨论streams是从集合Collection开始的，因为这是我们使用Streams的最简单的方式，在java8 的java.util.stream.Stream包中，在后面的章节中你也可以看到streams的各种各样的用法！

​	究竟什么是Streams？一个简短确切的描述就是：一系列的数据处理操作（a sequence of element from a source that support datas processing opration）

![](/Users/biwh/Desktop/blue_whale/文档/java8/assets/IMG_B2646F7E330D-1.jpeg)

![](/Users/biwh/Desktop/blue_whale/文档/java8/assets/IMG_CFB7EE8452F9-1.jpeg)

![](/Users/biwh/Desktop/blue_whale/文档/java8/assets/IMG_9128FC929C44-2.jpeg)

 除此之外，Stream还有两个重要的特性

- 管道传输：许多的Stream操作返回的是本身，可以链式的进行下面的操作
- 内部迭代：集合的构造，可以使用迭代器来循环数据

示例：

```java
		List<String> collect = menu.stream()
				.filter(d -> d.getCalorious() > 300)
				.sorted(Comparator.comparing(Dishes::getCalorious))
				.map(Dishes::getName)
				.limit(2)
				.collect(Collectors.toList());

            List<PipedWriter> p = new ArrayList<>();
            p.stream().distinct().peek(
                   a ->  System.out.println("") //做外部处理 无返回值
            ).map((a) -> {
                return a.getClass();  //匹配PipedWriter中的字段挑选出来
            }).limit(100).collect(Collectors.toCollection(LinkedList::new));
```

![](/Users/biwh/Desktop/blue_whale/文档/java8/assets/IMG_2BA40AA9E433-1.jpeg)

## Streams 和 Collections

​	举个例子，我们有一个电影存储在DVD中，这是一个集合，因为所有的数据都保存在DVD中，现在我们将数据通过网络以数据流的方式获取，我们只需要先下载我们正在看的这一部分数据！

![](/Users/biwh/Desktop/blue_whale/文档/java8/assets/IMG_BCB86B1A670D-1.jpeg)

### 遍历一次

注意，与迭代器类似，流只能遍历一次。在那之后，一个流就被消耗掉了。您可以从初始数据源获取一个新的流来再次遍历它，就像迭代器一样(假设它是一个可重复的源，比如一个集合;如果是I/O通道，那就太不幸了)。例如，下面的代码将抛出一个异常，指示流已被消费:

~~~java
        List<String> list = Arrays.asList("1","2","3","4");
        Stream<String> stream = list.stream();
        stream.forEach(System.out::print);
        stream.forEach(System.out::print);
~~~

所以记住这一点，一个流只能被消费一次。

流和集合另一个关键的不同是在于它们如何管理数据上的迭代。

### 流和集合的不同区别

使用Collection接口需要用户进行迭代(例如，使用for-each);这称为外部迭代。相比之下，Streams库使用内部迭代——它为您执行迭代，并负责将结果流值存储在某个地方;您只需提供一个函数来说明要做什么。下面的代码清单说明了这种差异。

~~~java
        for (String string : list) {
            System.out.print(string);
        }
~~~

这正是您每天对Java集合所做的。您在外部迭代一个集合，显式地逐个提取和处理项。如果你能告诉Sofia，“把地板上所有的玩具都放到盒子里。”“有内部迭代比其他两个原因:首先,索非亚可以选择同时娃娃用一只手和球,第二,她可以决定采取最接近的对象框,然后其他人。同样，使用内部迭代，可以透明地并行完成项的处理，也可以按不同的顺序进行优化。如果您像在Java中那样在外部迭代集合，那么这些优化将是困难的。这可能看起来像是吹毛求疵，但它在很大程度上是Java 8引入流的核心——流库中的内部迭代可以自动选择数据表示和并行性的实现来匹配您的硬件。相比之下，一旦您通过为-each编写代码来选择了外部迭代，那么您基本上就可以自行管理任何并行性了。(在实践中，自我管理意味着“某天我们将并行化它”或“开始一场涉及任务和同步的漫长而艰巨的战斗”。)Java 8需要一个像Collection这样的接口，但是没有迭代器，因此流!图4.4说明了流(内部迭代)和集合(外部迭代)之间的区别。

![](/Users/biwh/Desktop/blue_whale/文档/java8/assets/IMG_A2ECA580DB1C-1 2.jpeg)

我们已经描述了集合和流之间的概念差异。具体来说，流利用内部迭代:迭代为您处理。但是，只有当您有一组要使用的预定义操作(例如，过滤器或映射)来隐藏迭代时，这才是有用的。大多数这些操作都将lambda表达式作为参数，因此您可以对它们的行为进行参数化，正如我们在前一章中所展示的那样。Java语言设计人员为Streams API提供了一个广泛的操作列表，您可以使用这些操作来表达复杂的数据处理查询。

