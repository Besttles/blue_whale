# 从BeanUtils到反射

## BeanUtils

BeanUtils是Apache旗下的产品，针对我们对javabean进行操作的工具。

### BeanUtils.*copyProperties*(source, target)

```java
		People p = new People("lili", 180);
		People p1 = new People();
		BeanUtils.copyProperties(p, p1);
```

方法是将source中的属性赋给target中，获取相同的属性和值！

源码：

在BeanUtils的抽象类中

```java
public abstract class BeanUtils {	
	//使用桥接设计模式来打印日志
    //可以看到我们在使用Spring-bean包的时候，需要导入commens-logging包
    private static final Log logger = LogFactory.getLog(BeanUtils.class);
	public static void copyProperties(Object source, Object target) throws BeansException {
		copyProperties(source, target, null, (String[]) null);
	}
}
```

调用了本类中的`BeanUtils.copyProperties(Object source, Object target, Class<?> editable, String... ignoreProperties)`方法,其中BeanUtils.copyProperties的三个public方法都是调用的这个私有方法

```java
	/**
	 * Copy the property values of the given source bean into the given target bean.
	 * <p>Note: The source and target classes do not have to match or even be derived
	 * from each other, as long as the properties match. Any bean properties that the
	 * source bean exposes but the target bean does not will silently be ignored.
	 * @param source the source bean
	 * @param target the target bean
	 * @param editable the class (or interface) to restrict property setting to
	 * @param ignoreProperties array of property names to ignore
	 * @throws BeansException if the copying failed
	 * @see BeanWrapper
	 */
	private static void copyProperties(Object source, Object target, Class<?> editable, String... ignoreProperties)
			throws BeansException {

		Assert.notNull(source, "Source must not be null");
		Assert.notNull(target, "Target must not be null");
		//使用反射来获取target类
		Class<?> actualEditable = target.getClass();
        //另一个方法，判断目标类与editable是否是同一个类或是其子类
		if (editable != null) {
			if (!editable.isInstance(target)) {
				throw new IllegalArgumentException("Target class [" + target.getClass().getName() +
						"] not assignable to Editable class [" + editable.getName() + "]");
			}
			actualEditable = editable;
		}
        //获取类中的所有属性，使用反射来获取
		PropertyDescriptor[] targetPds = getPropertyDescriptors(actualEditable);
        //判断和处理忽略的属性列表
		List<String> ignoreList = (ignoreProperties != null ? Arrays.asList(ignoreProperties) : null);
		//遍历该属性列表，进行赋值操作
		for (PropertyDescriptor targetPd : targetPds) {
            //获取属性写的方法，就是对应的setName()方法
			Method writeMethod = targetPd.getWriteMethod();
            //如果该属性没有对应写的方法，就不会进行赋值
			if (writeMethod != null && (ignoreList == null || !ignoreList.contains(targetPd.getName()))) {
				PropertyDescriptor sourcePd = getPropertyDescriptor(source.getClass(), targetPd.getName());
				if (sourcePd != null) {
                    //获取source的getName()方法
					Method readMethod = sourcePd.getReadMethod();
                    //若无该属性的get方法，和该属性是否可编辑
					if (readMethod != null &&
							ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], readMethod.getReturnType())) {
						try {
                   //判断该属性是否私有，若为私有，则开启访问权限
					if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
								readMethod.setAccessible(true);
							}
                            //读取source中的值
							Object value = readMethod.invoke(source);
					if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
								writeMethod.setAccessible(true);
							}
                            //将值赋给target
							writeMethod.invoke(target, value);
						}
						catch (Throwable ex) {
							throw new FatalBeanException(
									"Could not copy property '" + targetPd.getName() + "' from source to target", ex);
						}
					}
				}
			}
		}
	}
```

可以看到在赋值的时候并不是简单的1=1那样赋值，而是我们通过反射来获取该类中的属性，然后获取该类中属性的读的方法和写的方法进行赋值！

