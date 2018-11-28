# SrpingBoot

## redis

环境：redis 192.168.10.163:6379

[Redis]: https://redisbook.readthedocs.io/en/latest/index.html



### 连接redis，配置yml文件

```yaml
spring:
    redis:
      host: 192.168.10.163
      #redis密码，没有密码的可以用～表示
      password: ~
      port: 6379
      pool:
        max-active: 100  #最大连接数
        max-idle: 10
        max-wait: 100000
```

### 编写工具类：***RedisTemplate*** 实例

``` java
public class RedisServiceImpl implements RedisService {

    private static int seconds=3600*24;

    //使用redisTemplate对redis进行操作
    @Autowired
    private RedisTemplate<String, ?> redisTemplate;
}
```

### Ubuntu安装redis数据库

配置数据源，更新数据源，默认配置的数据源不支持在CN下载更新软件

```yaml
# 安装redis
apt-get install redis-server
#查看是否启动
ps -aux|grep redis
#1.使用redis账号访问
#默认情况下，访问Redis服务器是不需要密码的
#为了增加安全性，设置Redis服务器的访问密码
vi /etc/redis/redis.conf
#取消requirepass前的注释#，并设置密码
#requirepass 123456
#重启redis
/etc/init.d/redis-server restart
#允许远程访问
#修改配置文件redis.conf
#bind 127.0.0.1

```

###redis原理

#### 持久化

由于Redis的数据都存放在内存中，如果没有配置持久化，redis重启后数据就全丢失了，于是需要开启redis的持久化功能，将数据保存到磁盘上，当redis重启后，可以从磁盘中恢复数据。redis提供两种方式进行持久化，一种是RDB持久化（原理是将Reids在内存中的数据库记录定时dump到磁盘上的RDB持久化），另外一种是AOF持久化（原理是将Reids的操作日志以追加的方式写入文件）。

- RDB 将数据库的快照（snapshot）以二进制的方式保存到磁盘中。

- AOF 则以协议文本的方式，将所有对数据库进行过写入的命令（及其参数）记录到 AOF 文件，以此达到记录数据库状态的目的。

  - `rdbSave` 会将数据库数据保存到 RDB 文件，并在保存完成之前阻塞调用者。

  - [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 命令直接调用 `rdbSave` ，阻塞 Redis 主进程； [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 用子进程调用 `rdbSave` ，主进程仍可继续处理命令请求。

  - [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 执行期间， AOF 写入可以在后台线程进行， [BGREWRITEAOF](http://redis.readthedocs.org/en/latest/server/bgrewriteaof.html#bgrewriteaof) 可以在子进程进行，所以这三种操作可以同时进行。

  - 为了避免产生竞争条件， [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 执行时， [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 命令不能执行。

  - 为了避免性能问题， [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 和 [BGREWRITEAOF](http://redis.readthedocs.org/en/latest/server/bgrewriteaof.html#bgrewriteaof) 不能同时执行。

  - 调用 `rdbLoad` 函数载入 RDB 文件时，不能进行任何和数据库相关的操作，不过订阅与发布方面的命令可以正常执行，因为它们和数据库不相关联。

  - RDB 文件的组织方式如下：

    ```
    +-------+-------------+-----------+-----------------+-----+-----------+
    | REDIS | RDB-VERSION | SELECT-DB | KEY-VALUE-PAIRS | EOF | CHECK-SUM |
    +-------+-------------+-----------+-----------------+-----+-----------+
    
                          |<-------- DB-DATA ---------->|
    ```

  - 键值对在 RDB 文件中的组织方式如下：

    ```
    +----------------------+---------------+-----+-------+
    | OPTIONAL-EXPIRE-TIME | TYPE-OF-VALUE | KEY | VALUE |
    +----------------------+---------------+-----+-------+
    ```

    RDB 文件使用不同的格式来保存不同类型的值。

##### RDB

RDB方式可以将Redis 在内存中的数据库状态保存到磁盘里面，避免数据意外丢失
RDB持久化既可以手动执行，也可以根据服务器配置选项定期执行，该功能可以将某个时间点上的数据库状态保存到一个RDB文件中，如图10-2所示
RDB 持久化功能所生成的RDB 文件是一个经过压缩的二进制文件，通过该文件可以还原生成RDB 文件时的数据库状态

RDB文件的创建与载入

```:six:
redis> SAVE  //等待直到RDB 文件创建完毕
OK
redis> BGSAVE //派生子进程，并由子进程创建RDB 文件
Background saving started
```

和使用SAVE命令或者BGSAVE命令创建RDB文件不同，RDB 文件的载入工作是在服务启动时自动执行的，所以Redis并没有专门用于载入RDB文件的命令，只要Redis服务器在启动时检测到RDB文件存在，它就会自动载入RDB文件
以下是Redis服务器启动时打印的日志记录，其中第二条日志DB loaded from disk就是服务器在成功载入RDB 文件之后打印的

##### AOF

[AOF持久化]: https://redisbook.readthedocs.io/en/latest/internal/aof.html





## mysql

ubuntu系统安装mysql，注意版本，5.7之后的版本在设置外部访问，和用户创建方面有些需要注意的地方

#### 连接池：druid 连接池

简介：

[常规数据库连接池原理]: https://mp.weixin.qq.com/s/tLysIX9KChNioJ-fMMimxw

#### sql 优化

