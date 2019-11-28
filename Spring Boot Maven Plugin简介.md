### Spring Boot Maven Plugin简介

##### 在pom.xml中引入

```xml
<build>
  <plugins>
   <plugin>
			<groupId>org.springframework.boot</groupId> 
     	<artifactId>spring-boot-maven-plugin</artifactId>
     	<version>2.0.5.RELEASE</version>
			<executions>
     		<execution>
      		<goals>
						<goal>repackage</goal> 
       		</goals>
          <configuration>
          	...
          </configuration>
     		</execution>
    	</executions>
   </plugin>
  </plugins>
</build>
```

#### 包含的goals

##### spring-boot:run 

运行SpringBoot应用

##### spring-boot:repackage 

将jar/war包重新包装成可执行。

使用 layout=NONE 可以将依赖打入jar包, 但非可执行jar包。

###### classifier

因为默认情况下, repackage将会替换原jar包, 故当项目被别的项目依赖时, 可添加classifier, 以做区分。

```xml
...
<configuration>
	<classifier>exec</classifier>
</configuration>
...
```

attach

将会生成可执行jar包和original包, 只有original会被installe和deployed(jar包名称不会带origina)l

```xml

<configuration>
  <attach>false</attach>
</configuration>
```



##### spring-boot:start and spring-boot:stop 

在mvn integration-test阶段, 管理SpringBoot应用的生命周期。

###### 分配随机端口

[random port](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/maven-plugin/examples/it-random-port.html)

###### 跳过完整性测试

添加属性，用于configuration。

[skip test](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/maven-plugin/examples/it-skip.html)

###### 指定profle

如下激活了foo和bar的profile

```xml
<configuration>
  <profiles>
    <profile>foo</profile>
    <profile>bar</profile>
  </profiles>
</configuration>
```



##### spring-boot:build-info 

基于当前maven项目生成build-info.propertie，可被Actuator使用。

##### spring-boot:help	

展示关于spring-boot-maven-plugin的帮助信息.
如下所示

```
mvn spring-boot:help -Ddetail=true -Dgoal=<goal-name> 
```

#### 参考

[Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/maven-plugin/plugin-info.html)