---
title: springsecurity参考手册
tags: JAVA
date: 2019-03-30 16:11:58
---

### 创建java配置

![1553933604145](C:\Users\fengchaofan\AppData\Roaming\Typora\typora-user-images\1553933604145.png)

1.使用@EnableWebSecurity 注解启用Web安全功能

2.创建类SecurityConfiguration继承WebSecurityConfigurerAdapter，来对应用中所有的安全相关的事项（所有url，验证用户名密码，表单重定向等）进行控制，这个配置在你的应用程序中创建一个springSecurityFilterChain 的Servlet的过滤器，负责所有安全。

WebSecurityConfigurerAdapter共有三个configure方法

```
configure(WebSecurity) 通过重载，配置Spring Security的Filter链
configure(HttpSecurity) 通过重载，配置如何通过拦截器保护请求
configure(AuthenticationManagerBuilder) 通过重载，配置user-detail服务
```
#### Filter链配置
#### HttpSecurity配置

```
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()
			.anyRequest().authenticated()
			.and()
		.formLogin()
			.and()
		.httpBasic();
}
```

上面的默认配置:

- 应用中的所有请求都需要用户被认证
- 允许用户进行基于表单的认证(WEB可用)
- 允许用户使用HTTP基于验证进行认证

#### 用户存储配置
configure(AuthenticationManagerBuilder) 方法中可配置多个用户存储。
#####  JDBC配置
```
@Autowired
private DataSource dataSource;

@Override
public void configure(AuthenticationManagerBuilder auth)throws Exception{
    //基于数据库的用户存储、认证
    auth.jdbcAuthentication().dataSource(dataSource)
        .usersByUsernameQuery("select account,password,true from user where account=?")
        .authoritiesByUsernameQuery("select account,role from user where account=?")；
}
```
##### LDAP配置
```
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
	auth
		.ldapAuthentication()
			.userDnPatterns("uid={0},ou=people")
			.groupSearchBase("ou=groups");
}
```
##### 自定义设置
实现接口UserDetailsService的loadUserByUsername(String username)，返回的是代表用户的UserDetails对象。 
然后通过auth.userDetailsService()方法将其设置到安全配置中,userDetailsService()类似于jdbcAuthentication()和ldapAuthentication()，配置了一个用户存储。


### 方法安全

在任何使用`@Configuration`的实例上，使用`@EnableGlobalMethodSecurity`注解来启用基于注解的安全性,添加一个注解到一个方法（或者一个类）限制对相应方法的访问。

可以在同一个应用程序中启用多种注解，但是在一个接口或者类中只能使用一种类型的注解，否则会出现不明确的行为。如果对特定的方法使用了两个注解，只有其中的一个会被应用。

![1553938519459](C:\Users\fengchaofan\AppData\Roaming\Typora\typora-user-images\1553938519459.png)

![1553938541419](C:\Users\fengchaofan\AppData\Roaming\Typora\typora-user-images\1553938541419.png)

### 当前用户获取信息

将如下代码写在类的方法中，交由Spring管理

![1554033780770](C:\Users\fengchaofan\AppData\Roaming\Typora\typora-user-images\1554033780770.png)