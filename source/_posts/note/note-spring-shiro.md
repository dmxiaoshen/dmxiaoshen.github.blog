title: Spring与Shiro整合实现权限控制
date: 2014-12-07 15:17:11
categories: note
tags: [spring, shiro]
---

## Spring与Shiro整合实现权限控制

如果你的项目需要权限管理，不妨试试Apache Shiro.  

### 什么是Apache Shiro?  

> Apache Shiro(发音为“shee-roh”，日语“堡垒(Castle)”的意思)是一个强大易用的Java安全框架，提供了认证、授权、加密和会话管理功能，可为任何应用提供安全保障 - 从命令行应用、移动应用到大型网络及企业应用。Shiro为解决下列问题(我喜欢称它们为应用安全的四要素)提供了保护应用的API：

> * 认证 - 用户身份识别，常被称为用户“登录”；  
* 授权 - 访问控制；  
* 密码加密 - 保护或隐藏数据防止被偷窥；  
* 会话管理 - 每用户相关的时间敏感的状态。

> Shiro还支持一些辅助特性，如Web应用安全、单元测试和多线程，它们的存在强化了上面提到的四个要素。
核心概念：Subject，SecurityManager和Realms

### shiro的jar包  

如果你的项目是maven管理的，需要配置如下:  

```xml
<properties>
	<shiro.version>1.2.1</shiro.version>
</properties>
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
```

### web.xml  

配置shiro的过滤器:  

```xml
<!-- shiro 过滤器 -->
<filter>
	<filter-name>shiroFilter</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	<init-param>
		<param-name>targetFilterLifecycle</param-name>
		<param-value>true</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>shiroFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

### spring-shiro.xml  

shiro的主配置文件:  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://www.springframework.org/schema/beans" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" 
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation=" http://www.springframework.org/schema/beans 
			http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
			http://www.springframework.org/schema/context 
			http://www.springframework.org/schema/context/spring-context-3.1.xsd 
			http://www.springframework.org/schema/mvc
			http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd 
			http://www.springframework.org/schema/aop
			http://www.springframework.org/schema/aop/spring-aop-3.1.xsd 
			http://www.springframework.org/schema/tx
			http://www.springframework.org/schema/tx/spring-tx-3.1.xsd ">

	<description>Shiro Configuration</description>

	<!-- Shiro's main business-tier object for web-enabled applications -->
	<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
		<property name="realm" ref="myRealm" />
		<property name="cacheManager" ref="cacheManager" />
	</bean>

	<!-- 用户授权信息Cache -->
	<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
		 <property name="cacheManagerConfigFile" value="classpath:cache/ehcache-shiro-local.xml" 
			/> 
	</bean>

	<!-- 项目自定义的Realm -->
	<bean id="myRealm" class="com.hsg.plms.shiro.entity.MyRealm">
		<property name="credentialsMatcher" ref="passwordMatcher" />
		<property name="cacheManager" ref="cacheManager" />
	</bean>

	<bean id="passwordMatcher"
		class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
		<property name="hashAlgorithmName" value="SHA-256" /><!-- MD5 
			SHA-256 SHA-1 -->
		<property name="storedCredentialsHexEncoded" value="false" />
			<!-- 是否16,默认true,此处用base64 -->
		<property name="hashIterations" value="1"></property><!-- 迭代次数,默认1 -->
	</bean>

	<!-- Shiro Filter -->
	<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
		<property name="securityManager" ref="securityManager" />
 		<property name="loginUrl" value="/login/index" /> 
		<property name="successUrl" value="/" />
		<property name="unauthorizedUrl" value="/login/index" />
		 <property name="filters">
			<map>
				<entry key="my_authc">
					<bean class="com.hsg.plms.shiro.filter.MyAuthenticatingFilter" />
				</entry>
			</map>
		</property>
		<property name="filterChainDefinitions">
			<value>			
				/login/logout = logout
				/static/** = anon	
				/login/getCaptcha = anon
				/login/checkCaptcha = anon							
				/** = my_authc
			</value>
		</property>
	</bean>

	<!-- 保证实现了Shiro内部lifecycle函数的bean执行 -->
	<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />

</beans>
```

#### 自定义的Realm  

```java
public class MyRealm extends AuthorizingRealm{
    
    @Autowired
    private UserService userService;

    /** 授权回调*/
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //ShiroUser u = (ShiroUser)principals.getPrimaryPrincipal();
        //String currentUsername = (String)super.getAvailablePrincipal(principals);
        ShiroUser shiroUser = (ShiroUser) principals.fromRealm(getName()).iterator().next(); // 只有一个桥梁
        User user = userService.findUserByName(shiroUser.getUsername());
        if (user != null) {
            SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();           
            info.addObjectPermission(new AllPermission());          
            return info;
        }
        return null;
    }

    /** 登录回调*/
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String name = token.getPrincipal().toString();
        User user = userService.findUserByName(name);
        if(user!=null){
            Object principal = getPrincipal(user);
            SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(principal, user.getPassword(),new SimpleByteSource(user.getUsername()), getName());
            return info;
        }
        return null;
    }
    
    private Object getPrincipal(User user){
        ShiroUser principal = new ShiroUser();
        principal.setUserId(user.getId());
        principal.setUsername(user.getUsername());
        return principal;
    }

}
```

