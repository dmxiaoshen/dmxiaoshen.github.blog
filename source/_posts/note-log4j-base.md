title: Log4j简单使用与配置分析
date: 2014-11-24 15:14:11
categories: note
tags: [log4j, slf4j]
---

## Log4j简单使用与配置分析

### log4j.properties  

```xml
#设置级别和目的地
log4j.rootLogger=debug,Console,SingleFile,RollingFile,DailyRollingFile

#Console 控制台
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
#自定义样式  
# %r 时间 0  
# %t 方法名 main  
# %p 优先级 DEBUG/INFO/WARN/ERROR  
# %c 所属类的全名(包括包名)  
# %l 发生的位置，在某个类的某行  
# %m 输出代码中指定的讯息，如log(message)中的message  
# %n 输出一个换行  
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
# %-5p的意思是最多5个字符，不足时左对齐

#SingleFile 追加文件
log4j.appender.SingleFile=org.apache.log4j.FileAppender
log4j.appender.SingleFile.File=D:/demo1.log
log4j.appender.SingleFile.layout=org.apache.log4j.PatternLayout
log4j.appender.SingleFile.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n

#RollingFile 文件大小到达指定尺寸的时候产生新文件
log4j.appender.RollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.RollingFile.MaxFileSize=100MB
log4j.appender.RollingFile.MaxBackupIndex=10
log4j.appender.RollingFile.File=D:/demo2.log
log4j.appender.RollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.RollingFile.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n

#DailyRollingFile 每天产生一个日志文件
log4j.appender.DailyRollingFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.DailyRollingFile.File=D:/demo3.log
log4j.appender.DailyRollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.DailyRollingFile.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n

#设置特定包的级别
log4j.logger.com.charles=DEBUG

#log4jdbc相关,需要依赖slf4j日志框架,(日志信息如果全部为off,log4jdbc将不会生效,因此对性能没有任何影响)
log4j.logger.jdbc.sqlonly=OFF  
log4j.logger.jdbc.sqltiming=INFO  
log4j.logger.jdbc.audit=OFF  
log4j.logger.jdbc.resultset=OFF  
log4j.logger.jdbc.connection=OFF  
```

### 几个需要强调的点

Log4j由三个重要的组件构成：**日志信息的优先级**，**日志信息的输出目的地**，**日志信息的输出格式**。  
日志信息的**优先级从高到低**有ERROR、WARN、 INFO、DEBUG，分别用来指定这条日志信息的重要程度；  
日志信息的输出目的地指定了日志将打印到控制台还是文件中；而输出格式则控制了日志信息的显示内容。  

#### Appender日志目的地  

* ConsoleAppender 目的地为控制台的Appender
* FileAppender 目的地为文件的Appender 
* RollingFileAppender 目的地为大小受限的文件的Appender 
* DailyRollingFileAppender 目的地为每天一个文件的Appender 

#### Layout日志格式化器

* org.apache.log4j.HTMLLayout(以HTML表格形式布局)
* org.apache.log4j.PatternLayout(可以灵活地指定布局模式)
* org.apache.log4j.SimpleLayout(包含日志信息的级别和信息字符串)
* org.apache.log4j.TTCCLayout(包含日志产生的时间、线程、类别等等信息)


```xml
#PatternLayout  
# %r 时间 0  
# %t 方法名 main  
# %p 优先级 DEBUG/INFO/WARN/ERROR  
# %c 所属类的全名(包括包名)  
# %l 发生的位置，在某个类的某行  
# %m 输出代码中指定的讯息，如log(message)中的message  
# %n 输出一个换行  
```

#### 使用log4jdbc的好处  

**现大家使用的ibatis,hibernate,spring jdbc的sql日志信息，有一点个缺点是占位符与参数是分开打印的。而log4jdbc是在jdbc层的一个日志框架，可以将占位符与参数全部合并在一起显示。**    

### slf4j  

在不使用slf4j的情况下，一般为了提高效率，可以这么写:  

```java
// 记录debug级别的信息
if (logger.isDebugEnabled()) {
	logger.debug("This is debug message from Dao.");
}

// 记录info级别的信息
if (logger.isInfoEnabled()) {
	logger.info("This is info message from Dao.");
}

// 记录error级别的信息
logger.error("This is error message from Dao.");
```

如果用slf4j,则可以这么写:  

```java
logger.debug("Processing trade with id: {} and symbol : {} ", id, symbol);
logger.debug("Value {} was inserted between {} and {}.", new Object[] {newVal, below, above});//多个参数
```

**为什么这么做？**  

> 占位符(place holder)，在代码中表示为"{}"的特性。占位符是一个非常类似于在String的format()方法中的%s，因为它会在运行时被某个提供的实际字符串所替换。这不仅降低了你代码中字符串连接次数，而且还节省了新建的String对象。即使你可能没需要那些对象，但这个依旧成立，取决于你的生产环境的日志级别，例如在DEBUG或者INFO级别的字符串连接。因为String对象是不可修改的并且它们建立在一个String池中，它们消耗堆内存( heap memory)而且大多数时间他们是不被需要的，例如当你的应用程序在生产环境以ERROR级别运行时候，一个String使用在DEBUG语句就是不被需要的。通过使用SLF4J,你可以在运行时延迟字符串的建立，这意味着只有需要的String对象才被建立。而如果你已经使用log4j，那么你已经对于在if条件中使用debug语句这种变通方案十分熟悉了，但SLF4J的占位符就比这个好用得多。  


> 1. 在你的开源或内部类库中使用SLF4J会使得它独立于任何一个特定的日志实现，这意味着不需要管理多个日志配置或者多个日志类库，你的客户端会很感激这点。
> 2. SLF4J提供了基于占位符的日志方法，这通过去除检查isDebugEnabled(), isInfoEnabled()等等，提高了代码可读性。
> 3. 通过使用SLF4J的日志方法，你可以延迟构建日志信息（Srting）的开销，直到你真正需要，这对于内存和CPU都是高效的。
> 4. 作为附注，更少的暂时的字符串意味着垃圾回收器（Garbage Collector）需要做更好的工作，这意味着你的应用程序有为更好的吞吐量和性能。
> 5. 这些好处只是冰山一角，你将在开始使用SL4J和阅读其中代码的时候知道更多的好处。我强烈建议，任何一个新的Java程序员，都应该使用SLF4J做日志而不是使用包括Log4J在内的其他日志API。


一般spring中配置log4j，只需在web.xml文件中加入:

```xml
<context-param>
 <param-name>log4jConfigLocation</param-name>
 <param-value>\WEB-INF\classes\log4j.properties</param-value>
</context-param>
```

如果是maven管理,在pom.xml文件中加入依赖(log4j包会被当做slf4j的依赖相关包导入进来):

```xml
<dependency>
 <groupId>org.slf4j</groupId>
 <artifactId>slf4j-log4j12</artifactId>
 <version>1.6.4</version>
</dependency>
```


参考网址:   
<http://www.iteye.com/topic/378077>  
<http://blog.csdn.net/kobejayandy/article/details/17306613>