其中的关键点就是我们获取该类中的属性：

```java
PropertyDescriptor[] targetPds = getPropertyDescriptors(actualEditable);

	/**
	 * Retrieve the JavaBeans {@code PropertyDescriptor}s of a given class.
	 * @param clazz the Class to retrieve the PropertyDescriptors for
	 * @return an array of {@code PropertyDescriptors} for the given class
	 * @throws BeansException if PropertyDescriptor look fails
	 */
	public static PropertyDescriptor[] getPropertyDescriptors(Class<?> clazz) throws BeansException {
		CachedIntrospectionResults cr = CachedIntrospectionResults.forClass(clazz);
		return cr.getPropertyDescriptors();
	}

	/**
	 * Create CachedIntrospectionResults for the given bean class.
	 * @param beanClass the bean class to analyze
	 * @return the corresponding CachedIntrospectionResults
	 * @throws BeansException in case of introspection failure
	 */
	@SuppressWarnings("unchecked")
	static CachedIntrospectionResults forClass(Class<?> beanClass) throws BeansException {
		CachedIntrospectionResults results = strongClassCache.get(beanClass);
		if (results != null) {
			return results;
		}
		results = softClassCache.get(beanClass);
		if (results != null) {
			return results;
		}

		results = new CachedIntrospectionResults(beanClass);
		ConcurrentMap<Class<?>, CachedIntrospectionResults> classCacheToUse;

		if (ClassUtils.isCacheSafe(beanClass, CachedIntrospectionResults.class.getClassLoader()) ||
				isClassLoaderAccepted(beanClass.getClassLoader())) {
			classCacheToUse = strongClassCache;
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("Not strongly caching class [" + beanClass.getName() + "] because it is not cache-safe");
			}
			classCacheToUse = softClassCache;
		}

		CachedIntrospectionResults existing = classCacheToUse.putIfAbsent(beanClass, results);
		return (existing != null ? existing : results);
	}
```

为什么一个简单的赋值，会有这么多的骚操作呢？

上面的这段代码中，他考虑到了在类加载器加载的时候的内存会出现泄漏，所以会调用`CachedIntrospectionResults.forClass(clazz)`来防止内存泄漏！

BeanUtils中基本所有的方法追根溯源都能看到反射的影子，那么我们深入看一下反射的机制！

### BeanUtil拓展：手写Map映射实体

从上面的例子我们可以看到，我们可以根据属性名和属性类型来进行对象属性的复制，那么我们可以进阶的想一想我们是不是也可以将Map中的键值关系对应到实例中呢？

```java
	public static void CopyPropertiesForMap(Map<String, Object> map, Object o)
			throws IllegalAccessException, IllegalArgumentException, InvocationTargetException {
		// 获取所有该对象类型的属性
		PropertyDescriptor[] propertyDescriptors = BeanUtils.getPropertyDescriptors(o.getClass());
		// 遍历该属性数组
		for (PropertyDescriptor propertyDescriptor : propertyDescriptors) {
			String name = propertyDescriptor.getName();
			Object object = map.get(name);
			if (object != null) {
				// 获取读的方法
				Method writeMethod = propertyDescriptor.getWriteMethod();
				// 判断是否为空，对应map中的属性类型是否可以转换成writeMethod的入参类型
				if (writeMethod != null
						&& ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], object.getClass())) {
					// 若为私有属性，开启写入的权限
					if (Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
						writeMethod.setAccessible(true);
					}
					// 执行方法
					writeMethod.invoke(o, object);
				}
			}
		}
	}

```

从上面的代码我们可以看到，我们将Map中的属性进行匹配，匹配实体中的属性！你可能之前不知道为什么在RequestBody中的参数是怎么对应实体类中的属性的，还有在Mybatis中的返回结果是怎么和实体中的属性是如何对应的，其实原理大同小异，就是通过获取ResultMap中的键和获取实体中的属性名进行匹配！

