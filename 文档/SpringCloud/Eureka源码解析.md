# Eureka源码

## Eureka服务的操作:

**register服务注册**：当Eureka客户端向Eureka Server注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等。

<div align="center"> <img src="assets/1568786526089.png" width="450"/> </div><br>
**renew服务续约**：Eureka客户会每隔30秒发送一次心跳来续约。 通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。 正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。 建议不要更改续约间隔。

**fetch register获取服务注册信息**：Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与Eureka客户端的缓存信息不同， Eureka客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，Eureka客户端则会重新获取整个注册表信息。 Eureka服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。Eureka客户端和Eureka 服务器可以使用JSON / XML格式进行通讯。在默认的情况下Eureka客户端使用压缩JSON格式来获取注册列表的信息。

**cancel服务下线**：Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。

**eviction服务剔除**：在默认的情况下，当Eureka客户端连续90秒没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。

## Register服务注册

服务注册，即Eureka Client向Eureka Server提交自己的服务信息，包括IP地址、端口、service ID等信息。如果Eureka Client没有写service ID，则默认为 ${[spring.application.name](http://spring.application.name/)}。

com.netflix.discovery包下的DiscoveryClient类，该类包含了Eureka Client向Eureka Server的相关方法。

~~~java
@Singleton
public class DiscoveryClient implements EurekaClient {}
~~~

<div align="center"> <img src="assets/1568788040723.png" width="450"/> </div><br>
~~~java
    boolean register() throws Throwable {
        logger.info("DiscoveryClient_{}: registering service...", this.appPathIdentifier);

        EurekaHttpResponse httpResponse;
        try {
            //使用HTTP的方式注册服务
            httpResponse = this.eurekaTransport.registrationClient.register(this.instanceInfo);
        } catch (Exception var3) {
            logger.warn("DiscoveryClient_{} - registration failed {}", new Object[]{this.appPathIdentifier, var3.getMessage(), var3});
            throw var3;
        }
        if (logger.isInfoEnabled()) {
            logger.info("DiscoveryClient_{} - registration status: {}", this.appPathIdentifier, httpResponse.getStatusCode());
        }

        return httpResponse.getStatusCode() == 204;
    }
~~~

$$
@Deprecated  接口过时，需要声明新的接口名称和地址
$$

$$
@Inject在构造方法上使用 @Inject 时，其参数在运行时由配置好的IoC容器提供。(因为JRE无法决定构造方法注入的优先级，所以规范中规定类中只能有一个构造方法带@Inject注解)
$$

在DiscoveryClient类继续追踪register()方法，它被InstanceInfoReplicator 类的run()方法调用，其中InstanceInfoReplicator实现了Runnable接口

~~~java
    public void run() {
        boolean var6 = false;

        ScheduledFuture next;
        label53: {
            try {
                var6 = true;
                this.discoveryClient.refreshInstanceInfo();
                Long dirtyTimestamp = this.instanceInfo.isDirtyWithTime();
                if (dirtyTimestamp != null) {
                    this.discoveryClient.register();
                    this.instanceInfo.unsetIsDirty(dirtyTimestamp);
                    var6 = false;
                } else {
                    var6 = false;
                }
                break label53;
            } catch (Throwable var7) {
                logger.warn("There was a problem with the instance info replicator", var7);
                var6 = false;
            } finally {
                if (var6) {
                    ScheduledFuture next = this.scheduler.schedule(this, (long)this.replicationIntervalSeconds, TimeUnit.SECONDS);
                    this.scheduledPeriodicRef.set(next);
                }
            }

            next = this.scheduler.schedule(this, (long)this.replicationIntervalSeconds, TimeUnit.SECONDS);
            this.scheduledPeriodicRef.set(next);
            return;
        }

~~~

在服务注册的过程中需要向Server服务器进行续约

~~~java
    private void initScheduledTasks() {
        int renewalIntervalInSecs;
        int expBackOffBound;
        if (this.clientConfig.shouldFetchRegistry()) {
            renewalIntervalInSecs = this.clientConfig.getRegistryFetchIntervalSeconds();
            expBackOffBound = this.clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
            this.scheduler.schedule(new TimedSupervisorTask("cacheRefresh", this.scheduler, this.cacheRefreshExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.CacheRefreshThread()), (long)renewalIntervalInSecs, TimeUnit.SECONDS);
        }

        if (this.clientConfig.shouldRegisterWithEureka()) {
            renewalIntervalInSecs = this.instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
            expBackOffBound = this.clientConfig.getHeartbeatExecutorExponentialBackOffBound();
            logger.info("Starting heartbeat executor: renew interval is: {}", renewalIntervalInSecs);
            this.scheduler.schedule(new TimedSupervisorTask("heartbeat", this.scheduler, this.heartbeatExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.HeartbeatThread()), (long)renewalIntervalInSecs, TimeUnit.SECONDS);
            this.instanceInfoReplicator = new InstanceInfoReplicator(this, this.instanceInfo, this.clientConfig.getInstanceInfoReplicationIntervalSeconds(), 2);
            this.statusChangeListener = new StatusChangeListener() {
                public String getId() {
                    return "statusChangeListener";
                }

                public void notify(StatusChangeEvent statusChangeEvent) {
                    if (InstanceStatus.DOWN != statusChangeEvent.getStatus() && InstanceStatus.DOWN != statusChangeEvent.getPreviousStatus()) {
                        DiscoveryClient.logger.info("Saw local status change event {}", statusChangeEvent);
                    } else {
                        DiscoveryClient.logger.warn("Saw local status change event {}", statusChangeEvent);
                    }

                    DiscoveryClient.this.instanceInfoReplicator.onDemandUpdate();
                }
            };
            if (this.clientConfig.shouldOnDemandUpdateStatusChange()) {
                this.applicationInfoManager.registerStatusChangeListener(this.statusChangeListener);
            }

            this.instanceInfoReplicator.start(this.clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
        } else {
            logger.info("Not registering with Eureka server per configuration");
        }
    }
~~~

使用`ScheduledExecutorService`来定时获取注册列表

在server端，先追踪PeerAwareInstanceRegistryImpl类，在该类的register()方法，该方法提供了注册，并且将注册后信息同步到其他的Eureka Server服务。

~~~java
    public void register(InstanceInfo info, boolean isReplication) {
        int leaseDuration = 90;
        if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
            leaseDuration = info.getLeaseInfo().getDurationInSecs();
        }
        super.register(info, leaseDuration, isReplication);
        this.replicateToPeers(PeerAwareInstanceRegistryImpl.Action.Register, info.getAppName(), info.getId(), info, (InstanceStatus)null, isReplication);
    }
~~~

点击进入子类中，我们会发现Eureka将发现的服务存入Map中。map是通过`ConcurrentHashMap`存储的各个客户端注册的服务，其中key为注册服务的AppName，value为其中注册的服务信息的map

~~~java
            this.read.lock();
            Map<String, Lease<InstanceInfo>> gMap = (Map)this.registry.get(registrant.getAppName());
            EurekaMonitors.REGISTER.increment(isReplication);
            if (gMap == null) {
                ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap();
                gMap = (Map)this.registry.putIfAbsent(registrant.getAppName(), gNewMap);
                if (gMap == null) {
                    gMap = gNewMap;
                }
            }
~~~



## Eureka源码入口

**com.netflix.appinfo.InstanceInfo** 存储了服务注册的所有信息

------

# feign源码

## feign客户端

**@feign注解**

~~~java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {
    @AliasFor("name")
	String value() default "";

	/**
	 * The service id with optional protocol prefix. Synonym for {@link #value() value}.
	 *
	 * @deprecated use {@link #name() name} instead
	 */
	@Deprecated
	String serviceId() default "";

	/**
	 * The service id with optional protocol prefix. Synonym for {@link #value() value}.
	 */
	@AliasFor("value")
	String name() default "";
}
~~~

@FeignClient注解被@Target(ElementType.TYPE)修饰，表示FeignClient注解的作用目标在接口上；
@Retention(RetentionPolicy.RUNTIME)，注解会在class字节码文件中存在，在运行时可以通过反射获取到；

@Documented表示该注解将被包含在javadoc中。

feign 用于声明具有该接口的REST客户端的接口的注释应该是创建（例如用于自动连接到另一个组件。 如果功能区可用，那将是用于负载平衡后端请求，并且可以配置负载平衡器使用与伪装客户端相同名称（即值）@RibbonClient 。

其中value()和name()一样，是被调用的 service的名称。
url(),直接填写硬编码的url,decode404()即404是否被解码，还是抛异常；configuration()，标明FeignClient的配置类，默认的配置类为FeignClientsConfiguration类，可以覆盖Decoder、Encoder和Contract等信息，进行自定义配置。fallback(),填写熔断器的信息类。

### FeignClient的配置

默认的配置类为`FeignClientsConfiguration`，这个类在spring-cloud-netflix-core的jar包下，打开这个类，可以发现它是一个配置类，注入了很多的相关配置的bean，包括feignRetryer、FeignLoggerFactory、FormattingConversionService等,其中还包括了Decoder、Encoder、Contract，如果这三个bean在没有注入的情况下，会自动注入默认的配置。

~~~java
@Configuration
public class FeignClientsConfiguration {

	@Autowired
	private ObjectFactory<HttpMessageConverters> messageConverters;
	@Autowired(required = false)
	private List<AnnotatedParameterProcessor> parameterProcessors = new ArrayList<>();
	@Autowired(required = false)
	private List<FeignFormatterRegistrar> feignFormatterRegistrars = new ArrayList<>();
	@Autowired(required = false)
	private Logger logger;
}

~~~

**重写配置：**

你可以重写FeignClientsConfiguration中的bean，从而达到自定义配置的目的，比如FeignClientsConfiguration的默认重试次数为Retryer.NEVER_RETRY①，即不重试，那么希望做到重写，写个配置文件，注入feignRetryer的bean。

~~~java
@Configuration
public class FeignConfig {

    @Bean
    public Retryer feignRetryer() {
        return new Retryer.Default(100, SECONDS.toMillis(1), 5);
    }
}
~~~

在上述代码更改了该FeignClient的重试次数，重试间隔为100ms，最大重试时间为1s,重试次数为5次。

### Feign的工作原理

feign是一个伪客户端，即它不做任何的请求处理。**Feign通过处理注解生成request，从而实现简化HTTP API开发的目的**，即开发人员可以使用注解的方式定制request api模板，在发送http request请求之前，feign通过处理注解的方式替换掉request模板中的参数，这种实现方式显得更为直接、可理解。

通过包扫描注入FeignClient的bean，该源码在FeignClientsRegistrar类：
首先在启动配置上检查是否有@EnableFeignClients注解，如果有该注解，则开启包扫描，扫描被@FeignClient注解接口。

~~~java
	private void registerDefaultConfiguration(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
		Map<String, Object> defaultAttrs = metadata
				.getAnnotationAttributes(EnableFeignClients.class.getName(), true);

		if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
			String name;
			if (metadata.hasEnclosingClass()) {
				name = "default." + metadata.getEnclosingClassName();
			}
			else {
				name = "default." + metadata.getClassName();
			}
			registerClientConfiguration(registry, name,
					defaultAttrs.get("defaultConfiguration"));
		}
	}
~~~

程序启动后通过包扫描，当类有@FeignClient注解，将注解的信息取出，连同类名一起取出，赋给BeanDefinitionBuilder，然后根据BeanDefinitionBuilder得到beanDefinition，最后beanDefinition式注入到ioc容器中。

~~~java
	public void registerFeignClients(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
		ClassPathScanningCandidateComponentProvider scanner = getScanner();
		scanner.setResourceLoader(this.resourceLoader);

		Set<String> basePackages;

		Map<String, Object> attrs = metadata
				.getAnnotationAttributes(EnableFeignClients.class.getName());
		AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
				FeignClient.class);
		final Class<?>[] clients = attrs == null ? null
				: (Class<?>[]) attrs.get("clients");
		if (clients == null || clients.length == 0) {
			scanner.addIncludeFilter(annotationTypeFilter);
			basePackages = getBasePackages(metadata);
		}
		else {
			final Set<String> clientClasses = new HashSet<>();
			basePackages = new HashSet<>();
			for (Class<?> clazz : clients) {
				basePackages.add(ClassUtils.getPackageName(clazz));
				clientClasses.add(clazz.getCanonicalName());
			}
			AbstractClassTestingTypeFilter filter = new AbstractClassTestingTypeFilter() {
				@Override
				protected boolean match(ClassMetadata metadata) {
					String cleaned = metadata.getClassName().replaceAll("\\$", ".");
					return clientClasses.contains(cleaned);
				}
			};
			scanner.addIncludeFilter(
					new AllTypeFilter(Arrays.asList(filter, annotationTypeFilter)));
		}

		for (String basePackage : basePackages) {
			Set<BeanDefinition> candidateComponents = scanner
					.findCandidateComponents(basePackage);
			for (BeanDefinition candidateComponent : candidateComponents) {
				if (candidateComponent instanceof AnnotatedBeanDefinition) {
					// verify annotated class is an interface
					AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
					AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
					Assert.isTrue(annotationMetadata.isInterface(),
							"@FeignClient can only be specified on an interface");

					Map<String, Object> attributes = annotationMetadata
							.getAnnotationAttributes(
									FeignClient.class.getCanonicalName());

					String name = getClientName(attributes);
					registerClientConfiguration(registry, name,
							attributes.get("configuration"));

					registerFeignClient(registry, annotationMetadata, attributes);
				}
			}
		}
	}
~~~

注入bean之后，通过jdk的代理，当请求Feign Client的方法时会被拦截，代码在ReflectiveFeign类。

~~~java
    public <T> T newInstance(Target<T> target) {
        Map<String, MethodHandler> nameToHandler = this.targetToHandlersByName.apply(target);
        Map<Method, MethodHandler> methodToHandler = new LinkedHashMap();
        List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList();
        Method[] var5 = target.type().getMethods();
        int var6 = var5.length;

        for(int var7 = 0; var7 < var6; ++var7) {
            Method method = var5[var7];
            if (method.getDeclaringClass() != Object.class) {
                if (Util.isDefault(method)) {
                    DefaultMethodHandler handler = new DefaultMethodHandler(method);
                    defaultMethodHandlers.add(handler);
                    methodToHandler.put(method, handler);
                } else {
                    methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
                }
            }
        }
        InvocationHandler handler = this.factory.create(target, methodToHandler);
        T proxy = Proxy.newProxyInstance(target.type().getClassLoader(), new Class[]{target.type()}, handler);
        Iterator var12 = defaultMethodHandlers.iterator();

        while(var12.hasNext()) {
            DefaultMethodHandler defaultMethodHandler = (DefaultMethodHandler)var12.next();
            defaultMethodHandler.bindTo(proxy);
        }

        return proxy;
    }
~~~

在SynchronousMethodHandler类进行拦截处理，当被FeignClient的方法被拦截会根据参数生成RequestTemplate对象，该对象就是http请求的模板。

~~~java
    public Object invoke(Object[] argv) throws Throwable {
        RequestTemplate template = this.buildTemplateFromArgs.create(argv);
        Retryer retryer = this.retryer.clone();
        while(true) {
            try {
                return this.executeAndDecode(template);
            } catch (RetryableException var5) {
                retryer.continueOrPropagate(var5);
                if (this.logLevel != Level.NONE) {
                    this.logger.logRetry(this.metadata.configKey(), this.logLevel);
                }
            }
        }
    }
~~~

其中有个executeAndDecode()方法，该方法是通RequestTemplate生成Request请求对象，然后根据用client获取response.

~~~java
    Object executeAndDecode(RequestTemplate template) throws Throwable {
        Request request = this.targetRequest(template);
        if (this.logLevel != Level.NONE) {
            this.logger.logRequest(this.metadata.configKey(), this.logLevel, request);
        }

        long start = System.nanoTime();

        Response response;
        try {
            response = this.client.execute(request, this.options);
            response.toBuilder().request(request).build();
        } catch (IOException var15) {
            if (this.logLevel != Level.NONE) {
                this.logger.logIOException(this.metadata.configKey(), this.logLevel, var15, this.elapsedTime(start));
            }

            throw FeignException.errorExecuting(request, var15);
        }
~~~

### Feign的负载均衡

通过FeignRibbonClientAutoConfiguration类配置Client的类型(httpurlconnection，okhttp和httpclient)时候，可知最终向容器注入的是LoadBalancerFeignClient，即负载均衡客户端。现在来看下LoadBalancerFeignClient的代码。

~~~java
	@Override
	public Response execute(Request request, Request.Options options) throws IOException {
		try {
			URI asUri = URI.create(request.url());
			String clientName = asUri.getHost();
			URI uriWithoutHost = cleanUrl(request.url(), clientName);
			FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
					this.delegate, request, uriWithoutHost);

			IClientConfig requestConfig = getClientConfig(options, clientName);
			return lbClient(clientName).executeWithLoadBalancer(ribbonRequest,
					requestConfig).toResponse();
		}
		catch (ClientException e) {
			IOException io = findIOException(e);
			if (io != null) {
				throw io;
			}
			throw new RuntimeException(e);
		}
	}
~~~

其中有个executeWithLoadBalancer()方法，即通过负载均衡的方式请求。

~~~java
    public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
        LoadBalancerCommand command = this.buildLoadBalancerCommand(request, requestConfig);

        try {
            return (IResponse)command.submit(new ServerOperation<T>() {
                public Observable<T> call(Server server) {
                    URI finalUri = AbstractLoadBalancerAwareClient.this.reconstructURIWithServer(server, request.getUri());
                    ClientRequest requestForServer = request.replaceUri(finalUri);

                    try {
                        return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
                    } catch (Exception var5) {
                        return Observable.error(var5);
                    }
                }
            }).toBlocking().single();
        } catch (Exception var6) {
            Throwable t = var6.getCause();
            if (t instanceof ClientException) {
                throw (ClientException)t;
            } else {
                throw new ClientException(var6);
            }
        }
    }
