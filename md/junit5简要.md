---
title: JUnit5简要
tags: test
date: 2020-05-14 11:25:17

---

### 前言

针对每个要求, 必须至少有两个单元测试用例: 一个正测试, 一个负测试。

该文只是描述了一部分知识点, 系统学习请参考[官方文档](https://junit.org/junit5/docs/current/user-guide/)或[博客](https://blog.csdn.net/ryo1060732496/article/details/80792246)。

maven示例如未指明, 使用的是`maven-surefire-plugin`。

### 编写测试

#### 注解

[junit提供的注解](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)

* @Test注解不要导入的是junit4的`org.junit.Test`。

* 使用@Diasbled时, 请详细标注不启用的原因。

* `@Enable*`和`@Disable*`, 可以相互配合使用, 但是对于同一个注解, 只有第一个被发现的会使用, 其它的会被忽略, 例如将某个`@Enable*`作为Meta-Annotations标注在自定义注解上, 虽然junit会自动识别该注解, 但其顺序在直接标注在方法的注解之后。

* @Tag可以将测试分组, 可以激活指定的测试, 与maven结合使用配置如下。

  ```xml
  ...
  <configuration>
    <!--通过maven的参数来控制执行哪个Tag的测试-->
    <groups>${junit.tag.expression}</groups>
  </configuration>
  ...
  
  ```

* 使用@TestMethodOrder指定方法执行顺序, 可使用MethodOrder的三种默认实现。

* Test Instance LifeCycle, 使用@TestInstance指定, 默认为LifeCycle.PER_METHOD。

* 因内部类不可以有静态成员和静态方法, 故不能声明@BeforeAll和@AfterAll, 可通过指定LifeCycle.PER_CLASS来绕过该限制, 即此时这两个注解可声明在普通方法上。

* 可以为constructor和method注入三种类型参数, 来获取对应信息: TestInfo、RepetitionInfo、TestReporter。

* 参数化测试, 通过@ParameterizedTest, 至少需指定一个参数来源, [parameterized-tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)。

* Assumptions异常时, 只是跳过当前测试, 不会中断整个测试流程。

* @RepeatedTest, 重复执行测试。

* DynamicTest, 方法返回值为DynamicNode的Stream、Collection、Iterable、Iterator即可。

#### Extensions Model

@ExtendWith、Extension、@RegisterExtension

* 使用@ExtendWith和@RegisterExtension来注册Extension的扩展, 可以用来声明各种生命周期回调等。
* 还可以通过ServiceLoader的机制加载Extension, 通过`junit.jupiter.extensions.autodetection.enabled=true`启用, 然后`/META-INF/services/org.junit.jupiter.api.extension.Extension `中声明。
* 使用的具体例子, 官方推荐`MockitoExtension`及`SpringExtension`。
* ExecutionCondition、TestInstanceFactory、TestInstancePostProcessor、TestInstancePreDestroyCallback。
* TestWatcher可在testDisabled、testSuccessful、testAborted、testFailed时实现对应的逻辑处理。
* TestExecutionExceptionHandler、LifecycleMethodExecutionExceptionHandler, 处理指定的异常, 避免因此导致失败。
* 可以通过InvocationInterceptor进行对应处理。
* 使用Store来存储信息, 如SpringExtension中的`context.getRoot().getStore(NAMESPACE);`。
* 尽量使用junit提供的工具类: AnnotationSupport、ClassSupport、ReflectionSupport、ModifierSupprt。
* [extensions-execution-order-diagram](https://junit.org/junit5/docs/current/user-guide/#extensions-execution-order-diagram)。

### 运行测试

#### IDEA

使用2017.3以后版本。

#### Maven

##### maven-surefire-plugin

文件名格式

- `**/Test*.java`
- `**/*Test.java`
- `**/*Tests.java`
- `**/*TestCase.java`

`maven-surefire-pluigin`默认将排除所有内嵌类, 可使用以下配置修改

```xml
...
<configuration>
  <!--exclude all nested classes (including static member classes) by default.
    override its exclude rules as follows.
    https://junit.org/junit5/docs/current/user-guide/#writing-tests-built-in-extensions
    -->
  <excludes>
    <exclude/>
  </excludes>
</configuration>
...
```

配置Configuration Parameters，其余方式参考[Configuration Parameters](https://junit.org/junit5/docs/current/user-guide/#running-tests-config-params)。

```xml
....
<configuration>
  <properties>
    <configurationParameters>
      junit.jupiter.conditions.deactivate = *
      junit.jupiter.extensions.autodetection.enabled = true
      junit.jupiter.testinstance.lifecycle.default = per_class
    </configurationParameters>
  </properties>
</configuration>
...
```

### 下回分解

* [Spring test](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html)
* [maven-failesafe-plugin](https://maven.apache.org/surefire/maven-failsafe-plugin/)