在BeanUtils这个工具类中，有很多有用的工具方法可以使用，可以参考Spring的API进行学习，你会发现再庞大的工程也是一块一块砖头垒起来的！

## 反射

​	我们看过了在BeanUtils中进行实例属性复制的方法中，我们使用反射来进行获取实体的属性集合，那么什么是反射呢？反射又在java的API中扮演什么类型的角色呢？

### Class对象

​	类是程序的一部分，每一个类都有一个Calss对象。每当我们编写一个新类，就会产生一个Class对象（被保存在一个以'.class'为后缀的文件），为了生成这个对象，运行这个程序的java虚拟机将会使用称为**类加载器**的子系统。

​	**反射：**运行时的类信息

Class和java.lang.reflect类库一起对反射的概念进行支持，该类库包含了：Filed，Method，constructor类，每个类都实现了Member接口，这些类型的对象是在jvm在运行时创建的，用以表示位置类中对应的成员，你可以使用constructor来创建对象，用get()和set()方法类设置filed对象的属性，用invoke()来调用Method对象关联的方法，还有可以获取各种对象的方法，匿名对象的类信息在运行时可以被确定，而在编译时不需要知道任何信息！

在你想检查一个类的信息之前，你首先需要获取类的 Class 对象。Java 中的所有类型包括基本类型(int, long, float等等)，即使是数组都有与之关联的 Class 类的对象。如果你在编译期知道一个类的名字的话，那么你可以使用如下的方式获取一个类的 Class 对象。

```java
Class myObjectClass = MyObject.class;
```

如果你在编译期不知道类的名字，但是你可以在运行期获得到类名的字符串,那么你则可以这么做来获取 Class 对象:

```java
String className = ... ;//在运行期获取的类名字符串
Class class = Class.forName(className);
```

### 类名

在使用 Class.forName() 方法时，你必须提供一个类的全名，这个全名包括类所在的包的名字。例如 MyObject 类位于 com.jenkov.myapp 包，那么他的全名就是 com.jenkov.myapp.MyObject。 如果在调用Class.forName()方法时，没有在编译路径下(classpath)找到对应的类，那么将会抛出ClassNotFoundException。

你可以从 Class 对象中获取两个版本的类名。

通过 getName() 方法返回类的全限定类名（包含包名）：

```java
Class aClass = ... //获取Class对象，具体方式可见Class对象小节
String className = aClass.getName();
```

如果你仅仅只是想获取类的名字(不包含包名)，那么你可以使用 getSimpleName()方法:

```java
Class aClass = ... //获取Class对象，具体方式可见Class对象小节
String simpleClassName = aClass.getSimpleName();
```

### 修饰符

可以通过 Class 对象来访问一个类的修饰符， 即public,private,static 等等的关键字，你可以使用如下方法来获取类的修饰符：

```java
Class  aClass =  //获取Class对象，具体方式可见Class对象小节
int modifiers = aClass.getModifiers();
```

修饰符都被包装成一个int类型的数字，这样每个修饰符都是一个位标识(flag bit)，这个位标识可以设置和清除修饰符的类型。 可以使用 java.lang.reflect.Modifier 类中的方法来检查修饰符的类型：

```java
Modifier.isAbstract(int modifiers);
Modifier.isFinal(int modifiers);
Modifier.isInterface(int modifiers);
Modifier.isNative(int modifiers);
Modifier.isPrivate(int modifiers);
Modifier.isProtected(int modifiers);
Modifier.isPublic(int modifiers);
Modifier.isStatic(int modifiers);
Modifier.isStrict(int modifiers);
Modifier.isSynchronized(int modifiers);
Modifier.isTransient(int modifiers);
Modifier.isVolatile(int modifiers);
```

### 包信息

可以使用 Class 对象通过如下的方式获取包信息：

```java
Class  aClass = Class.forName(className); //获取Class对象，具体方式可见Class对象小节
Package package = aClass.getPackage();
```

