# zookeeper理论

## 了解zookeeper

### zookeeper基础

**zookeeper不直接暴露原语，而是将方法的一小部分的调用方法组成类似文件系统的API，以便允许应用来实现这些原语！**

我们通常使用recipes来表示这些原语的实现。recipes包括zookeeper操作的一个小型的数据节点，这些节点被称为znode采用类似文件系统的树状层级结构进行管理。

![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_856248B55369-1.jpeg)

### API概述

znode节点可能有数据，也可能没有。如果节点中存在数据，那么数据的格式以字节数字的形式储存，字节数组的格式和应用有关，zookeeper并不提供解析。

zookeeper暴露的方法：

- create/path data  创建一个/path的节点，并包含数据data
- delete/path   删除名为/path 的znode
- exists/path 检查是否存在名为 /path 的znode
- getDate/path  获取/path节点的数据信息
- getChildren/path  获取/path节点下的字节点列表

zookeepering不支持znode局部的写入或读取，当设置一个znode节点的写入或读取的时候，znode节点会整个替换或读取过来。

### znode的不同类型

**一个临时的znode会在一下两种情况下被删除**

1. 当创建该znode的客户端的绘画因超时或是主动关闭而终止的时候
2. 当某个客户端主动删除该节点

**有序节点**

一个znode可以设置为有序，一个有序的znode节点被分配唯一的单调递增的整数，当创建有序节点的时候，一个序号会被增加到路径之后。

znode一共有四种类型

- 持久节点
- 临时节点
- 持久有序节点
- 临时有序节点

### 监视和通知

Zookeeper通常以远程服务的方式被访问。

当客户端访问节点的时候，需要获得节点中的内容，这样的代价是非常大的，而且zookeeper需要更多的操作，而且第二次调用getChildren/task的时候也会返回相同的内容，这是非常没有必要的！

![客户端访问zookeeper](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_09D01C53A576-1.jpeg)

为了替换客户端轮训，我们选择了基于通知的机制：客户端向zookeeper注册需要接收通知的znode，通过对znode设置监听点来接收通知，监视点是单次出发的操作，为了接收多个通知客户端需要在每次通知后设置一个新的监视点。



![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_09D01C53A576-1-9969557.jpeg)

> 当节点/task发生变化的时候，客户端会收到一个通知，并从zookeeper中读取一个新值。

通知机制的一个重要保障就是：对同一个znode的操作，先向客户端传送通知，然后再对该节点进行变更，如果客户端对一个znode设置可监视点，而该节点发生了连续两次更新，当第一次更新开始后，客户端在发生第二次变化之之前获取了通知，然后读取znode中的数据。我们认为主要特性在于通知机制阻止了客户端所观察的更新顺序。

zookeeper可以定义不同类型的通知，这依赖于设置监视点对应通知的类型。

### 版本

每个znode都有一个版本号。它随着每次数据变化而自增，两个API会根据条件执行：setData和delete，这两个调用以版本号作为参数只当传入的版本号与服务器的版本号相同的时候，才会调用成功。

![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_F6E6DC669F10-1.jpeg)

## Zookeeper架构

**上面我们了解了zookeeper的上层如何操作来暴露给应用，我们也需要了解服务是如何运行的？**

zookeeper的两种运行模式：

1. 单机模式

   单独的服务器，服务器的状态无法复制。

2. 集群模式

   一组zookeeper服务器，它们之间可以相互复制状态，并同时服务于客户端的请求。

### 集群模式

在集群模式下，每个zookeeper服务器都将复制所有服务器的数据树，但是让每个客户端在每个服务器保存完成后继续，会有很大的延迟。

> 法定人数：进行一项投票所需的最小人数。

在整个集群模式中，我们允许少于一半的服务器崩溃，最好是使用奇数个服务器构建集群。

### 会话

在zookeeper集合执行任何请求前，一个客户端必须与服务建立会话。

