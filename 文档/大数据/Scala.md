## Scala

### 什么是scala

Scala 是一门`多范式（multi-paradigm）`（面向对象，面向过程，函数式编程）的编程语言，设计初衷是要集成面向对象编程和函数式编程的各种特性。

Scala 运行在 Java 虚拟机上，并兼容现有的 Java 程序。

Scala 源代码被编译成 Java 字节码，所以它可以运行于 JVM 之上，并可以调用现有的 Java 类库。

### 学习scala的原因

简洁，高效，函数式编程

scala可以使用Java的部分语法

scala的特有语法

**函数式编程**：偏函数 ， 函数的柯里华 ， 高阶函数 ， 将函数作为参数传递 ， 对java的类接口进行包装

使用scalac编译，生成`.class`文件，执行时加载到jvm中

作者：马丁·奥德斯基

### scala编译

object scala是一个对应scala$的静态对象 module

在我们的程序中对应的是单例的，可以理解Scala是对Java代码的封装

### 字符串的输出

```scala
    println("hello world!")
    var str : String = "lili"
    printf("name = %s , age = %d", str , 12)
    println(s"name:${str+"and mingming"} \n age:${10}")
```

### scala输出注释文档

```
命令：scaladoc -d /Users/biwh/Downloads Test.scala
```

注释：

```scala
  /**
    * 求和
    * @example
    *          input num1 = 1 , num2 = 2 , sum = 1 + 2 = 3
    * @param num1
    * @param num2
    * @return num1 + num2
    */
  def sum (num1 : Int , num2 : Int): Int ={
    return num1 + num2
  }
```

### scala的类和对象

```scala
/**
  * Created by biwh on 2021/3/15.
  * scala的类和对象
  * 类是对象的抽象，对象是累的具象
  */
class DemoClass(str1 : String , str2 : String) {
  var x = str1
  var y = str2
  def concat (param1 : String , param2 : String): String ={
    return param1+ x + param2 + y
  }
}
```

### scala的数据类型

#### 整数类型

scala的整形默认为int型

```scala
var i = 10l;//l表示数据为long类型数据
```

#### 浮点型

浮点型常量的默认为double类型

```scala
    var f1:Float = 1.2F
    var f2 = 3.8
    var f3:Double = 4.6F//低精度赋值给高精度
```

Float精确到小数点后7位

5.12e2 = 5，12 * 10^2

### 顺序控制

程序的流程控制，方法自上而下执行

### 分支控制

if else来进行分支控制

