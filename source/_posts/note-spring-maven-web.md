title: Spring+Maven+Hibernate的Web项目搭建
date: 2014-11-26 10:17:11
categories: note
tags: [spring, 项目]
---

## Spring+Maven+Hibernate的Web项目搭建

这里不详细叙述如何构建工程之类的，主要是对其中涉及的配置文件的分析和解释。简单罗列一下主要涉及到的配置文件:  

* **spring-common.xml** (spring配置文件，名字可以自取)
* **spring-mvc.xml** (spring MVC配置文件，名字可以自取)
* **web.xml** (web项目配置文件)
* **pom.xml** (maven依赖相关配置文件) 

### spring-common.xml  

先上完整文件:  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xsi:schemaLocation="http://www.springframework.org/schema/jdbc 
		http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
		http://www.springframework.org/schema/aop 
		http://www.springframework.org/schema/aop/spring-aop-3.1.xsd
		http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
		http://www.springframework.org/schema/tx 
		http://www.springframework.org/schema/tx/spring-tx-3.1.xsd
		http://www.springframework.org/schema/context 
		http://www.springframework.org/schema/context/spring-context-3.1.xsd">

	<!-- 自动注入扫描 -->
	<context:component-scan base-package="com.dmxiaoshen.examples" use-default-filters="false">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Repository" />
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Service" />
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Component" />
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
	</context:component-scan>

	<!-- 数据源配置，使用DBCP数据库连接池 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close">
		<!-- Connection Info -->
		<property name="driverClassName" value="${jdbc.driver}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />

		<!-- Connection Pooling Info -->
		<property name="defaultAutoCommit" value="false" />
		<!-- 超时设置 -->
		<property name="timeBetweenEvictionRunsMillis" value="3600000" />
		<property name="minEvictableIdleTimeMillis" value="3600000" />
	</bean>

	<!-- hibernate sessionFactory -->
	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
				<prop key="hibernate.current_session_context_class">
					org.springframework.orm.hibernate4.SpringSessionContext
				</prop>					
				<prop key="hibernate.cache.use_second_level_cache">false</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">false</prop>
				<prop key="hibernate.use_sql_comments">false</prop>
			</props>
		</property>
		<!-- 动态表名映射相关 -->
		<property name="namingStrategy">
			<bean class="org.hibernate.cfg.ImprovedNamingStrategy" />
		</property>
		<!-- 需要扫描的model所在包 -->
		<property name="packagesToScan">
			<value>com.dmxiaoshen.examples.*.entity</value>
		</property>
	</bean>		

	<!-- 事务开始 -->
	<!-- 事务管理 -->
	<bean id="transactionManager"
		class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>
	
	<!-- 使用annotation定义事务 -->
	<tx:annotation-driven transaction-manager="transactionManager" />

	<!-- 对哪些方法加配置事务 -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="*" propagation="REQUIRED"/>
		</tx:attributes>
	</tx:advice>

	<!-- 事务切在哪一层 -->
	<aop:config>
		<aop:pointcut id="allManagerMethod"
			expression="execution(* com.dmxiaoshen.examples.*.service.*.*(..))" />
		<aop:advisor advice-ref="txAdvice" pointcut-ref="allManagerMethod" />
	</aop:config>	
	<!-- 事务结束 -->

