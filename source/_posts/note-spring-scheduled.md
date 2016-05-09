title: Spring自带的@Scheduled定时任务功能
date: 2015-09-11 11:23:00
categories: note
tags: [Spring, Quartz]
---

## Spring自带的@Scheduled定时任务功能

Spring有自带的定时任务功能，配置简单方便，比集成quartz.jar第三方简单很多，如果不是需要持久化动态管理quartz的话，用spring自带的就可以啦。

超级简单。
**第一步**：在spring配置文件头上加入一下配置

xmlns 多加下面的内容

```xml
 xmlns:task="http://www.springframework.org/schema/task"
```

xsi:schemaLocation多加下面的内容

```xml
 http://www.springframework.org/schema/task  
 http://www.springframework.org/schema/task/spring-task-3.1.xsd  
```

**第二步**：配置中加入这一行:

```xml
<task:annotation-driven/> 
```

**第三步**：开发Quartz类，用注解配置:

```java
@Component
public class QuartzJob {
	
	@Scheduled(cron="*/10 * * * * ?")
	public void sy(){//每隔10秒打印一次
		System.out.println("quartz job execute -------"+System.currentTimeMillis());
	}

}
```

至此，spring自带的定时任务功能就开发完了，超级简单实用。
