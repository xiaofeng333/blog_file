---
title: spring test 
tags: test
date: 2020-05-31 10:08:05

---

### 前言

本文涉及的内容包括spring test、spring boot test、mockito, 然本文并不是教程文档, 详情请参考如下推荐文档。

[spring test](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html) 

[spring boot test](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-testing)

[mockito](https://wenku.baidu.com/view/8def451a227916888486d73f.html)

### spring test

spring的IOC使得测试更加方便。

因为基于构造函数或setter方法注入的依赖, 我们可以使用new或者mock对象来进行测试。 

#### Unit Testing

对于单元测试, spring提供了mock objects和support classes(但我们可能不需要使用这些)。

mock类位于`org.springframework.mock.env`包下, 主要分为`env`、`jndi`、`http`、`web`四个子包。

通用测试工具类位于`org.springframework.test.util`包下, 如`ReflectionTestUtils`、`AopTestUtils`等。

`ModelAndViewAssert`工具类用于spring mvc单元测试(结合对应的mock对象)。

#### Integration Testing

用于测试正确配置spring ioc container contexts及数据访问(如sql语句、Hibernate查询、JPA entity映射等)。

##### Context Management and Caching

加载一次, 后续测试可重复使用已配置的ApplicationContext。

###### @ContextConfiguration

Spring支持从`xml`、`groovy scripts`、`component classes`中加载配置, 但由于历史原因, 必须选择一种类型作为入口, 其余类型在此类型中导入。例如在`component classes`中通过`@ImportSource`导入`xml`或`groovy scripts`配置。

还可以通过属性`initializers`指定`ApplicationContextInitializer`的实现类来声明bean,省略`xml`等配置。

###### context cache

context通过[context cache key](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-ctx-management-caching)来存储。

spring TestContext framework存储application contexts在static cache中, 因此test选择使用多个处理器来执行时, 其缓存机制无法生效。

[forkCount](https://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#forkCount)/forkMode为默认值即可。

context cache默认最大值为32, 当达到最大值时, 将会`LRU`策略删除老的context。可通过`spring.test.context.cache.maxSize`来修改默认值(声明方式为`system properties`或`SpringProperties`)。

设置`org.springframework.test.context.cache`的日志级别为`debug`查看已加载多少context cache。

##### Dependency Injection of test fixture

通过DI从`ApplicationContext`获取已定义的bean。

##### Transaction Management

默认情况, 每个测试的`transaction`都会回滚。

##### Supported Classes for Intergation Testing

`ApplicationContext`和`JdbcTemplate`

##### Annotations

###### Spring Testing Annotations

@BootstrapWith: 类级别注解, 如何加载`Spring TestContext Framework`。默认为`DefaultTestContextBootstrapper`或`WebTestContextBootstrapper`, 基于是否出现`@WebAppConfiguration`来判断。

@ContextConfiguration: 类级别注解, 如何加载`ApplicationContext`。指定resource locations或者component classes。

@DirtiesCOntext: 当`context`被标记为`dirty`, 其将会从`test framework's caches`中移除。因此在随后使用相同`configuration meta`的测试中, `context`将会重建。可指定methodMode、classMode、hierarchyMode。

@BeforeTransaction/@AfterTransaction: 指定在`@Transactional`标注的方法前或之后运行, 不属于当前事务, @Before/@After在当前事务中。

@TestExecutionListeners: 用于注册自定义的`TestExecutionListener`实现。此时, 应将mergeMode改为`MergeMode.MERGE_WITH_DEFAULTS`, 以免默认的listener失效。

还可以通过SpringFactoriesLoader机制加载TestExecutionListener的实现。

TestExecutionListener的实现需实现`Ordered`接口或通过`@Order`注解声明执行顺序。

其余包括@WebAppConfiguration、@ContextHierarchy、@ActiveProfiles、@TestPropertySource、@Commit、@Rollback、@Sql、@SqlConfig、@SqlMergeMode、@SqlGroup。

###### Standard Annotation Support

@Autowired、@Qualifier、@Value、@Resource、@ManagedBean、@Inject、@Named、@PersistenceContext、@PersistenceUnit、@Required、@Transactional

注意: `@PostConstruct`将会运行在所有的before方法之前, 但`@PreDestory`不会生效。推荐使用`lifcycle callbacks`替代`@PostConstruct`和`@PreDestory`。

###### Spring Junit4 Testing Annotations

@IfProfileValue、@ProfileValueSourceConfiguration、@Timed、@Repeat

###### Spring Junit Jupiter Testing Annotations

@SpringJunitConfig、@SpringJunitWebConfig、@TestConstructor、@EnabledIf、@DisabledIf

##### Spring TestContext Framework

##### Key Abstractions

* TestContext: 包含测试执行的context, 提供测试实例的context management和caching support, 同时也可委派`SmartContextLoader`加载`ApplicationContext`。
* TextContextManager: Spring TestContext Framework的main entry point。负责管理单个`TestContext`及分发事件至已注册的`TestExecutionListener`。
* TestExecutionListener
* SmartContextLoader: 用于加载`ApplicationContext`。默认的loaders为`DelegatingSmartContextLoader`和`WebDelegatingSmartContextLoader`。

### Spring Boot Test

@SpringBootTest

### Mockito

其对于final类、匿名类及基本类型无法mock。

stub对象用于提供测试时所需要的测试数据, 可以对各种交互设置相应的回应。

mock对象用于验证测试中所依赖的对象间的交互是否能够达到预期。mock对象只能调用已stub的方法, 调用不了真实的方法。

doNothing()用于void方法。

`@Mock`、`@Runwith(MockitoJUnitRunner)`、`@ExtendWith(MockitoExtension.class)`

@InjectMocks: Mockito依次基于构造函数和setter方法注入mock对象。

```java
when().thenReturn(); // 用于事先stub, 对于static或final的方法无法进行设定
verify(obj，times(1)) // 对应方法只调用了一次 atMost()、atLeast()至多/至少调用了几次
```

Mockito还提供了Inorder类, 负责验证在适当的行动过程中进行的方法调用顺序。

ArgumentMatchers用于参数匹配。

spy()用于模拟真实对象, 此时使用`doReturn().when()`代替`when().thenReturn()`避免调用真实对象。

reset()用于重置模拟对象。

timeout()指定超时时间。