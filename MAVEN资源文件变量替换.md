---
title: MAVEN资源文件变量替换
tags: JAVA
date: 2019-05-19 18:10:55
---

项目在不同环境下部署时，使用的属性是不同的，这时可以使用MAVEN进行资源文件的过滤，在打包时传递参数，打出适合部署在对应环境的资源文件。

如定义了如下资源文件:

```properties
spring.env=${env}
```

pom文件中可添加如下信息

```xml
<project>
	<properties>
        
        <!--使用了 spring-boot-starter-parent 做项目版本管理，其默认值为@-->
        <resource.delimiter>${}</resource.delimiter>
        
        <!--默认值-->
        <env>DEV</env>
    </properties>
    <build>
    	<!-- 配置文件的环境变量 -->
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <!--可通过includes指定需要进行过滤的文件-->
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>false</filtering>
            <!--可通过excludes指定排除需要进行过滤的文件, 这里的路径与上相同, 否则其他文件将不会被打包-->
        </resource>
    </resources>
    </build>
</project>
```

maven打包命令, 将该环境下需要使用的参数进行传递

```script
mvn clean install -Denv=PRO
```

当参数过多时，可同时使用profiles，将无需保密的参数定义在pom中，通过激活对应profile的形式来进行参数替换。易可通过使用apollo等平台，在此只需动态配置环境即可。

```
mvn clean install -P${profileId}
```

### 参考

[Maven打包时，环境变量替换,并解决spring-boot项目中${}无效的问题](<https://www.jianshu.com/p/cf3bd9ddfe6f>)