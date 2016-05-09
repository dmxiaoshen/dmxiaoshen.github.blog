title: Spring与ActiveMQ的整合
date: 2015-01-13 13:32:18
categories: note
tags: [activemq]
---

## Spring与ActiveMQ的整合

### ActiveMQ简介  

> 企业消息软件从80年代起就存在，它不只是一种应用间消息传递风格，也是一种集成风格。因此，消息传递可以满足应用间的通知和互相操作。但是开源的解决方案是到最近10年才出现的。Apache ActiveMQ就是其中一种。它使应用间能以异步，松耦合方式交流。  
> ActiveMQ是Apache软件基金下的一个开源软件，它遵循JMS1.1规范（Java Message Service），是消息驱动中间件软件（MOM）。它为企业消息传递提供高可用，出色性能，可扩展，稳定和安全保障。ActiveMQ使用Apache许可协议。  
> 因此，任何人都可以使用和修改它而不必反馈任何改变。这对于商业上将ActiveMQ用在重要用途的人尤为关键。MOM的工作是在分布式的各应用之间调度事件和消息。所以高可用，高性能，高可扩展性尤为关键。  
> ActiveMQ的目标是在尽可能多的平台和语言上提供一个标准的，消息驱动的应用集成。ActiveMQ实现JMS规范并在此之上提供大量额外的特性。

### 下载及配置  

