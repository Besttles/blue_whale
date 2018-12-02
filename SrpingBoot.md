# SrpingBoot

## SpringBoot

### 自动配置原理

springboot 启动的时候加载主配置类，开启了自动配置功能`@EnableAutoConfigration`

`@EnableAutoConfigration`给容器中导入组件

- SpringFactoriesLoader/loadFactoryName()

- 扫描jar包类路径下的。 MATA-INF/spring.factories
- 把扫描到的文件的内容包装成properties文件
- 从properties中获取到EnableAutoConfigration.class对应的值，然后把他们添加到容器中
- 所有配置文件能配置的属性都在XXXProperties类中封装着
- SpringBoot启动会加载大量的配置类

将类路径下的 `MATA_INF/spring.factories` 里面配置的所有`EnableConfigration`的值加入到容器中

```java
@Configuration
@EnableConfigurationProperties(HttpEncodingProperties.class)
@ConditionalOnWebApplication
@ConditionalOnClass(CharacterEncodingFilter.class)
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration{
    
}
```

```java
@ConfigurationProperties(prefix = "spring.http.encoding")
public class HttpEncodingProperties {
    
}
```



`@Configration`  表示这是一个配置类，给容器中添加组件

`@ConfigrationProperties` 从配置文件中获取指定的值和bean的属性绑定

`@ConditionalOnWebApplication` Spring底层的@condition注解，如果满足条件，则配置类生效，判断当前应用是否为web应用，如果是web应用，则配置类生效

`@ConditionOnProperties（XXXX.class）` 判断当前项目是不是有这个类

**根据当前不同的条件判断，决定这个配置类是否生效**，一旦配置类生效，就会给容器添加各种组件

`@bean` 给容器中添加一个组件

**我们可以配置的属性都是来源于properties类！**

我们可以使用debug模式来判断哪些自动配置类生效(打印自动匹配报告)

```properties
debug=true
```





### @Conditional派生注解

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置类中的内容才会生效

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnBeanCondition.class)
public @interface ConditionalOnMissingBean {}
```

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

### WEB开发

SpringBoot根据我们导入的场景进行自动配置，只需要添加少量的属性就可以运行起来

SpringBoot对静态文件的映射规则

```java
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Integer cachePeriod = this.resourceProperties.getCachePeriod();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler("/webjars/**")
								.addResourceLocations(
										"classpath:/META-INF/resources/webjars/")
						.setCachePeriod(cachePeriod));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(
										this.resourceProperties.getStaticLocations())
						.setCachePeriod(cachePeriod));
			}
		}
```

`webjars`  以jar包的形式引入静态文件，将jequery和js以maven依赖的形式进行导入

www.webjars.org 

在访问的时候只需要写访问资源的名称

`/**` 访问当前项目的任何资源 （静态资源文件夹）

```
"classpath:/META-INF/resources/"
"classpath:/resources/"  (在resources下面再建一个“resources”文件夹)
"classpath:/static/"
"classpath:/public"
```

```java
@ConfigurationProperties(prefix = "spring.mvc")
public class WebMvcProperties {
    /**
	 * Path pattern used for static resources.
	 */
	private String staticPathPattern = "/**";
}
```

```java
		@Bean  //配置欢迎页  index.html
		public WelcomePageHandlerMapping welcomePageHandlerMapping(
				ResourceProperties resourceProperties) {
			return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage());
		}
       
        @Configuration
		@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
		public static class FaviconConfiguration {

			private final ResourceProperties resourceProperties;

			public FaviconConfiguration(ResourceProperties resourceProperties) {
				this.resourceProperties = resourceProperties;
			}

			@Bean
			public SimpleUrlHandlerMapping faviconHandlerMapping() {
				SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
				mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
				mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
						faviconRequestHandler()));//配置图标
				return mapping;
			}

```

定义静态资源文件夹`spring.resources.static-location = /hello/,class path:/stuff/`

自定义静态资源文件夹后，之前默认的都会失效！



### 缓存

JSR-107 和 SpringBoot缓存抽象

从缓存中存取数据

CatchManager 接口统一不同的缓存技术

`CatchManager` 缓存管理器，管理各种缓存组件

`Catch` 缓存接口，定义缓存操作，实现有：redisCatch，EhCatchCatch，ConcurrentMapCatch等

`@Cacheable` 主要针对方法配置，能根据方法的请求参数对结果进行缓存

`@CacheEvict` 清空缓存（当我们删除什么的时候，需要清空缓存）

`@CachePut` 保证方法被调用，又希望结果被缓存（当我们更新一个数据的时候，可以执行方法并更新缓存）

`@EnableCaching` 开启基于注解的缓存





## redis

环境：redis 192.168.10.163:6379

[Redis]: https://redisbook.readthedocs.io/en/latest/index.html



### 连接redis，配置yaml文件

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

