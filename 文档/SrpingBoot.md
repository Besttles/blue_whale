# SrpingBoot

## SpringBoot

[SpringBoot官方文档](https://docs.spring.io/spring/docs/5.1.4.RELEASE/spring-framework-reference/core.html#spring-core)  [SpringBoot测试](https://docs.spring.io/spring/docs/5.1.4.RELEASE/spring-framework-reference/testing.html#testing) [SpringMVC](https://docs.spring.io/spring/docs/5.1.4.RELEASE/spring-framework-reference/web.html#mvc-servlet) 

### 自动配置原理

[笔记](https://blog.csdn.net/sdzhangshulong/article/details/81075579)

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

### AOP源码

AOP部分的解析器是由`AopNamespaceHandler` 注册，其init方法

```java
	@Override
	public void init() {
		// In 2.0 XSD as well as in 2.1 XSD.
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());

		// Only in 2.0 XSD: moved to context namespace as of 2.1
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
	}
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



### 全局异常

[全局异常](https://www.jianshu.com/p/99ded527bc47)

### 定制错误响应

Springboot 默认的错误处理的自动配置类

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class })
// Load before the main WebMvcAutoConfiguration so that the error View is available
@AutoConfigureBefore(WebMvcAutoConfiguration.class)
@EnableConfigurationProperties(ResourceProperties.class)
public class ErrorMvcAutoConfiguration {}
```

添加的组件：

```java
	@Bean
	@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
	public DefaultErrorAttributes errorAttributes() {
		return new DefaultErrorAttributes();
	}
    /**
    @Controller
    @RequestMapping("${server.error.path:${error.path:/error}}")
    public class BasicErrorController extends AbstractErrorController {
    进行错误处理，如果没有配置error处理的controller，默认会去处理/error请求，
    */
	@Bean
	@ConditionalOnMissingBean(value = ErrorController.class, search = SearchStrategy.CURRENT)
	public BasicErrorController basicErrorController(ErrorAttributes errorAttributes) {
		return new BasicErrorController(errorAttributes, this.serverProperties.getError(),
				this.errorViewResolvers);
	}
    /**
    当系统出现错误的时候，会转到/error请求
    @Value("${error.path:/error}")
	private String path = "/error";
    */
	@Bean
	public ErrorPageCustomizer errorPageCustomizer() {
		return new ErrorPageCustomizer(this.serverProperties);
	}
    //处理浏览器请求的到哪个页面
	@Bean
	@ConditionalOnBean(DispatcherServlet.class)
	@ConditionalOnMissingBean
	public DefaultErrorViewResolver conventionErrorViewResolver() {
		return new DefaultErrorViewResolver(this.applicationContext,
			this.resourceProperties);
	}
    //先去找error/404 的控制器，如果没有的话，就会去静态文件夹里去找error/404.html 的文件
	private ModelAndView resolveResource(String viewName, Map<String, Object> model) {
		for (String location : this.resourceProperties.getStaticLocations()) {
			try {
				Resource resource = this.applicationContext.getResource(location);
				resource = resource.createRelative(viewName + ".html");
				if (resource.exists()) {
					return new ModelAndView(new HtmlResourceView(resource), model);
				}
			}
			catch (Exception ex) {
			}
		}
		return null;
	}

```

basicErrorController方法在处理系统错误的时候，会有两种形式，一种是出现页面的形式，一种是返回json类型的数据

怎么区分是浏览器请求还会其他请求，根据浏览器请求的请求头来识别请求中的**Accept：text/html**代表优先接收html类型的文件，如果是 

```java
Accept:*/*   //代表优先接受任意类型的文件
```

浏览器请求

```java
	//浏览器请求，返回错误的页面
    @RequestMapping(produces = "text/html")
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
	}
    //处理请求的页面信息
    protected ModelAndView resolveErrorView(HttpServletRequest request,
			HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
		for (ErrorViewResolver resolver : this.errorViewResolvers) {
			ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
			if (modelAndView != null) {
				return modelAndView;
			}
		}
		return null;
	}
```

请求返回的内容

```java
	 /*（需要在模版引擎中获取相应的数据，如果html文件是放在静态文件夹下的，就不能获取下面的值）
	 DefaultErrorAttributes 定义了所有返回的内容
	 status 状态码
	 timestamp
	 error
	 exception
	 message
      */
     private void addStatus(Map<String, Object> errorAttributes,
			RequestAttributes requestAttributes) {
		Integer status = getAttribute(requestAttributes,
				"javax.servlet.error.status_code");
		if (status == null) {
			errorAttributes.put("status", 999);
			errorAttributes.put("error", "None");
			return;
		}
		errorAttributes.put("status", status);
		try {
			errorAttributes.put("error", HttpStatus.valueOf(status).getReasonPhrase());
		}
		catch (Exception ex) {
			// Unable to obtain a reason
			errorAttributes.put("error", "Http Status " + status);
		}
	}
```



定制错误页面，将错误码.html的文件放入静态文件夹

```java
    //所有以4，5开头的错误码，
    static {
		Map<Series, String> views = new HashMap<Series, String>();
		views.put(Series.CLIENT_ERROR, "4xx");
		views.put(Series.SERVER_ERROR, "5xx");
		SERIES_VIEWS = Collections.unmodifiableMap(views);
	}
```



其他客户端请求

```java
 //其他客户端，返回错误的json格式的数据
	@RequestMapping
	@ResponseBody
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}
```

定制错误的json数据

自定义异常处理器

```java
@ControllerAdvice
public class MyExceptionHandler {
    //浏览器和其他客户端都是json
	@ExceptionHandler(OApiException.class)
	public Map<String,String> handleException() {
		Map<String,String> map = new HashMap<>();
		map.put("code", "OPI");
		return map;
	}
}
    //转发到/errer请求进行处理,达到自适应效果
	@ExceptionHandler(OApiException.class)
	public String getErrorView(Exception e,HttpServletRequest request) {
		Map<String,String> map = new HashMap<>();
        //需要传入我们的错误码，不然的话会是200请求，不进行错误解析
        request.setAttribute("javax.servlet.error.status_code", "404");
        map.put("message", e.getMessage());
		map.put("code", "OPI");
		return "forward:/error";
	}
```

自定义errorAttribute

```java
@Component
public class MyErrorAttribute extends DefaultErrorAttributes{

	
	@Override
	public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
			boolean includeStackTrace) {
           Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
		map.put("message", "get it");
        //可以获取在异常处理器中自适应的数据
        //参数中的key 和 scope ，scope中request：0， session ：1
        Map<String,Object> sta = (Map<String, Object>)        requestAttributes.getAttribute("ext", 0);
           map.put("ext", sta);
           return map;		
	}
	
}
```



### 数据源

#### 数据源自动配置

```
org.apache.tomcat.jdbc.pool.DataSource
HikariDataSource
org.apache.commons.dbcp.BasicDataSource
org.apache.commons.dbcp2.BasicDataSource
自定义数据源
```



### WEB MVC扩展

使用WebMvcConfigrationAdaptor来扩展SpringMVC 的功能

```java
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter{

	@Override
	public void addViewControllers(ViewControllerRegistry registry) {
		//super.addViewControllers(registry);
		registry.addViewController("/welcome").setViewName("welcome");
	}
}
```

既保留了自动配置的SpringMVC的自动配置，又可以手动配置资源的映射！

**源码解析**：

```java
	@Configuration
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {

```

自动配置的SpringMvc也是继承的这个类来实现的！

```java
@Configuration
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {
```

做自动配置的时候会导入@Import(EnableWebMvcConfiguration.class)

```java
	/**
	 * Configuration equivalent to {@code @EnableWebMvc}.
	 */
	@Configuration
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {

```

他继承的父类：**DelegatingWebMvcConfiguration**

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

    //从容器中获取所有的WebMvcConfigurer赋值到configurers中
	@Autowired(required = false)//自动装配
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebMvcConfigurers(configurers);
		}
	}
    //实现的方法WebMvcConfigurer里所有的配置都会起作用，所以我们扩展的配置类也会起作用
	@Override
	public void addViewControllers(ViewControllerRegistry registry) {
        //将所有的
		for (WebMvcConfigurer delegate : this.delegates) {
			delegate.addViewControllers(registry);
		}
	}
