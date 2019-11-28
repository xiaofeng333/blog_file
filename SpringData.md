### SpringData

Spring Data存储库抽象的目标是显着减少为各种持久性存储实现数据访问层所需的样板代码量。

Spring Data存储库抽象中的中央接口是Repository。它将域类以及域类的ID类型作为类型参数进行管理。此接口主要用作标记接口，用于捕获要使用的类型，并帮助您发现扩展此接口的接口。(CrudRepository, PagingAndSortingRepository)

#### Querydsl

使用Querydsl支持，Repository需继承QuerydslPredicateExecutor

```java
interface UserRepository extends CrudRepository<User, Long>, QuerydslPredicateExecutor<User> {
}
```

需引入依赖

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>${querydsl.version}</version>
</dependency>

<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>${querydsl.version}</version>
</dependency>
```

添加插件

```xml
<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
            </configuration>
        </execution>
    </executions>
</plugin>
```

