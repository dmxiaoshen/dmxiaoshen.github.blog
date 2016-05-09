title: Maven基础用法
date: 2014-11-17 21:17:11
categories: note
tags: [maven]
---

## Maven基础用法

> Maven是一个项目管理工具，它包含了一个项目对象模型 (Project Object Model)，一组标准集合，一个项目生命周期(Project Lifecycle)，
一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段(phase)中插件(plugin)目标(goal)的逻辑。当你使用
Maven的时候，你用一个明确定义的项目对象模型来描述你的项目，然后Maven可以应用横切的逻辑，这些逻辑来自一组共享的（或者自定义的）  
插件。    
Maven 有一个生命周期，当你运行 mvn install 的时候被调用。这条命令告诉 Maven 执行一系列的有序的步骤，直到到达你指定的生命周期。
遍历生命周期旅途中的一个影响就是，Maven 运行了许多默认的插件目标，这些目标完成了像编译和创建一个 JAR 文件这样的工作。此外，
Maven能够很方便的帮你管理项目报告，生成站点，管理JAR文件，等等。  

### 安装  
本地需要jdk环境，下载maven的zip包到本地，解压文件。例如：**E:\apache-maven-3.1.1**   
配置环境变量：  

* 新建变量**MAVEN_HOME**,值设为zip包的解压路径，如：**E:\apache-maven-3.1.1**   
* 修改**PATH**变量，增加**%MAVEN_HOME%\bin**  

### 创建一个简单java项目  
命令行执行:  
`mvn archetype:generate -DgroupId=com.dmxs -DartifactId=my-test -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`  
*-DinteractiveMode=false 表示不使用交互模式填写版本信息使用默认配置.*  
会在当前目录生成一个**my-test**的java项目。  
对于此项目可执行如下命令：  

* **mvn clean compile** 编译成功，在my-test目录下多出一个target目录，target\classes里面存放的就是编译后的class文件。  
* **mvn clean test** 测试成功，在my-test\target目录下会有一个test-classes目录，存放的就是测试代码的class文件。  
* **mvn clean package** 执行打包命令前，会先执行编译和测试命令.构建成功后，会再target目录下生成my-test-1.0-SNAPSHOT.jar包。  
* **mvn clean install** 执行安装命令前，会先执行编译、测试、打包命令. 构建成功，就会将项目的jar包安装到本地仓库。  
* **java -cp target\my-test-1.0-SNAPSHOT.jar com.dmxs.App** 命令行进入my-test目录，运行jar包。  

### 创建一个简单web项目  
命令行执行:  
`mvn archetype:generate -DgroupId=com.dmxs -DartifactId=my-web-app -DarchetypeArtifactId=maven-archetype-webapp -DinteractivMode=false`   
*-DinteractiveMode=false 表示不使用交互模式填写版本信息使用默认配置.*    
会在当前目录生成一个**my-web-app**的web项目。  
**对于此项目可执行的命令同上，唯一不同的是打包生成的是war包而非jar包**  

### 手动构建maven多模块项目  

#### 首先创建一个父项目plms  
命令行执行:  
`mvn archetype:create -DgroupId=com.dmxs.plms -DartifactId=plms -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveModel=false`  
这时会生成**plms**项目，该项目就是所需父项目，进入目录删除**src**文件夹，修改**pom.xml**文件.  
修改pom文件中的 `<packaging>jar</packaging>为 <packaging>pom</packaging>`:  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.dmxs.plms</groupId>
  <artifactId>plms</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>plms</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

#### 创建一个子项目plms-common   
进入plms目录，命令行执行:  
`mvn archetype:create -DgroupId=com.dmxs.plms -DartifactId=plms-common -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveModel=false`  
在plms目录中的pom.xml文件中新增:  

```xml
<modules>
 <module>plms-common</module>
</modules>
```

修改后文件如下:  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.dmxs.plms</groupId>
  <artifactId>plms</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>plms</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <modules>
    <module>plms-common</module>
  </modules>
</project>
```

修改plms-common下的pom.xml文件，`删除 <groupId>com.dmxs.plms</groupId>和<version>1.0-SNAPSHOT</version>，添加<packaging>jar</packaging>。`    
修改后的pom文件如下：

```xml
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" 
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <artifactId>plms</artifactId>
    <groupId>com.dmxs.plms</groupId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  <artifactId>plms-common</artifactId>
  <name>plms-common</name>
  <packaging>jar</packaging>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

#### 创建一个子项目plms-web  
大致与plms-common相同，只是maven-archetype-quickstart换成**maven-archetype-webapp**    
packaging修改为`<packaging>war</packaging>`  
如需在plms-web中依赖plms-common,则在plms-web中添加:  

```xml
<dependency>
	<groupId>com.dmxs.plms</groupId>
	<artifactId>plms-common</artifactId>
	<version>${project.version}</version>
</dependency>
```


参考网址：<http://blog.csdn.net/edward0830ly/article/details/8748986>