到官网下载；[ActiveMQ](http://activemq.apache.org/)  
下载完成解压zip文件，进入bin目录启动activemq.bat便可以启动服务，类似于tomcat。  

如果是maven项目:

```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jms</artifactId>  
    <version>3.1.2.RELEASE</version>  
</dependency>   
<dependency>  
    <groupId>org.apache.activemq</groupId>  
    <artifactId>activemq-core</artifactId>  
    <version>5.10.0</version>  
</dependency>  
```

如果不是maven项目，那就要把所有依赖到的jar包放入lib中，具体哪些jar包可以到网上查找，上面解压的zip文件中的lib目录应该会有所需的jar包。  

### Spring的配置以及相关类的创建  

```xml
<!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
<bean id="connectinFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
	<property name="brokerURL" value="tcp://127.0.0.1:61616" />
</bean>

<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
<bean id="cachingConnectionFactory"
	class="org.springframework.jms.connection.CachingConnectionFactory">
	<!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
	<property name="targetConnectionFactory" ref="connectinFactory"></property>
	<!-- Session缓存数量 -->
	<property name="sessionCacheSize" value="30"></property>
</bean>

<!-- Spring JMS Template 配置JMS模版 -->
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
	<property name="connectionFactory" ref="cachingConnectionFactory" />
</bean>

<!-- 配置消息发送目的地方式 -->
<!-- Queue队列：仅有一个订阅者会收到消息，消息一旦被处理就不会存在队列中 -->
<bean id="notifyQueue" class="org.apache.activemq.command.ActiveMQQueue">
	<constructor-arg value="q.notify"></constructor-arg>
</bean>

<!--消息转换类-->
<bean id="messageConverter" class="com.dmxiaoshen.activemq.converter.NotifyMessageConverter"></bean>

<!-- 使用Spring JmsTemplate 的消息生产者 -->
<bean id="messageProducer" class="com.dmxiaoshen.activemq.producer.MessageProducer">
	<property name="jmsTemplate" ref="jmsTemplate"></property>
	<property name="notifyQueue" ref="notifyQueue"></property>
	<property name="messageConverter" ref="messageConverter"></property>
</bean>

<!-- 异步接收消息处理类 及消费者-->
<bean id="messageListener" class="com.dmxiaoshen.activemq.listener.MessageListener">
	<property name="messageConverter" ref="messageConverter"></property>
</bean>

<!-- 消息消费者 一般使用spring的MDP异步接收Queue模式 -->
<!-- 消息监听容器 -->
<!--说的通俗点就是把生产者，消费者，连接，消息目的地组装起来-->
<bean id="hqQueueContainer"
	class="org.springframework.jms.listener.DefaultMessageListenerContainer">
	<property name="connectionFactory" ref="cachingConnectionFactory"></property>
	<property name="destination" ref="notifyQueue"></property>
	<property name="messageListener" ref="messageListener"></property>
	<property name="concurrency" value="30"/><!-- 这个属性很关键，最大并发数30，也就是启动30个实例来消费 -->
</bean>
```

Spring的配置到这里结束了，可以看出其中有3个类是需要自己建的，**NotifyMessageConverter**，**MessageProducer**，**MessageListener**。  

额外多一个**PhoneNoticeInfo**类，实现序列化接口，用以传递信息。

**PhoneNoticeInfo**
```java
public class PhoneNoticeInfo implements Serializable {
 /** 消息标题 */
 public String noticeTitle;
 /** 消息内容 */
 public String noticeContent;
 /** 接收者 */
 public String receiver;
 /** 接收手机号 */
 public String receiverPhone;
 public String getNoticeTitle() {
  return noticeTitle;
 }
 public void setNoticeTitle(String noticeTitle) {
  this.noticeTitle = noticeTitle;
 }
 public String getNoticeContent() {
  return noticeContent;
 }
 public void setNoticeContent(String noticeContent) {
  this.noticeContent = noticeContent;
 }
 public String getReceiver() {
  return receiver;
 }
 public void setReceiver(String receiver) {
  this.receiver = receiver;
 }

 public String getReceiverPhone() {
  return receiverPhone;
 }
 public void setReceiverPhone(String receiverPhone) {
  this.receiverPhone = receiverPhone;
 }
 
}
```

以下给出3个类的实现:

**NotifyMessageConverter**

```java
/**
 * 消息转换
 */
public class NotifyMessageConverter implements MessageConverter {
 private static Logger logger = LoggerFactory.getLogger(NotifyMessageConverter.class);
 @Override
 /**
  * 转换接收到的消息为NoticeInfo对象
  */
 public Object fromMessage(Message message) throws JMSException,
   MessageConversionException {
  if (logger.isDebugEnabled()) {
   logger.debug("Receive JMS message :"+message);
  }
  if (message instanceof ObjectMessage) {
   ObjectMessage oMsg = (ObjectMessage)message;
   if (oMsg instanceof ActiveMQObjectMessage) {
    ActiveMQObjectMessage aMsg = (ActiveMQObjectMessage)oMsg;
    try {
     PhoneNoticeInfo noticeInfo = (PhoneNoticeInfo)aMsg.getObject();
     return noticeInfo;
    } catch (Exception e) {
     logger.error("Message:${} is not a instance of NoticeInfo."+message.toString());
     throw new JMSException("Message:"+message.toString()+"is not a instance of NoticeInfo."+message.toString());
    }
   }else{
    logger.error("Message:${} is not a instance of ActiveMQObjectMessage."+message.toString());
    throw new JMSException("Message:"+message.toString()+"is not a instance of ActiveMQObjectMessage."+message.toString());
   }
  }else {
   logger.error("Message:${} is not a instance of ObjectMessage."+message.toString());
   throw new JMSException("Message:"+message.toString()+"is not a instance of ObjectMessage."+message.toString());
  }
 }

 @Override
 /**
  * 转换NoticeInfo对象到消息
  */
 public Message toMessage(Object obj, Session session) throws JMSException,
   MessageConversionException {
  if (logger.isDebugEnabled()) {
   logger.debug("Convert Notify object to JMS message:${}"+obj.toString());
  }
  if (obj instanceof PhoneNoticeInfo) {
   ActiveMQObjectMessage msg = (ActiveMQObjectMessage)session.createObjectMessage();
   msg.setObject((PhoneNoticeInfo)obj);
   return msg;
  }else {
   logger.debug("Convert Notify object to JMS message:${}"+obj.toString());
  }
  return null;
 }

}
```

**MessageProducer**  

```java
/**
 * 消息生产者服务类
 */
public class MessageProducer {
 private JmsTemplate jmsTemplate;
 private Destination notifyQueue;
 private NotifyMessageConverter messageConverter;

 public void sendQueue(PhoneNoticeInfo noticeInfo){
  sendMessage(noticeInfo);
 }
 private void sendMessage(PhoneNoticeInfo noticeInfo) {
  jmsTemplate.setMessageConverter(messageConverter);
  jmsTemplate.setPubSubDomain(false);
  jmsTemplate.convertAndSend(notifyQueue,noticeInfo);
 }
 public JmsTemplate getJmsTemplate() {
  return jmsTemplate;
 }
 public void setJmsTemplate(JmsTemplate jmsTemplate) {
  this.jmsTemplate = jmsTemplate;
 }
 public Destination getNotifyQueue() {
  return notifyQueue;
 }
 public void setNotifyQueue(Destination notifyQueue) {
  this.notifyQueue = notifyQueue;
 }
 public NotifyMessageConverter getMessageConverter() {
  return messageConverter;
 }
 public void setMessageConverter(NotifyMessageConverter messageConverter) {
  this.messageConverter = messageConverter;
 }
}
```

**MessageListener**  

```java
public class MessageListener implements MessageListener {
 private static Logger logger = LoggerFactory.getLogger(QueueMessageListener.class);
 private NotifyMessageConverter messageConverter;
 
 /**
  * 接收消息
  */
 @Override
 public void onMessage(Message message) {
  try {
   ObjectMessage objectMessage = (ObjectMessage)message;
   PhoneNoticeInfo noticeInfo = (PhoneNoticeInfo)messageConverter.fromMessage(objectMessage);
   System.out.println("queue收到消息"+noticeInfo.getNoticeContent());
   System.out.println("model:"+objectMessage.getJMSDeliveryMode()); 
   System.out.println("destination:"+objectMessage.getJMSDestination()); 
   System.out.println("type:"+objectMessage.getJMSType()); 
   System.out.println("messageId:"+objectMessage.getJMSMessageID()); 
   System.out.println("time:"+objectMessage.getJMSTimestamp()); 
   System.out.println("expiredTime:"+objectMessage.getJMSExpiration()); 
   System.out.println("priority:"+objectMessage.getJMSPriority()); 

  } catch (Exception e) {
   logger.error("处理信息时发生异常",e);
  }
 }
 public NotifyMessageConverter getMessageConverter() {
  return messageConverter;
 }
 public void setMessageConverter(NotifyMessageConverter messageConverter) {
  this.messageConverter = messageConverter;
 }

}
```

上述就是所需的类，只需调用MessageProducer实例的sendQueue方法，就可以发送消息到队列了。  
需要在此处`<property name="brokerURL" value="tcp://127.0.0.1:61616" />`配置的服务器上启动activemq服务。  
浏览器进入[ActiveMQ管理页面](http://127.0.0.1:8161/admin/queues.jsp)即可查看队列的信息。   
用户名 admin 密码 admin