</beans>
```

对里面的几个点做下强调。  

#### `<context:component-scan>`  

其实，spring里面有`<context:annotation-config/>`标签，配置用以使用spring的各种注解，如@Resource ,@Antowired等，  
但是这里我们使用`<context:component-scan>`标签，这里配置了扫描包路径。`<context:include-filter>`和`<context:exclude-filter>`各代表引入和排除的过滤。  
这里之所以过滤掉了Controller扫描，是因为准备把他放到spring-mvc.xml中。

#### dataSource  

这里配置了数据库连接的一些信息。**${jdbc.driver}**，**${jdbc.url}**，**${jdbc.username}**，**${jdbc.password}**这四个参数我们会在稍后的profile.dev.properties,profile.production.proterties,  
profile.test.properties三个文件中分别配置，这在将pom.xml文件时会涉及到，明显这是在区分环境，maven打包是需要用到。这里我们只要知道，这些参数是动态可变的就行。

#### txAdvice  

这里我们做了`<tx:method name="*" propagation="REQUIRED"/>`配置，其实可以细化的配置，如: 

```xml
<tx:method name="save*" propagation="REQUIRED" />
<tx:method name="add*" propagation="REQUIRED" />
<tx:method name="get*" propagation="REQUIRED" read-only="true" />
<tx:method name="count*" propagation="REQUIRED" read-only="true" />
<tx:method name="find*" propagation="REQUIRED" read-only="true" />
<tx:method name="list*" propagation="REQUIRED" read-only="true" />
<tx:method name="query*" propagation="REQUIRED" read-only="true" />
...

```

#### `<aop:config> ` 

这里我们对所有`expression="execution(* com.dmxiaoshen.examples.*.service.*.*(..))"`下的方法都加了事务，  
简单点说就是 `examples.*.service.*` 包下所有类的所有方法。

#### `<property name="packagesToScan">`

这里如要配置多个包，可用:  

```xml
<list>
 <value>...</value>
 <value>...</value>
 <value>...</value>
</list>
```

### spring-mvc.xml  

先上完整文件:  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
		http://www.springframework.org/schema/context 
		http://www.springframework.org/schema/context/spring-context-3.1.xsd
		http://www.springframework.org/schema/mvc 
		http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd">
	
	<!-- 默认的注解映射的支持 --> 
	<mvc:annotation-driven />
	
	<!-- 自动扫描controller包下的所有类，使其认为spring mvc的控制器 -->
	<context:component-scan base-package="com.dmxiaoshen.examples.*.controller" use-default-filters="false">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
	</context:component-scan>
	
	<!-- 静态资源处理 -->
	<mvc:default-servlet-handler />
	
	<mvc:view-controller path="/" view-name="redirect:/login/index"/>
	
	<!-- 对模型视图名称的解析，即在模型视图名称添加前后缀 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/jsp/" />
		<property name="suffix" value=".jsp" />
	</bean>
			
</beans>
```

#### `<mvc:annotation-driven />`  

会自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter 两个bean,是spring MVC为@Controllers分发请求所必须的。  
这里我们这样写，spring会自动注入进来，配置是简单了，但是我们要知道是这里，注册了这两个bean。  

#### `<mvc:default-servlet-handler />`  

这是处理静态文件的一种方案，我们暂定这种方法为方案一。我们需要能直接访问静态资源，而不是被spring所拦截。  

方案二:  

```xml
<mvc:resources mapping="/styles/**" location="/static_resources/css/"/>    
<mvc:resources mapping="/images/**" location="/static_resources/images/"/>
<mvc:resources mapping="/js/**" location="/static_resources/js/"/>
```

#### `<mvc:view-controller>`  

重定向，这里是把"/",重定向到"/login/index"。这里可以定义多个`<mvc:view-controller>`标签，来满足各种重定向需求。  

#### InternalResourceViewResolver  

视图解析器，我们如上配置的话，"/login/index"其实完整路径是"/WEB-INF/jsp/login/index.jsp"。

### web.xml  

