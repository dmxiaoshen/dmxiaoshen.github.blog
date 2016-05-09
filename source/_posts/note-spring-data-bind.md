title: Spring MVC 数据绑定问题
date: 2014-12-01 10:17:11
categories: note
tags: [spring, mvc]
---

## Spring MVC 数据绑定问题

### 问题来了  

**我们在用spring mvc时，如果需要传递日期参数，spring mvc默认是不支持string直接转date的，要报错，咋办？**

### 方法一

使用@InitBinder。在所需要的controller类中加入如下代码:  

```java
@InitBinder
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(
                dateFormat, false));
    }
```

### 方法二  

继承WebBindingInitializer接口实现全局注册。写一个类：  

```java
public class CustomerBinding implements WebBindingInitializer {
    @Override
    public void initBinder(WebDataBinder binder, WebRequest request) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(
                dateFormat, false));

    }
```

修改spring-mvc.xml配置文件:  

```xml
  <bean
        class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
        <property name="webBindingInitializer">
            <bean
                class="net.zhepu.web.customerBinding.CustomerBinding" />
        </property>
  </bean>
```

**但是美中不足的是，这样一来，因为手动去注册了AnnotationMethodHandlerAdapter类，导致无法使用`<mvc:annotation-driven/>`**。

### 方法三  

使用自定义转换器。配置文件如下:  

```xml

<mvc:annotation-driven conversion-service="conversionService" />

<bean id="conversionService"
        class="org.springframework.format.support.FormattingConversionServiceFactoryBean">

        <property name="converters">
            <list>
                <bean class="com.dmxiaoshen.converter.CustomerConverter" />
            </list>
        </property>
        
</bean>
```

CustomerConverter类如下:

```java
public class CustomerConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        try {
            return dateFormat.parse(source);
        } catch (ParseException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }        
        return null;
    }
```

### 对于requestBody或httpEntity中数据的类型转换  

Spring MVC中对于requestBody中发送的数据转换不是通过databind来实现，而是使用HttpMessageConverter来实现具体的类型转换。  
例如，之前提到的json格式的输入，在将json格式的输入转换为具体的model的过程中，spring mvc首先找出request header中的contenttype，  
再遍历当前所注册的所有的HttpMessageConverter子类， 根据子类中的canRead()方法来决定调用哪个具体的子类来实现对requestBody中的数据的解析。  
如果当前所注册的httpMessageConverter中都无法解析对应contexttype类型，则抛出HttpMediaTypeNotSupportedException （http 415错误）。   
那么需要如何注册自定义的messageConverter呢，很不幸，在spring 3.0.5中如果使用annotation-driven的配置方式的话，  
无法实现自定义的messageConverter的配置，必须老老实实的自己定义AnnotationMethodHandlerAdapter的bean定义，  
再设置其messageConverters以注册自定义的messageConverter。 在3.1版本中，将增加annotation-driven对自定义的messageConverter的支持 (SPR-7504)，具体格式如下

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter"/>
        <bean class="org.springframework.http.converter.ResourceHttpMessageConverter"/>
        <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"/>
    </mvc:message-converters>
</mvc:annotation-driven>
```


参考网址: <http://www.cnblogs.com/crazy-fox/archive/2012/02/18/2357699.html>

