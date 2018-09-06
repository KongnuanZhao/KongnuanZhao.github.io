title: Maven初体验（一）
date: 2015-03-27 13:35:33
author: 赵空暖
tags: maven
categories: maven
---
#Maven简介#
是一个采用纯Java编写的开 源项目管理工具。Maven采用了一种被称之为project object model (POM)概念来管理项目，所有的项目配置信息都被定义在一个叫做POM.xml的文件中，通过该文件，Maven可以管理项目的整个声明周期，包括编译，构建，测试，发布，报告等等。目前Apache下绝大多数项目都已经采用Maven进行管理。
#实践环境#
```bash
C:\Users\Administrator\Desktop>java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) Client VM (build 24.51-b03, mixed mode, sharing)

C:\Users\Administrator\Desktop>echo %JAVA_HOME%
C:\Program Files (x86)\Java\jdk1.7.0_51
```
下载maven并解压
```bash
G:\>jar xvf "apache-maven-3.3.1-bin.zip"
  已创建: apache-maven-3.3.1/
  已创建: apache-maven-3.3.1/boot/
  已解压: apache-maven-3.3.1/boot/plexus-classworlds-2.5.2.jar
  已创建: apache-maven-3.3.1/lib/
  已解压: apache-maven-3.3.1/lib/maven-embedder-3.3.1.jar
  已解压: apache-maven-3.3.1/lib/maven-settings-3.3.1.jar
  .....
  已解压: apache-maven-3.3.1/lib/ext/README.txt

G:\>
```
配置环境变量
```bash
M2_HOME:G:\apache-maven-3.3.1
PATH:G:\apache-maven-3.3.1\bin;
```
查看Maven安装情况
```bash
C:\Users\Administrator\Desktop>echo %M2_HOME%
G:\apache-maven-3.3.1

C:\Users\Administrator\Desktop>mvn -v
Apache Maven 3.3.1 (cab6659f9874fa96462afef40fcf6bc033d58c1c; 2015-03-14T04:10:27+08:00)
Maven home: G:\apache-maven-3.3.1
Java version: 1.7.0_51, vendor: Oracle Corporation
Java home: C:\Program Files (x86)\Java\jdk1.7.0_51\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 7", version: "6.1", arch: "x86", family: "windows"
```
基于linux的Maven的安装可以参看[Ubuntu搭建Thrift和Maven](http://www.zhaokongnuan.com/2015/03/18/thrift-maven/)

#Maven初体验#
##Maven目录结构##
![maven-dir](/image/maven-dir.jpg)

##简单的例子##
首先简单编写`pom.xml`配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
                      http://maven.apache.org/maven-v4_0_0.xsd"> 
					  <modelVersion>4.0.0</modelVersion>
					  <groupId>com.zkn.mavenstudy</groupId>
					  <artifactId>hellomaven</artifactId>
					  <version>1.0-SNAPSHOT</version>
					  <name>Maven Study</name>
</project>
```
编写java代码
```java
G:\maven-study\Hello-maven>mvn clean compile
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Study 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ hellomaven ---
[INFO] Deleting G:\maven-study\Hello-maven\target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ hellomaven ---
[WARNING] Using platform encoding (GBK actually) to copy filtered resources, i.e. build is platform
dependent!
[INFO] skip non existing resourceDirectory G:\maven-study\Hello-maven\src\main\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ hellomaven ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding GBK, i.e. build is platform depend
ent!
[INFO] Compiling 1 source file to G:\maven-study\Hello-maven\target\classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.448 s
[INFO] Finished at: 2015-03-27T14:26:27+08:00
[INFO] Final Memory: 9M/23M
[INFO] ------------------------------------------------------------------------
```
可以看到输出没有错误，编译成功！接下来对HelloMaven类进行测试。

##Maven测试##
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
                      http://maven.apache.org/maven-v4_0_0.xsd"> 
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.zkn.mavenstudy</groupId>
	<artifactId>hellomaven</artifactId>
	<version>1.0-SNAPSHOT</version>
	<name>Maven Study</name>
	<dependencies>
	<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.7</version>
	<scope>test</scope>
	</dependency>
	</dependencies>
</project>
```
测试java代码
```java
package com.zkn.mavenstudy.hellomaven;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class HelloMavenTest{
   @Test
   public void testsayHello(){
        HelloMaven helloMaven=new HelloMaven();
		String result=helloMaven.sayHello();
		assertEquals("Hello Maven",result);
   }
}
```
执行测试
```bash
G:\maven-study\Hello-maven>mvn clean test
这里是省略号……
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.zkn.mavenstudy.hellomaven.HelloMavenTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.039 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 19.410 s
[INFO] Finished at: 2015-03-27T14:42:14+08:00
[INFO] Final Memory: 12M/29M
[INFO] ------------------------------------------------------------------------

```
测试成功，没有错误。下面学习一下打包.

##Maven打包##
```bash
G:\maven-study\Hello-maven>mvn clean package
这里是省略号……
[INFO] Building jar: G:\maven-study\Hello-maven\target\hellomaven-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 9.272 s
[INFO] Finished at: 2015-03-27T14:46:05+08:00
[INFO] Final Memory: 13M/32M
[INFO] ------------------------------------------------------------------------

```
打包完了如何让其他的maven项目直接引用这个jar呢，需要安装。

##Maven安装##
```bash
G:\maven-study\Hello-maven>mvn clean install
这里是省略号……
[INFO] Installing G:\maven-study\Hello-maven\target\hellomaven-1.0-SNAPSHOT.jar to C:\Users\Administ
rator\.m2\repository\com\zkn\mavenstudy\hellomaven\1.0-SNAPSHOT\hellomaven-1.0-SNAPSHOT.jar
[INFO] Installing G:\maven-study\Hello-maven\pom.xml to C:\Users\Administrator\.m2\repository\com\zk
n\mavenstudy\hellomaven\1.0-SNAPSHOT\hellomaven-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 10.898 s
[INFO] Finished at: 2015-03-27T14:48:14+08:00
[INFO] Final Memory: 12M/29M
[INFO] ------------------------------------------------------------------------

```
可以看到把这个项目输出的jar文件安装到了Maven本地仓库` C:\Users\Administ
rator\.m2\repository`当中。
查看本地仓库`hellomaven-1.0-SNAPSHOT.jar\META-INF`
```bash
Manifest-Version: 1.0
Built-By: Administrator
Build-Jdk: 1.7.0_51
Created-By: Apache Maven 3.3.1
Archiver-Version: Plexus Archiver
``` 
默认打包生成的jar包是不能直接运行的，因为带有main方法的类不会添加到manifect中，从上面的`hellomaven-1.0-SNAPSHOT.jar\META-INF`文件中可以看到没有`Main-Class`。为了生成可执行的jar文件，需要借助于`maven-shade-plugin`这个插件，配置`pom.xml`如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
                      http://maven.apache.org/maven-v4_0_0.xsd"> 
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.zkn.mavenstudy</groupId>
	<artifactId>hellomaven</artifactId>
	<version>1.0-SNAPSHOT</version>
	<name>Maven Study</name>
	
	<dependencies>
	<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.7</version>
	<scope>test</scope>
	</dependency>
	</dependencies>
	
	<build>
	<plugins>
	<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>1.2.1</version>
    <executions>
      <execution>
        <phase>package</phase>
        <goals>
          <goal>shade</goal>
        </goals>
        <configuration>
          <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
              <mainClass>com.zkn.mavenstudy.hellomaven.HelloMaven</mainClass>
            </transformer>
          </transformers>
        </configuration>
      </execution>
    </executions>
	</plugin>
	</plugins>
	</build>
</project>
```

```bash
Manifest-Version: 1.0
Build-Jdk: 1.7.0_51
Built-By: Administrator
Created-By: Apache Maven 3.3.1
Main-Class: com.zkn.mavenstudy.hellomaven.HelloMaven
Archiver-Version: Plexus Archiver

```
运行测试一下
```java
G:\maven-study\Hello-maven>java -jar target\hellomaven-1.0-SNAPSHOT.jar
Hello Maven
```
可以运行，完成测试！

##Archetype骨架构建##
Maven为我们提供了archetype可以帮助我们迅速勾勒出项目文件目录骨架。
```bash
G:\maven-study\Hello-archetype>mvn archetype:generate
这里是省略号……
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 574:
Choose org.apache.maven.archetypes:maven-archetype-quickstart version:
1: 1.0-alpha-1
2: 1.0-alpha-2
3: 1.0-alpha-3
4: 1.0-alpha-4
5: 1.0
6: 1.1
Choose a number: 6:
Define value for property 'groupId': : com.zkn.archetypetest
Define value for property 'artifactId': : archTest
Define value for property 'version':  1.0-SNAPSHOT: :
Define value for property 'package':  com.zkn.archetypetest: :  com.zkn.archetypetest.archTest
Confirm properties configuration:
groupId: com.zkn.archetypetest
artifactId: archTest
version: 1.0-SNAPSHOT
package:  com.zkn.archetypetest.archTest
 Y: :
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Old (1.x) Archetype: maven-archetype-qui
ckstart:1.1
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.zkn.archetypetest
[INFO] Parameter: packageName, Value:  com.zkn.archetypetest.archTest
[INFO] Parameter: package, Value:  com.zkn.archetypetest.archTest
[INFO] Parameter: artifactId, Value: archTest
[INFO] Parameter: basedir, Value: G:\maven-study\Hello-archetype
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] project created from Old (1.x) Archetype in dir: G:\maven-study\Hello-archetype\archTest
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 18:25 min
[INFO] Finished at: 2015-03-27T15:31:51+08:00
[INFO] Final Memory: 10M/31M
[INFO] ------------------------------------------------------------------------

```
构建成功。

Maven初体验结束，酷酷哒！