客户端提交给zookeeper的所有操作都是关联在这个会话上，当某个会话因某种原因中止后，在这个会话期间创建的所有临时节点都将会消失。当客户端连接集群中的某一台服务器或一个独立的服务器，客户端通过TCP协议与服务器进行连接并通信，但当会话无法与当前连接的服务器继续通信时，会话可能转移到另一台服务器上，zookeeper客户端透明的转移会话到不同的服务器。

会话提供了顺序保障，这意味着同一个会话总的请求会以先进先出的顺序执行。若客户端有多个会话，顺序就未必会在多个会话中保持。

## 开始使用zookeeper

安装zookeeper

启动zookeeper

客户端访问

ls /  列出跟节点下多有的节点

![image-20190609153751245](/Users/biwh/Desktop/blue_whale/文档/assets/image-20190609153751245.png)

创建节点 create /workers ""

![image-20190609154124380](/Users/biwh/Desktop/blue_whale/文档/assets/image-20190609154124380.png)

删除节点

![image-20190609154254558](/Users/biwh/Desktop/blue_whale/文档/assets/image-20190609154254558.png)

退出

![image-20190609154337645](/Users/biwh/Desktop/blue_whale/文档/assets/image-20190609154337645.png)

### 会话的状态和声明周期

会话的声明周期是指会话从创建到结束的周期。

一个会话的状态有：**connecting**，**connected**，**closed**，**not_collected**。

状态的转换依赖于客户端与服务器之间的各种事件。

![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_10B067FE1D6B-1.jpeg)

创建一个会话时，你需要设置会话超时这个参数，这个参数声明了zookeeper服务允许会话被声明为超时之前存在的时间，如果经过时间T之后服务器接受不到这个会话的任何消息，服务就回啊声明会话过期。而在客户端这里，如果经过T/3的时间未收到任何消息，客户端会向服务器发送心跳消息。而在2T/3后，zookeeper会开始寻找其他服务器，这是它还有T/3的时间去寻找。

在集群模式下，当尝试连接到不同的服务器时，非常重要的是，这个服务器zookeeper的状态要与最后连接的服务器的zookeeper状态保持最新。zookeeper通过在服务中排序更新操作来决定状态是否最新。zookeeper确保每一个变化相对于其他已经执行的更新是完全有序的。

![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_BBDF54873575-1.jpeg)

### zookeeper的集群模式

我们配置zookeeper为集群模式，在conf.cfg中我们配置

```properties
server.1 = 127.0.0.1:2222:2223
server.2 = 127.0.0.1:3333:3334
server.3 = 127.0.0.1:4444:4445
```

每一个server.n指定了zookeeper服务器使用的地址和端口号，每个地址被：分隔为三部分，第一部分为服务器的IP地址，第二部分为集群通信端口，第三部分为群首选举端口。

我们还需要分别设置data目录，我们可以用下面的命令。

```nginx
mkdir z1
mkdir z1/data
mkdir z2
mkdir z2/data
mkdir z3
mkdir z3/data
```

当启动一个服务器时，我们需要知道是哪个服务器，一个服务器通过data目录下的一个myid的文件来获取服务器ID信息我们通过下面的命令来创建这些文件

```nginx
echo 1 > z1/data/myid
echo 2 > z2/data/myid
echo 3 > z3/data/myid
```

当服务器启动时，服务器通过配置文件中的dataDir来读取目录的配置，通过读取myData来获取ID信息，之后使用服务器配置的端口设置监听。

当我们打开三个服务器中的一个时，整个服务无法运行，在日志中我们看到

![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_BBBF97F7C58A-1.jpeg)

这个服务器在尝试连接其他的服务器，然后失败，这是我们启动另一台服务器，我们可以达到整个集群的1/2以上，这时我们会看到

![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_DCDC200F0A61-1.jpeg)

服务器2已经被选举为首领了。这时我们看服务器1的日志

![](/Users/biwh/Desktop/blue_whale/文档/assets/IMG_2149F5EF4D42-1.jpeg)

服务器1作为服务器2的追随者被激活。

### 通过zooleeper实现锁

