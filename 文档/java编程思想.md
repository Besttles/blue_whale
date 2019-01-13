# java编程思想

## java如何在运行时识别对象和类的信息？

### RTTI

让代码只操作基类的引用，这样来添加一个新类来拓展程序，就不会影响到原来的代码。

通过继承来实现：例如Spring core ！

**可以使用父类声明抽象方法，来强制子类来重写该方法，可以避免很多问题！**（多态）

java中所有的类型转换都需要在运行时进行正确性检查，也就是：***识别一个对象的类型***（RTTI）

多态机制：父类执行的方法，是由对象所指向的子类来决定的！

**class对象：每一个java的新类都会生成一个class对象，为了生成这个类的对象，将会使用JVM的子系统：类加载器**

所有的类都在其第一次使用时，动态加载到JVM中，当程序创建第一个对类的静态成员的引用时，就会加载这个类，构造方法也是这个类的静态方法，因此，使用NEW 关键字来创建类会被当作是对类静态方法的引用！



类字面常量：Fancy.class 这样做不仅安全，而且在编译时会收到检查（不需要将其置于try代码块中），并且根除了对方法forName方法的调用，所以更加高效！

1. 加载

   由类加载器执行，查找**字节码**（在classPath中查找），并从这些字节码文件中创建一个Class文件

2. 链接

   在链接阶段会进行检查，为静态区域分配空间，并且如果必须的话，会解析这个类对其他类的引用

3. 初始化

   如果这个类具有超类，则对其进行初始化，执行静态初始化器和静态初始化块

初始化会有效的进行惰性，仅使用.class来获得对类的引用不会引发初始化，但是为了产生Class的引用，使用Class.forName（）立即就进行了初始化

泛型：提供了编辑期检查

```java
public class FillType<T> {

	private Class<T> type;
	public FillType(Class<T> type){
		this.type = type;
	}
	
	public List<T> create(int num){
		List<T> result = new ArrayList<>();
			try {
				for(int i = 0; i<num;i++) {
				result.add(type.newInstance());
				}
			} catch (InstantiationException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (IllegalAccessException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		return result;
	}
	public static void main(String [] args) {
		FillType<CountedInteger> fl = 
				new FillType<CountedInteger>(CountedInteger.class);
		System.out.println(fl.create(15));
	}
}
```

```java
public class CountedInteger {

	private static long counter;
	
	private final long id = counter++;
	
	public String toString() {return Long.toString(id);};
}

```

