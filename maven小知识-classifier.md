---
title: maven小知识-classifier
date: 2019-12-02 19:58:19
tags: maven
---

#### classifier

classifier通常用于区分从同一POM构建的具有不同内容的构件（artifact）。它是可选的，它可以是任意的字符串，附加在版本号之后。

##### 区分基于不同JDK版本的jar包

如果项目依赖json-lib-2.2.2-jdk13.jar。则XML配置内容如下

```xml
<dependency>  
    <groupId>net.sf.json-lib</groupId>   
    <artifactId>json-lib</artifactId>   
    <version>2.2.2</version>  
    <classifier>jdk13</classifier>    
</dependency>  
```

注意，如果json-lib没有提供，json-lib-2.2.2.jar。那么，设置依赖的时候，必须使用 classifier ，否则会报错，因为找不到指定的jar包。

##### 区分项目的不同组成部分

```xml
<dependency>  
    <groupId>net.sf.json-lib</groupId>   
    <artifactId>json-lib</artifactId>   
    <version>2.2.2</version>  
    <classifier>jdk15-javadoc</classifier>    
</dependency> 
```

即classifier中的内容附在版本号之后。

##### 参考

[Maven中classifier](https://blog.csdn.net/qiumengchen12/article/details/71688395)