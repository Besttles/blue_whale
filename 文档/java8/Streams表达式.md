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

