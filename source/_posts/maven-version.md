title: Maven(二)用MyEclipse建立Maven骨架的项目
date: 2016-01-30 19:44:46
author: 赵空暖
tags: maven
categories: maven
---
#实践环境#
Myeclipse 10.7
Java Version "1.7.0_51"
M2_HOME=G:\apache-maven-3.3.1
#项目构建#
![maven_tree](/image/maven-tree.png)
Maven 项目结构分析 
	`src/main/java` 存放项目源码
	`src/main/resources` 存放项目配置文件 
	`src/test/java` 存放测试用例代码
	`src/test/resources` 存放测试配置文件 
	
	`src/main/webapp` 文件夹用来存放页面代码 

问题：
```bash
myeclipe java.lang.UnsupportedClassVersionError....51.0 
```
beacase,
java.lang.UnsupportedClassVersionError happens because of a higher JDK during compile time and lower JDK during runtime.

解决：
在`G:\apache-maven-3.3.1\conf`的`settings.xml`加
```xml
 <profile>
            <id>jdk-1.7</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.7</jdk>
            </activation>
            <properties>
                <maven.compiler.source>1.7</maven.compiler.source>
                <maven.compiler.target>1.7</maven.compiler.target>
                <maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
            </properties>
  </profile>
```

问题：
```bash
Java compiler level does not match the version of the installed Java project facet
```
解决：
在创建好的maven project的`pom.xml`里加
```xml
  <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
之后Upload Project

运行`run as --> maven build -->tomcat:run`
报错：
```bash
-Dmaven.multiModuleProjectDirectory system propery is not set. Check $M2_HOME environment variable and mvn script match.
```
解决：
环境变量M2_HOME指向maven安装目录
`M2_HOME=G:\apache-maven-3.3.1`
然后在`Window --> Preference -->Java --> Installed JREs -->Edit`
在Default VM arguments中设置
-Dmaven.multiModuleProjectDirectory=$M2_HOME

运行成功
```bash
...
...
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ nuanbos_maven ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] <<< tomcat-maven-plugin:1.1:run (default-cli) < compile @ nuanbos_maven <<<
[INFO] 
[INFO] --- tomcat-maven-plugin:1.1:run (default-cli) @ nuanbos_maven ---
[INFO] Running war on http://localhost:8080/nuanbos_maven
...
...
```

