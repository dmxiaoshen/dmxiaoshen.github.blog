title: SpringMVC的restful支持
date: 2015-09-01 14:17
categories: note
tags: [Spring, restful]
---

## SpringMVC的restful支持

springmvc可以实现对同一资源的不同展示，在同一个方法里。


需要用到:
```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>${spring.version}</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-expression</artifactId>
	<version>${spring.version}</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-beans</artifactId>
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
	<artifactId>spring-oxm</artifactId>
	<version>${spring.version}</version>
</dependency>

<dependency>
	<groupId>org.codehaus.jackson</groupId>
	<artifactId>jackson-mapper-asl</artifactId>
	<version>1.9.3</version>
</dependency>
```

关键是spring-mvc.xml的配置:
```xml
<bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
		<property name="order" value="1" />	
		 <property name="contentNegotiationManager">  
            <bean class="org.springframework.web.accept.ContentNegotiationManager">  
                <constructor-arg>  
                    <bean class="org.springframework.web.accept.PathExtensionContentNegotiationStrategy">  
                        <constructor-arg>  
                            <map>  
                                <entry key="json" value="application/json"/>  
                                <entry key="xml" value="application/xml"/>  
                            </map>  
                        </constructor-arg>  
                    </bean>  
                </constructor-arg>  
            </bean>  
        </property>  
		<property name="viewResolvers">
			<list>
				<bean class="org.springframework.web.servlet.view.BeanNameViewResolver" />
				<bean
					class="org.springframework.web.servlet.view.InternalResourceViewResolver">
					<property name="prefix" value="/WEB-INF/views/" />
					<property name="suffix" value=".jsp" />
				</bean>
			</list>
		</property>
		<property name="defaultViews">
			<list>
				<bean id="jsonView"
					class="org.springframework.web.servlet.view.json.MappingJacksonJsonView" />
				<bean id="xmlView"
					class="org.springframework.web.servlet.view.xml.MarshallingView">
					<constructor-arg>
						<bean class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
							<property name="classesToBeBound">
								<list>
									<value>com.cnblogs.yjmyzz.dto.UserInfo</value>
									<value>com.cnblogs.yjmyzz.dto.ListBean</value>
								</list>
							</property>
						</bean>
					</constructor-arg>
				</bean>
			</list>
		</property>
	</bean>
```

controller层可以这么写:
```java
  @RequestMapping(value = {"/user/{id}"}, method = RequestMethod.GET)
  public UserInfo show(@PathVariable int id, HttpServletRequest request,
		HttpServletResponse response) throws Exception {
	return userService.getUserInfo(id);

  }

  @RequestMapping(value = "/user/list", method = RequestMethod.GET)
  public ListBean getAll() throws Exception {
	return userService.getAllUsers();

  }

  @RequestMapping(value = {"/uu/{id}"}, method = RequestMethod.GET)
  public String uuids(@PathVariable int id,ModelMap model, HttpServletRequest request,
		HttpServletResponse response) throws Exception {
	UserInfo u = userService.getUserInfo(id);
	model.put("user", u);
	model.put("xyz", "o ha yo u");
	return "/index";
  }
```

实际达到的效果是，访问/uu/1 返回相应的视图页面 访问/uu/1.json 返回json格式数据。