~~~

其中服务在submit()方法上，点击submit进入具体的方法,这个方法是LoadBalancerCommand的方法：

~~~java
     Observable<T> o = 
                (server == null ? selectServer() : Observable.just(server))
                .concatMap(new Func1<Server, Observable<T>>() {
                    @Override
                    // Called for each server being selected
                    public Observable<T> call(Server server) {
                        context.setServer(server);
        }}
~~~

上述代码中有个selectServe()，该方法是选择服务的进行负载均衡的方法，代码如下：

~~~java
    private Observable<Server> selectServer() {
        return Observable.create(new OnSubscribe<Server>() {
            public void call(Subscriber<? super Server> next) {
                try {
                    Server server = LoadBalancerCommand.this.loadBalancerContext.getServerFromLoadBalancer(LoadBalancerCommand.this.loadBalancerURI, LoadBalancerCommand.this.loadBalancerKey);
                    next.onNext(server);
                    next.onCompleted();
                } catch (Exception var3) {
                    next.onError(var3);
                }
            }
        });
    }
~~~

最终负载均衡交给loadBalancerContext来处理，即之前讲述的Ribbon。

### 总结

首先通过@EnableFeignCleints注解开启FeignCleint根据Feign的规则实现接口，并加@FeignCleint注解程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。
当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplateRequesTemplate在生成RequestRequest交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。

------

