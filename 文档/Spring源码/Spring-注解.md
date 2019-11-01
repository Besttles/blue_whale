

# Spring注解



```java
@AliasFor：
//表示别名，它可以注解到自定义注解的两个属性上，表示这两个互为别名，也就是说这两个属性其实同一个含义
1.同个注解中的两个属性互为别名
/*****************************************************************************************/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Inherited
public @interface MyAnnotation {
    @AliasFor(attribute = "location")
    String value() default "";
    
    @AliasFor(attribute = "value")
    String location() default "";
}


@MyAnnotation(location = "location")
public class AliasTest extends BaseTest {

    @Test
    public void test() {
        MyAnnotation myAnnotation = AnnotationUtils.getAnnotation(this.getClass(), MyAnnotation.class);
        System.out.println("value:" + myAnnotation.value() + ";loation:" + myAnnotation.location());
    }
}

输出：value:location;loation:location
//无论指明哪个属性值，另一个属性的属性值也是相同的

继承父注解，使其功能更加强大
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Inherited
@MyAnnotation
public @interface SubMyAnnotation {
    
    @AliasFor(value="location",annotation=MyAnnotation.class)
    String subLocation() default "";
    @AliasFor(annotation=MyAnnotation.class)   //缺省指明继承的父注解的中的属性名称，则默认继承父注解中同名的属性名
    String value() default "";
}
```

[@SpringBootApplication如何启动应用程序](https://blog.csdn.net/qq_28289405/article/details/81302498)

