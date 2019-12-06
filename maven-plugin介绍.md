---
title: maven plugin介绍
date: 2019-12-02 19:58:19
tags: maven
---

介绍工作中用到的maven插件, 会时常补全。



### antrun plugin

[Apache Maven AntRun Plugin](http://maven.apache.org/plugins/maven-antrun-plugin/)

#### 简介

提供了运行ant task的能力,  帮助基于ant构建的项目进行迁移。

#### 引入

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>1.8</version>
        <executions>
        	<execution>
            	...
            </execution>
        </executions>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```

#### goals

#### antrun:help

展示maven-antrun-plugin的帮助信息。

```shell
 mvn antrun:help -Ddetail=true -Dgoal=<goal-name>
```

* detail: 展示所有可配置的属性, 默认为false。
* goal: 指定需展示的goal, 不指定时, 展示所有的goal。

#### antrun:run

Maven AntRun Mojo, 用来运行ant tasks, 最好将ant tasks放在build.xml中。

[antrun:run](http://maven.apache.org/plugins/maven-antrun-plugin/run-mojo.html)

#### usage

在target标签中定义行为,  否则将不会进行任何操作,  如下格式。

```xml
...
 <execution>
     <phase> <!-- a lifecycle phase --> </phase>
     <configuration>
         <target>
             <!--do something-->
         </target>
     </configuration>
     <goals>
         <goal>run</goal>
     </goals>
</execution>
...
```

#### 总结

用来帮助ant项目的迁移。

通过phase指定在某个maven lifecycle执行target, 执行特定任务。

* 可以用来将项目生成为自己想要的打包结构, 便于部署。

```xml
...
<executions>
    <execution>
        <id>copy-dependencies</id>
        <phase>package</phase>
        <goals>
            <goal>run</goal>
        </goals>
        <configuration>
                <target>
                    <delete dir="${project.build.directory}/../docker"/>        
                    <copy todir="{project.build.directory}/../docker/resource"
                        <fileset dir="${project.build.outputDirectory}">
                            <include name="*.properties"/>
                        </fileset>
                    </copy>		
                    <copy file="${project.build.directory}/${project.build.finalName}.jar"             tofile="${project.build.directory}/../docker/${project.build.finalName}.jar"/>
                </target>
            </configuration>
	</execution>
</executions>
```

* 克隆git submodule。

  ```xml
  ...
  <execution>
      <id>validate</id>
      <phase>validate</phase>
      <configuration>
          <target>
              <exec executable="git">
                  <arg value="submodule"/>
                  <arg value="update"/>
                  <arg value="--init"/>
                  <arg value="--recursive"/>
              </exec>
              <exec executable="git">
                  <arg value="submodule"/>
                  <arg value="update"/>
                  <arg value="--remote"/>
              </exec>
          </target>
      </configuration>
      <goals>
          <goal>run</goal>
      </goals>
  </execution>
  ...
  ```

  