# Lambda表达式

简述：lambda表达式实际上就是对类方法的简写，将java的语法函数化！



 接口名
   说明
  Function<T,R>   
  接收一个T类型的参数，返回一个R类型的结果
  Consumer<T>
  接收一个T类型的参数，不返回值
  Predicate<T>
  接收一个T类型的参数，返回一个boolean类型的结果
  Supplier<T>
  不接受参数，返回一个T类型的结果

---------------------
作者：漠风- 
来源：CSDN 
原文：https://blog.csdn.net/qq_36372507/article/details/78757811 
版权声明：本文为博主原创文章，转载请附上博文链接！Functional interface

lambda表达式只能用于functionInterface的环境中

```java
@FunctionalInterface
public interface BufferReaderProcess {
	String process(BufferedReader br) throws IOException;
}
```

添加方法去实现

```java
public class FileResource {
	public static String processFile(BufferReaderProcess p) throws FileNotFoundException, IOException {
		try(BufferedReader bf = new BufferedReader(new FileReader("/Users/biwh/Desktop/blue_whale/文档/everyday.md")))
		{
			return p.process(bf);
		}
	}
	public static void main(String[] args) throws FileNotFoundException, IOException {
		processFile((BufferedReader br) -> br.readLine());
	}
}
```

分析：首先你可以创建一个$@FunctionalInterface$的接口，然后我们在实现的方法中将接口作为参数，然后返回该参数对应的方法！

通过这个例子，我们可以自己使用functionalInterface来运行lambda表达式，但是你必须自己定义接口，但是我们可以使用哪些在java8已经存在的接口方法呢？

#### predicate

源码：

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
```

filter方法来处理过滤条件

```java
	public static<T> List<T> filter(List<T> list,Predicate<T> p){
		List<T> results = new ArrayList<T>();
		for (T t : list) {
			if(p.test(t)) {
				results.add(t);
			}
		}
		return results;
	}
```

测试方法

```java
		Predicate<String> notEmpty =  (String s) -> !s.isEmpty();
		List<String> listofString = new ArrayList<String>();
		listofString.add("");
		listofString.add("1");
		listofString.add("");
		listofString.add("3");
		listofString.add("");
		List<String> non = filter(listofString, notEmpty);
		non.forEach((String s) -> System.out.println(s));
```

#### Consumer

你可以使用这个方法来存取你需要的数据（type T），还可以对这些数据进行处理，比遍历一个集合（即使我们已经有现成的方法来遍历集合）

```java
	public static <T> void  forEach(List<T> t,Consumer<T> c){
		for (T t2 : t) {
			c.accept(t2);
		}
	}
```

测试方法：

```java
forEach(listofString, (String i ) -> System.out.println(i));
```

#### Function

这个类可以将T类型的数据通过运算转化为另一种类型R的数据，并传入！

```java
	public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
		List<R> r = new ArrayList<>();
		for (T r2 : list) {
			r.add(f.apply(r2));
		}
		return r;
	}

```

测试方法：

```java
		map(listofString, (String s) -> s.length()).forEach((Integer i) -> System.out.println(i));

```

### Primitive specialization

在上面我们描述了三个functional类型的接口，还有一些functional的接口可以作用于某些确定的数据类型！

```java
@FunctionalInterface
public interface IntPredicate {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param value the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(int value);
}
```

测试

```java
		IntPredicate pre = (int i) -> i%2 == 0;
		boolean test = pre.test(100);
```

练习：

![](/Users/biwh/Desktop/blue_whale/assets/IMG_100F28BCBEC9-1.jpeg)

