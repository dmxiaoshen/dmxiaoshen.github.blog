title: Spring结合JUnit4进行单元测试
date: 2014-12-21 13:17:11
categories: note
tags: [JUnit]
---

## Spring结合JUnit4进行单元测试

> 在任何工程中执行单元测试都十分重要。它们可以帮助确定局部代码缺陷。代码单元的所有基本的对象功能和特性应该(通过不同的测试用例)在单元测试中得到处理。在许多较大的工程中，小组领导人可能会认为不包含单元测试的代码是不完整的。  
> 编写单元测试与其说是一种科学，不如说是一种艺术。单元测试的目的是保证代码所执行的是期望它执行的工作，以及确定潜在的代码缺陷。编写单元测试的一个方法是首先确定一种可能存在的缺陷，然后编写代码以对其进行测试。一些开发人员甚至坚持认为应该在编写代码以前先编写单元测试。  

### BaseTest  

首先，创建一个测试类的基类  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"classpath*:spring-common.xml","classpath*:spring-shiro.xml"})
@TransactionConfiguration(transactionManager="transactionManager",defaultRollback=true)
public class BaseTest extends AbstractTransactionalJUnit4SpringContextTests{

}
```

### @RunWith

用来指定测试类的运行器，默认是JUnit4.class，这里我们指定SpringJUnit4ClassRunner.class因为需要用到Spring的相关功能，如依赖注入。

### @ContextConfigu  

用来指定配置文件的地址，因为我们既然使用了Spring，就必须引入相关的配置文件。(貌似Spring-Test的这个配置只认classpath，很无奈，我们必须拷贝这些文件到根目录！)

### @TransactionConfiguration  

这个注解是用来配置事务的，如果我们要测试一些方法，如增、删、改操作，对数据库的操作，这时需要继承一个类AbstractTransactionalJUnit4SpringContextTests，  
**transactionManager**,这里的事物管理器关联到配置文件里的事物管理器,**defaultRollback**,默认事物是否回滚。  

到此，一个测试基类算是完成。  

### @Test

那么如何编写测试类呢？  

```java
public class UserServiceTest extends BaseTest{
    @Autowired
    private UserService userService;
    
   
    @Test
    @Rollback(value=false)
    public void testInsert(){
        //插入user
    }
    
    @Test
    @Rollback(value=false)
    public void testDel(){
        //删除user
    }

}
```

一般情况下，只要继承测试基类，然后方法标注@Test就能对方法进行测试，这里用到的@Autowired是Spring的注入功能，@Rollback设置事物不回滚，因为在测试基类中，统一  
配置了事物回滚。  

关于测试类，其实还有一些注解：  

**@Ignore**  

有时候我们想暂时不运行某些测试方法\测试类，可以在方法前加上这个注解。在运行结果中，junit会统计忽略的用例数，来提醒你。但是不建议经常这么做，因为这样的坏处时，容易忘记去更新这些测试方法，导致代码不够干净，用例遗漏。  

**@BeforeClass**  

当我们运行几个有关联的用例时，可能会在数据准备或其它前期准备中执行一些相同的命令，这个时候为了让代码更清晰，更少冗余，可以将公用的部分提取出来，放在一个方法里，并为这个方法注解@BeforeClass。意思是在测试类里所有用例运行之前，运行一次这个方法。例如创建数据库连接、读取文件等。  
注意：方法名可以任意，但必须是public static void，即公开、静态、无返回。这个方法只会运行一次。  

**@AfterClass**  

跟@BeforeClass对应，在测试类里所有用例运行之后，运行一次。用于处理一些测试后续工作，例如清理数据，恢复现场。   
注意：同样必须是public static void，即公开、静态、无返回。这个方法只会运行一次。  

**@Before**  

与@BeforeClass的区别在于，@Before不止运行一次，它会在每个用例运行之前都运行一次。主要用于一些独立于用例之间的准备工作。比如两个用例都需要读取数据库里的用户A信息，但第一个用例会删除这个用户A，而第二个用例需要修改用户A。那么可以用@BeforeClass创建数据库连接。用@Before来插入一条用户A信息。   
注意：必须是public void，不能为static。不止运行一次，根据用例数而定。  

**@After**

与@Before对应。  

**顺序:** @BeforeClass->@Before->@Test->@After->@AfterClass