先上完整文件:  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" 
	 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	 http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">
  <display-name>examples</display-name>
  
  	<!-- 把一些spring相关的配置文件加载进来 -->
 	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath*:/spring-*.xml</param-value>
	</context-param>
	
	<!-- 把log4j日志相关的配置文件加载进来 -->
	<context-param>
		<param-name>log4jConfigLocation</param-name>
		<param-value>\WEB-INF\classes\log4j.properties</param-value>
	</context-param>
       
	<!-- 启动spring -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	
	<!-- spring mvc相关配置及文件加载 -->
	<servlet>
		<servlet-name>springMVC</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring-mvc.xml</param-value>
		</init-param>
		<!-- 标记容器是否在启动的时候就加载这个servlet。当值为0或者大于0时，表示容器在应用启动时就加载这个servlet；
			当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载。
			正数的值越小，启动该servlet的优先级越高。 -->
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>springMVC</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	
	<!-- Character Encoding filter -->
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
	<!-- 出错页面定义 -->
	<error-page>
		<exception-type>java.lang.Throwable</exception-type>
		<location>/WEB-INF/jsp/error/500.jsp</location>
	</error-page>
	<error-page>
		<error-code>500</error-code>
		<location>/WEB-INF/jsp/error/500.jsp</location>
	</error-page>
	<error-page>
		<error-code>404</error-code>
		<location>/WEB-INF/jsp/error/404.jsp</location>
	</error-page>
		