**ShiroUser** 

```java
public class ShiroUser implements Serializable {

    /** */
    private static final long serialVersionUID = -8167341999489366817L;

    private Long userId;
    private String username;

    public Long getUserId() {
        return userId;
    }

    public void setUserId(Long userId) {
        this.userId = userId;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

}
```

#### 自定义的fliter  

```java
public class MyAuthenticatingFilter extends FormAuthenticationFilter{
    
    @Override
    protected void setFailureAttribute(ServletRequest request, AuthenticationException ae) {
	/** 放入异常的完整信息，控制层controller需要用到*/
        request.setAttribute(getFailureKeyAttribute(), ae);
    }

}

```

### spring-mvc.xml  

为了开启Shiro权限注解，需要加入:  

```xml
<!-- shiro相关  @RequiresRoles，@RequiresPermissions 等等使用-->
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
	<property name="securityManager" ref="securityManager" />	
</bean>

<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" 
	depends-on="lifecycleBeanPostProcessor" /> 
```

如：`@RequiresRoles` 以及 `@RequiresPermissions`.  


### login.jsp  

```javascript
<form id="form" action="login" method="post">
<p>
<label>用户名:</label><input type="text" name="username" />
</p>
<p>
<label>密码:</label><input type="password" name="password" />
</p>
<p><input type="submit" value="登录" /></p>
</form>
```

#### LoginController  

```java
 public String login(HttpServletRequest request,Model model,HttpSession session){
	//因为自定义过滤器里面放入了完整异常信息，所以这里取得的不是string而是异常类
        Object obj = request.getAttribute(FormAuthenticationFilter.DEFAULT_ERROR_KEY_ATTRIBUTE_NAME);
        AuthenticationException authExp = (AuthenticationException) obj;
        if (authExp != null) {
            if(authExp instanceof UnknownAccountException || authExp instanceof IncorrectCredentialsException){
                model.addAttribute("error", "* 用户名或密码错误");
            }        
            return loginPage(model);
        }
                
        return "redirect:/";
    }
```

### 总结  

到此，基本的配置已经完成了，这里要注意的是，用户密码是通过sha256加密的，而且通过加盐（用户名）处理。  
生成的时候可以这样调用:  

```java
new Sha256Hash(password, username).toBase64()
```

遗留问题: 验证码问题，这里配置只负责了用户名，密码，没有涉及到验证码，下一篇补上。  


### 补充  

**Shiro自带过滤器**

```xml
默认过滤器(10个) 
anon -- org.apache.shiro.web.filter.authc.AnonymousFilter
authc -- org.apache.shiro.web.filter.authc.FormAuthenticationFilter
authcBasic -- org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter
perms -- org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter
port -- org.apache.shiro.web.filter.authz.PortFilter
rest -- org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter
roles -- org.apache.shiro.web.filter.authz.RolesAuthorizationFilter
ssl -- org.apache.shiro.web.filter.authz.SslFilter
user -- org.apache.shiro.web.filter.authc.UserFilter
logout -- org.apache.shiro.web.filter.authc.LogoutFilter


anon:例子/admins/**=anon 没有参数，表示可以匿名使用。 
authc:例如/admins/user/**=authc表示需要认证(登录)才能使用，没有参数 
roles：例子/admins/user/**=roles[admin],参数可以写多个，多个时必须加上引号，并且参数之间用逗号分割，当有多个参数时，例如admins/user/**=roles["admin,guest"],每个参数通过才算通过，相当于hasAllRoles()方法。 
perms：例子/admins/user/**=perms[user:add:*],参数可以写多个，多个时必须加上引号，并且参数之间用逗号分割，例如/admins/user/**=perms["user:add:*,user:modify:*"]，当有多个参数时必须每个参数都通过才通过，想当于isPermitedAll()方法。 
rest：例子/admins/user/**=rest[user],根据请求的方法，相当于/admins/user/**=perms[user:method] ,其中method为post，get，delete等。 
port：例子/admins/user/**=port[8081],当请求的url的端口不是8081是跳转到schemal://serverName:8081?queryString,其中schmal是协议http或https等，serverName是你访问的host,8081是url配置里port的端口，queryString是你访问的url里的？后面的参数。 
authcBasic：例如/admins/user/**=authcBasic没有参数表示httpBasic认证 
ssl:例子/admins/user/**=ssl没有参数，表示安全的url请求，协议为https 
user:例如/admins/user/**=user没有参数表示必须存在用户，当登入操作时不做检查 
```

**shiro页面标签**

```xml
<shiro:authenticated> 登录之后
<shiro:notAuthenticated> 不在登录状态时
<shiro:guest> 用户在没有RememberMe时
<shiro:user> 用户在RememberMe时
<shiro:hasAnyRoles name="abc,123" > 在有abc或者123角色时
<shiro:hasRole name="abc"> 拥有角色abc
<shiro:lacksRole name="abc"> 没有角色abc
<shiro:hasPermission name="abc"> 拥有权限abc
<shiro:lacksPermission name="abc"> 没有权限abc
<shiro:principal> 显示用户登录名
```


参考网址: <http://blog.csdn.net/he90227/article/details/38663553>