```

#### 全面接管SpringMVC

在自动配置类上面添加**@EnableWebMvc**

不启动Springmvc 的自动配置，全部执行手动配置，静态资源失效。







## redis

环境：redis 192.168.10.163:6379

[redis](https://redisbook.readthedocs.io/en/latest/index.html)

[SpringBoot-redis官方文档](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:template)



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

## dubbo

一个远程服务调用的分布式框架

**远程通讯**：提供多种基于长连接的NIO框架的抽象封装，包括多种线程模型，序列化，以及“请求-响应”的信息交换方式

**集群容错**：提供基于接口方法的透明远程调用，包括多种协议支持，以及软件负载均衡，失败容错，地址路由，动态配置等集群支持

**自动发现**：基于注册中心的目录服务，使服务消费方能动态查找服务提供方，使地址透明，使服务提供方可以平滑的增加或减少机器

### 源码分析

SpringBoot的自动配置初始化服务

```java
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        Environment env = applicationContext.getEnvironment();
        String scan = env.getProperty("spring.dubbo.scan");
        if (scan != null) {
            AnnotationBean scanner = BeanUtils.instantiate(AnnotationBean.class);
            scanner.setPackage(scan);
            scanner.setApplicationContext(applicationContext);
            applicationContext.addBeanFactoryPostProcessor(scanner);
            applicationContext.getBeanFactory().addBeanPostProcessor(scanner);
            applicationContext.getBeanFactory().registerSingleton("annotationBean", scanner);
        }