</web-app>
```

这里我们用到了log4j.properties文件，这是跟日志相关的。  

### pom.xml  

配置了上面3个文件，基本已经差不多了，而这个pom.xml才是涉及到maven相关的，这里会给出上面三个文件中需要的参数，以及依赖的  
jar包的引进，还有就是环境的选择(dev,production,test)。  

先上完整文件:  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
			    http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.dmxiaoshen.examples</groupId>
  <artifactId>examples</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>example Maven Webapp</name>
  <url>http://maven.apache.org</url>
  
   <properties>
    <jdk.version>1.6</jdk.version>
    <maven.test.skip>true</maven.test.skip>
    <mysql.version>5.1.13</mysql.version>
	<spring.version>3.1.2.RELEASE</spring.version>
	<hibernate.version>4.1.6.Final</hibernate.version>
	<jstl.version>1.2</jstl.version><!--用于jsp页面jstl表达式  -->
	<jackson.version>1.9.9</jackson.version><!--java处理json格式数据  -->
	<slf4j.version>1.6.4</slf4j.version><!--日志相关相当于接口  -->
	<junit.version>4.10</junit.version><!-- 单元测试 -->
	<servlet.version>3.1.0</servlet.version><!--servlet-api jsp-api-->
	<cglib.version>2.2.2</cglib.version><!--动态代理 (spring aop  hibernate)-->
	<dbcp.version>1.4</dbcp.version><!--数据库连接池  -->
	<aspectjweaver.version>1.6.9</aspectjweaver.version><!--这是Spring AOP所要用到的包  -->
	<commons-logging.version>1.1.1</commons-logging.version><!--日志相关相当于接口  -->
	<commons-lang3.version>3.1</commons-lang3.version><!--通用api(日期，字符串等)工具包  -->
	<commons-beanutils.version>1.8.3</commons-beanutils.version><!--操作javabean相关 与发射有关  -->
	<commons-collections.version>3.2.1</commons-collections.version><!--java集合相关的工具包  -->
	<commons-io.version>2.3</commons-io.version><!--java io 相关工具包  -->
	<commons-fileupload.version>1.2.2</commons-fileupload.version><!--文件上传相关工具包  -->
	<ehcache.version>2.6.6</ehcache.version><!--java进程内缓存框架 hibernate默认cache  -->
	<opensymphony.version>2.4.1</opensymphony.version><!--sitemesh相关 -->
	<shiro.version>1.2.1</shiro.version><!--shiro验证相关  -->
	<quartz.version>1.8.6</quartz.version><!--定时任务相关  -->
	<commons-codec.version>1.6</commons-codec.version><!--编码方法的工具类包,例如DES、SHA1、MD5、Base64 -->
  </properties>
  
  <dependencies>
 		 <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
			<exclusions>
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
			<scope>test</scope>
		</dependency>
		
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>${hibernate.version}</version>
		</dependency>

		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-ehcache</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
		
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>${jstl.version}</version>
		</dependency>
		
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>${servlet.version}</version>
			<scope>provided</scope>
		</dependency>

		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.2.1-b03</version>
			<scope>provided</scope>
		</dependency>
		
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-core-asl</artifactId>
			<version>${jackson.version}</version>
		</dependency>

		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		
	    <dependency>
	      <groupId>junit</groupId>
	      <artifactId>junit</artifactId>
	      <version>${junit.version}</version>
	      <scope>test</scope>
	    </dependency>
	    
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jcl-over-slf4j</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
    
   		<dependency>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
			<version>${cglib.version}</version>
		</dependency>
		
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>${dbcp.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjweaver</artifactId>
			<version>${aspectjweaver.version}</version>
		</dependency>
		
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>${commons-logging.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>${commons-lang3.version}</version>
		</dependency>
		
		<dependency>
			<groupId>commons-beanutils</groupId>
			<artifactId>commons-beanutils</artifactId>
			<version>${commons-beanutils.version}</version>
			<exclusions>
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<dependency>
			<groupId>commons-collections</groupId>
			<artifactId>commons-collections</artifactId>
			<version>${commons-collections.version}</version>
		</dependency>
		
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>${commons-io.version}</version>
		</dependency>
		
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>${commons-fileupload.version}</version>
		</dependency>
		
		<dependency>
			<groupId>net.sf.ehcache</groupId>
			<artifactId>ehcache-core</artifactId>
			<version>${ehcache.version}</version>
		</dependency>
		
		<dependency>
			<groupId>opensymphony</groupId>
			<artifactId>sitemesh</artifactId>
			<version>${opensymphony.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-web</artifactId>
			<version>${shiro.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-ehcache</artifactId>
			<version>${shiro.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring</artifactId>
			<version>${shiro.version}</version>
		</dependency>
		
	<dependency>
	    <groupId>org.quartz-scheduler</groupId>
	    <artifactId>quartz</artifactId>
	    <version>${quartz.version}</version>
        </dependency>
        
        <dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>${mysql.version}</version>
	</dependency>
		
	<dependency>
		<groupId>commons-codec</groupId>
		<artifactId>commons-codec</artifactId>
		<version>${commons-codec.version}</version>
	</dependency>
		
  </dependencies>
  
  <build>
    <finalName>examples</finalName>
    <resources>
	<resource>
		<directory>src/main/resources/</directory>
		<filtering>true</filtering>
	</resource>
    </resources>
		
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-surefire-plugin</artifactId>
			<version>2.9</version>
			<configuration>
				<argLine>-Xmx256M</argLine>
				<skip>${maven.test.skip}</skip>
				<testFailureIgnore>${maven.test.skip}</testFailureIgnore>
			</configuration>
		</plugin>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>2.3.2</version>
			<configuration>
				<source>${jdk.version}</source>
				<target>${jdk.version}</target>
			</configuration>
		</plugin>
	</plugins>
  </build>
  <profiles>
	<profile>
		<id>dev</id>
		<build>
			<filters>
				<filter>profile.dev.properties</filter>
			</filters>
		</build>
		<activation>
			<activeByDefault>true</activeByDefault><!--默认启用的是dev环境配置 -->
		</activation>
	</profile>
	<profile>
		<id>test</id>
		<build>
			<filters>
				<filter>profile.test.properties</filter>
			</filters>
		</build>
	</profile>
	<profile>
		<id>production</id>
		<build>
			<filters>
				<filter>profile.production.properties</filter>
			</filters>
		</build>
	</profile>
 </profiles>
</project>
```

需要在pom.xml同级目录下创建profile.dev.properties，profile.test.properties，profile.production.properties三个文件，其中内容如下，  
以profile.dev.properties为例：  

```java
log.level=DEBUG
log.file=F:/dev/examples.log

jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/examplesdev
jdbc.username=root
jdbc.password=root
```

### 总结  

以上就是对整个项目的配置文件的分析，maven打包的时候加上不同的参数就会连接不同的数据库，以及在不同位置产生日志文件。如：  

```bash
mvn clean package -P production
mvn clean package -P dev
mvn clean package -P test
```


