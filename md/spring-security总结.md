---
title: spring security总结
tags: spring security
date: 2019-03-10 16:26:09
---

#### 介绍

应用程序安全性可以归结为差不多两个独立的问题：身份验证（你是谁？）和授权（你可以做什么？）。

Spring Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI（控制反转Inversion of Control ,DI:Dependency Injection 依赖注入）和AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。

#### 身份认证和访问控制

##### 认证

认证策略的核心接口是 `AuthenticationManager` 。

```JAVA
public interface AuthenticationManager {
  Authentication authenticate(Authentication authentication)
    throws AuthenticationException;
}
```

`AuthenticationManager` 最常用的实现是 `ProviderManager`，它委托给一个`AuthenticationProvider` 实例链。 `AuthenticationProvider`有点像`AuthenticationManager`，但它有一个额外的方法来允许调用者询问它是否支持给定的认证类型。

，AuthenticationManager 接口的常用实现类 ProviderManager 内部会维护一个 List<AuthenticationProvider> 列表，存放多种认证方式。ProviderManager 中的 List，会依照次序去认证，认证成功则立即返回，若认证失败则返回 null，下一个AuthenticationProvider 会继续尝试认证，如果所有认证器都无法认证成功，则 ProviderManager 会抛出一个 ProviderNotFoundException 异常。

ProviderManager是Authentication的一个实现，并将具体的认证操作委托给一系列的AuthenticationProvider来完成，从而可以实现支持多种认证方式。为了帮助阅读和理解源码具体做了什么，这里删除了原来的一部分注释，并对重要的部分进行了注释说明。

```java
public interface AuthenticationProvider {
    Authentication authenticate(Authentication var1) throws AuthenticationException;

    boolean supports(Class<?> var1);
}
```

`supports()` 方法中的`Class <?>`参数实际上是 `Class<? extends Authentication>`（它只会被问到是否支持将被传递到 `authenticate()` 方法的东西）。 一个 `ProviderManager` 可以通过委托给一个 `AuthenticationProviders` 链来支持同一个应用程序中的多个不同认证机制。 如果一个 ProviderManager 不能识别一个特定的 `Authentication` 类型，它将被跳过。

`ProviderManager` 可以有一个父类认证器，如果所有的提供者返回null，则将再交给父类去认证。 如果父类不可用，则会导致 `AuthenticationException`。

有时应用程序具有受保护资源的逻辑组（例如所有与路径模式/ api / **相匹配的Web资源），并且每个组可以具有其自己的专用 `AuthenticationManager`。 通常，每个人都是一个 `ProviderManager`，他们共享一个父类。 父母是一种“全局”资源，充当所有提供者的失败回调。

Spring Security 提供了一些配置帮助类来快速获得应用程序中设置的通用身份验证管理器功能。 最常用的帮助类是 `AuthenticationManagerBuilder`，它非常适用于设置内存，JDBC或LDAP 中的用户详细信息，或添加自定义的`UserDetailsService`。 以下是配置全局（父类）`AuthenticationManager`的应用程序示例：

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {
   ... // web stuff here
  @Autowired
  public initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }
}
```

##### 授权

授权核心策略是 `AccessDecisionManager`， 框架提供了三个实现，并将所有三个委托连接到一个 `AccessDecisionVoter` 链。

对象在 `AccessDecisionManager` 和 `AccessDecisionVoter` 的签名中是完全通用的 - 它表示用户可能想要访问的任何内容。

`ConfigAttributes` 也是相当通用的，用一些元数据表示安全对象的装饰，这些元数据决定了访问它所需的权限级别。

Spring Security 作为一个单独的过滤器安装在链中，其配置类型为 `FilterChainProxy`。

在Spring Boot应用程序中，安全过滤器是ApplicationContext中的`@Bean`，并具有默认配置，以便将其应用于每个请求。它被安装在由 `SecurityProperties.DEFAULT_FILTER_ORDER` 定义的位置，而该位置又由`FilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER`（Spring Boot应用程序在包装请求时修改其行为的期望过滤器的最大顺序）决定。

![1](/images/security-filters.png)

##### 核心组件

###### SecurityContextHolder, SecurityContext和Authentication 对象

最根本的对象是`SecurityContextHolder，应用程序的当前安全环境的细节存储到它里边，它也包含了应用当前使用的主体细节。默认情况下`SecurityContextHolder`使用`ThreadLocal存储这些信息，这意味着，安全环境在同一个线程执行的方法一直是有效的， 即使这个安全环境没有作为一个方法参数传递到那些方法里。

##### FILTER

```
UsernamePasswordAuthenticationFilter
```

在Spring Security中,保存`SecurityContext`的任务落在了`SecurityContextPersistenceFilter`身上，

#### 接入方式

##### 创建Spring Security的Java 配置类
* 创建类SecurityConfiguration继承WebSecurityConfigurerAdapter，来对应用中所有的安全相关的事项（所有url，验证用户名密码，表单重定向等）进行控制，这个配置在你的应用程序中创建一个springSecurityFilterChain 的Servlet的过滤器。
* @EnableWebSecurity 注解启用Web安全功能。
* 初始化springSecurityFilter注册类，继承类AbstractSecurityWebApplicationInitializer。

 WebSecurityConfigurerAdapter共有三个configure方法

```
configure(WebSecurity) 通过重载，配置Spring Security的Filter链
configure(HttpSecurity) 通过重载，配置如何通过拦截器保护请求
configure(AuthenticationManagerBuilder) 通过重载，配置user-detail服务
```

##### 密码加密策略

* NoOpPasswordEncoder 明文方式保存

* BCtPasswordEncoder 强hash方式加密

* StandardPasswordEncoder SHA-256方式加密

* 实现PasswordEncoder接口,自定义加密方式

  通过方法passwordEncoder传入对应的加密实例即可。

##### 请求拦截策略

spring security的请求拦截匹配有两种风格，ant风格和正则表达式风格。编码方式是通过重载configure(HttpSecurity)方法实现。

```
access(String)     如果给定的SpEL表达式计算结果为true，就允许访问
anonymous()        允许匿名用户访问
authenticated()    允许认证过的用户访问
denyAll()          无条件拒绝所有访问
fullyAuthenticated()   如果用户是完整认证的话（不是通过Remember-me功能认证的），就允许访问
hasAnyAuthority(String...)   如果用户具备给定权限中的某一个的话，就允许访问
hasAnyRole(String...)   如果用户具备给定角色中的某一个的话，就允许访问
hasAuthority(String)   如果用户具备给定权限的话，就允许访问
hasIpAddress(String)   如果请求来自给定IP地址的话，就允许访问
hasRole(String)   如果用户具备给定角色的话，就允许访问
not()   对其他访问方法的结果求反
permitAll()   无条件允许访问
rememberMe()   如果用户是通过Remember-me功能认证的，就允许访问
```



##### remember-me功能

未完待续

##### 防止CSRF

未完待续

##### 自定义登录页面

未完待续

#### 参考

[spring security 入门教程 简书](https://www.jianshu.com/p/76bfa6743ba9)

[Spring 官方教程：Spring Security 架构](http://www.spring4all.com/article/554)