```

会获取对应再在自动配置类中的`spring.dubbo.scan`属性的值

```java
@ConfigurationProperties(prefix = "spring.dubbo")
public class DubboProperties {

    private String scan; //设置扫描包的路径

    private ApplicationConfig application;

    private RegistryConfig registry;

    private ProtocolConfig protocol;
}
```

RegistryConfig.  配置文件被封装成RefistryConfig.class

```java
public class RegistryConfig extends AbstractConfig {

	private static final long serialVersionUID = 5508512956753757169L;
	
	public static final String NO_AVAILABLE = "N/A";

    // 注册中心地址
    private String            address;
    
	// 注册中心登录用户名
    private String            username;

    // 注册中心登录密码
    private String            password;

    // 注册中心缺省端口
    private Integer           port;
    
    // 注册中心协议
    private String            protocol;

    // 客户端实现
    private String            transporter;
    
    private String            server;
    
    private String            client;

    private String            cluster;
    
    private String            group;

	private String            version;

    // 注册中心请求超时时间(毫秒)
    private Integer           timeout;

    // 注册中心会话超时时间(毫秒)
    private Integer           session;
    
    // 动态注册中心列表存储文件
    private String            file;
    
    // 停止时等候完成通知时间
    private Integer           wait;
    
    // 启动时检查注册中心是否存在
    private Boolean           check;

    // 在该注册中心上注册是动态的还是静态的服务
    private Boolean           dynamic;
    
    // 在该注册中心上服务是否暴露
    private Boolean           register;
    
    // 在该注册中心上服务是否引用
    private Boolean           subscribe;

    // 自定义参数
    private Map<String, String> parameters;

    // 是否为缺省
    private Boolean             isDefault;
}
    
```

```java
    private void doExportUrls() {
        //该方法根据配置文件转配成一个url的list
        List<URL> registryURLs = loadRegistries(true);
        //根据每一个协议配置分别来暴露服务
        for (ProtocolConfig protocolConfig : protocols) {
            doExportUrlsFor1Protocol(protocolConfig, registryURLs);
        }
    }
