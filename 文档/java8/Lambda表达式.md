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

![](https://github.com/Besttles/blue_whale/blob/master/assets/IMG_100F28BCBEC9-1.jpeg)



### Lambda Exception

记住这一点，所有的lambda表达式都不允许抛出任何检查时异常，如果你需要使用lambda表达式来抛出异常，你可以自己定义functional interface 来抛出异常，或者使用try catch 语句！

例子：

![](https://github.com/Besttles/blue_whale/blob/master/assets/IMG_0255.PNG)

```java
		Function<BufferedReader,String> f= (BufferedReader br) ->{
			try {
				return br.readLine();
			} catch (Exception e) {
				throw new RuntimeException();
			}
		};
```

我们在Functional interface中来声明异常！

### Type checking

lambda表达式的类型是从上下文中推理出来的！下面的例子看我们在使用lambda表达式的时候都发生了什么？
$$
Filter(inventory,(Apple a)->a.getWeight>50);
$$
在lambda表达式的定义中，我们需要怎么去使用？
$$
Filter(LIst<apple> inventory,predicate<apple> p ;)
$$
我们需要的target Type 就是这个 `predicate<apple> p` (T is bound to apple)!!

那么在predicate<T> 中，他的方法是什么呢？
$$
boolean test (apple a)
$$
那么我们在lambda表达式中怎么去返回一个boolean类型的值呢？

```
Apple -> boolean
```

Type-checking 的解析过程：

1. 检查filter方法的定义
2. filter方法需要一个正式的predicate类型的参数 Predicate<Apple>  target type
3. Predicate<apple> 是一个functional interface，定义了一个抽象方法 test()
4. Test() 方法描述了一个接收一个Apple类型的参数，返回了一个Boolean类型的返回值
5. 任何实际的内容对于Filter都需要去匹配请求的方法。   `Apple -> boolean`

### 同样的lambda表达式，不同的functional interface

同一个lambda表达式，如果有自己的抽象方法就可以关联不同的functional interface！

```java
Callable<Integer> = () -> 42;
PrivaledgeAction<Integer> () -> 42;
```

这就很像我们经常看到的Java中的多态，同样的方法，我们可以去定义不同的实现！

空实现无返回值的抽象方法

```java
//predicate has a bolean return
predicate<String> p  ->  list.add(s);
//consumer has a void return
consumer<String> p -> list.add(s);
```

下面的lambda表达式为什么是错的？

```java
Object  o  =  ()->{System.out.println("tricky example");};
```

该lambda表达式的内容是Object作为其Target Object，但是Object 并不是一个functional interface 你可以将Object去换成一个functional interface，例如 Runnable

```java
Runnabke o = () -> {System.out.println("tricky example");};
```

你已经知道了target type 在什么时候可以被用在特别的代码中！

### Type interface

你可以进一步简化你的代码。Java程序在编译的时候会去判断你到底关联了哪个lambda表达式从上下文中，也会去推断其功能的返回值是否适合target  type ，这有益于我们使用lambda表达式作为方法的参数使用，而且这可以在lambda 的语法中忽略！

#### java编译lambda表达式的过程：

1. 记住这一点，当一个lambda表达式的只有一个可推断其类型的参数时，在参数旁边的圆括号可以被省略

   ```java
   List<Apple> greenApple = filter(inventory, a -> a.weight()==10);
   ```

2. 当表达式中有多个参数的时候，我们将参数的类型定义好可以使我们的代码更加明确，下面的两个都是对的，但是哪个更好就是因人而异了！

3. ```java
   Comparetor<Apple> c = (Apple a1,Apple a2) -> (a1.getWeight()).compareTo(a2.getWeight());
   Comparetor<Apple> c = (a1,a2) -> (a1.getWeight()).compareTo(a2.getWeight());
   ```

### 使用成员变量

在lambda中，我们知道可以使用我们自己定义的变量，比如
$$
(Apple a1,Apple a2) -> (a1.getWeight()).compareTo(a2.getWeight())
$$
同样在lambda中，我们也可以使用已经定义好的成员变量，比如
$$
int port = 1337;

Runnable = （）-> System.out.println(port);
$$

```java
		int port = 100;
		Runnable a = () -> System.out.println(port);
		a.run();
		port = 200;
```

上面的这段代码会编译报错，因为在lambda表达式中使用的变量只能又一次赋值，或者是final类型的数据！

### lambda在本地变量中的限制

你可能会问在lambda中使用实例变量为什么会有那么多的限制，比如上面的代码！

首先，成员变量存储在堆(heap)中，本地变量存储在栈中，如果我们使用lambda表达式在一个单独的线程中，lambda会将使用的local 变量来复制一份，而不是直接访问原有的变量，所以这就是为什么在lambda中使用的本地变量我们只允许编辑一次！