通过 Package 对象你可以获取包的相关信息，比如包名，你也可以通过 Manifest 文件访问位于编译路径下 jar 包的指定信息，比如你可以在 Manifest 文件中指定包的版本编号。更多的 Package 类信息可以阅读 java.lang.Package。

### 父类

通过 Class 对象你可以访问类的父类，如下例：

```java
Class superclass = aClass.getSuperclass();
```

可以看到 superclass 对象其实就是一个 Class 类的实例，所以你可以继续在这个对象上进行反射操作。

### 实现的接口

可以通过如下方式获取指定类所实现的接口集合：

```
Class  aClass = ... //获取Class对象，具体方式可见Class对象小节
Class[] interfaces = aClass.getInterfaces();
```

由于一个类可以实现多个接口，因此 getInterfaces(); 方法返回一个 Class 数组，在 Java 中接口同样有对应的 Class 对象。 注意：getInterfaces() 方法仅仅只返回当前类所实现的接口。当前类的父类如果实现了接口，这些接口是不会在返回的 Class 集合中的，尽管实际上当前类其实已经实现了父类接口。

### 构造器

你可以通过如下方式访问一个类的构造方法：

```
Constructor[] constructors = aClass.getConstructors();
```

更多有关 Constructor 的信息可以访问 [Constructors](http://ifeve.com/java-reflection-constructors/)。

### 方法

你可以通过如下方式访问一个类的所有方法：

```java
Method[] method = aClass.getMethods();
```

### 变量

你可以通过如下方式访问一个类的成员变量：

```
Field[] method = aClass.getFields();
```

更多有关 Field 的信息可以访问 [Fields](http://ifeve.com/java-reflection-fields/)。

### 注解

你可以通过如下方式访问一个类的注解：

```
Annotation[] annotations = aClass.getAnnotations();
```



## 动态代理

代理是基本的设计模式之一，他是为你提供和外的或不同的操作，而插入用来代替**实际对象**的对象。

```java
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("proxy----");
		if(args != null) {
			for (Object object : args) {
				System.out.println(object);
			}
		}
		return method.invoke(iface, args);
	}
```

调用静态方法创建动态代理

```java
	public static void consumer(Interface iface) {
		iface.doSomething();
		iface.someThingElse("banado");
	}

	public static void main(String[] args) {
		RealObject real = new RealObject();
		consumer(real);
		Interface proxy = (Interface) Proxy.newProxyInstance(Interface.class.getClassLoader(),
				new Class[] { Interface.class }, new DynamicProxyHandler(real));
		consumer(proxy);
	}
```

这个方法需要一个类加载器，通常从已经加载的对象中获取，还有一个你希望该代理类实现的接口列表（不是类或者抽象类），以及InvocetionHandle的接口的一个实现类，动态代理可以将所有调用重定向到调用处理器，通常向该调用处理器传递一个实际对象的引用，从而使得该调用处理器执行其中介任务，可以将请求转发！

​	我们也可以使用通用的方法来获取被代理类！

定义被代理类的接口

```java
public interface SomeMethod {

	void boring1();
	void boring2();
	void interesting(String args);
	void boring3();
}

```

被代理类的实现类

```java
public class Impletation implements SomeMethod{

	@Override
	public void boring1() {
System.out.println("boring1");		
	}

	@Override
	public void boring2() {
		System.out.println("boring2");				
	}

	@Override
	public void interesting(String args) {
		System.out.println("interesting" + args);		
	}

	@Override
	public void boring3() {
		System.out.println("boring3");				
	}
}
```

获取被代理对象

```java
public class SelectionMethod {
	public static void main(String[] args) {

        //使用父类的构造器
		SomeMethod proxy = (SomeMethod) Proxy.newProxyInstance(SomeMethod.class.getClassLoader(),
				new Class[] { SomeMethod.class }, new MethodHandler(new Impletation()));
		proxy.boring1();
		proxy.boring2();
		proxy.interesting("yeah");
		proxy.boring3();
		
	}
}
```