```

这个protocols长这个样子`<dubbo:protocol name="dubbo" port="20888" id="dubbo" /> protocols`也是根据配置装配出来的。接下来让我们进入doExportUrlsFor1Protocol方法看看dubbo具体是怎么样将服务暴露出去的。这个方法特别大，有将近300多行代码，但是其中大部分都是获取类似protocols的name、port、host和一些必要的上下文

组装url的方法：

```java
com.alibaba.dubbo.config.ServiceConfig.class.doExportUrlsFor1Protocol(protocolConfig, registryURLs);
```

看到这里就比较明白dubbo的工作原理了doExportUrlsFor1Protocol方法，先创建URL，URL创建出来长这样`dubbo://192.168.xx.63:20888/com.xxx.xxx.VehicleInfoService?anyhost=true&application=test-web&default.retries=0&dubbo=2.5.3&interface=com.xxx.xxx.VehicleInfoService&methods=get,save,update,del,list&pid=13168&revision=1.2.38&side=provider&timeout=5000&timestamp=1510829644847`，是不是觉得这个URL很眼熟，没错在注册中心看到的services的providers信息就是这个，再传入url通过proxyFactory获取Invoker，再将Invoker封装成Exporter的数组，只需要将这个list提供给网络传输层组件，然后consumer执行Invoker的invoke方法就行了。让我们再看看这个proxyFactory的getInvoker方法。proxyFactory下有JDKProxyFactory和JavassistProxyFactory。官方推荐也是默认使用的是JavassistProxyFactory。因为javassist动态代理性能比JDK的高。

```java
public interface Invoker<T> extends Node {

    /**
     * get service interface.
     * 
     * @return service interface.
     */
    Class<T> getInterface();

    /**
     * invoke.
     * 
     * @param invocation
     * @return result
     * @throws RpcException
     */
    Result invoke(Invocation invocation) throws RpcException;

}
```

Invoker 的实现类（AbstractProxyInvoker）的invoke方法

```java
    public Result invoke(Invocation invocation) throws RpcException {
        try {
            return new RpcResult(doInvoke(proxy, invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments()));
        } catch (InvocationTargetException e) {
            return new RpcResult(e.getTargetException());
        } catch (Throwable e) {
            throw new RpcException("Failed to invoke remote proxy method " + invocation.getMethodName() + " to " + getUrl() + ", cause: " + e.getMessage(), e);
        }
    }
```

可以看到使用了动态代理的方式调用了要暴露的service的方法。并且返回了Invoker对象。在dubbo的服务发布中我们可以看到，这个Invoker贯穿始终，都可以看成是一个context的作用了





- RPC框架的简易结构

  服务消费方以本地调用的方式调用服务

  client stub 接收到调用后负责将方法，参数等组装成能够进行网络传输的消息体

  client stub 找到服务地址并将消息发送到服务端

  server stub 收到消息后进行解码

  server stub 根据解码的结果进行调用本地的服务

  本地服务执行，并将结果返回给server stub

  server stub 将返回结果进行打包成消息发送至消费方

  client stub 接受到消息，并进行解码

  服务消费方的到最终的结果

- dubbo客户端的初始化

- dubbo服务端的初始化

- dubbo客户端处理请求的流程

- dubbo服务端处理请求的流程

## Nginx

[nginx资料](https://www.jianshu.com/p/bed000e1830b?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)

## Docker

开源的应用容器容器引擎

- 轻量级，快速

  将软件编译好成为镜像

docker主机：安装了docker程序的机器，安装于操作系统之上的

docker客户端：客户端通过命令行或者图形化界面来使用docker

docker仓库：用来保存各种打包好的镜像

docker镜像：创建docker容器的镜像

docker容器：镜像启动后的实例或一组应用

