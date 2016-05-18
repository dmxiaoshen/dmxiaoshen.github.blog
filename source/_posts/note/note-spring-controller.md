title: SpringMVC控制层Controller那点事
date: 2014-12-10 10:17:25
categories: note
tags: [springMVC]
---

## SpringMVC控制层Controller那点事

### @Controller


srpingMVC中实现一个controller很简单，只需使用@Controller标记即可：  

```java
@Controller
public class LoginController{

}
```


### @RequesetMapping

如何生成相应的url映射呢？使用@RequestMapping注解，可以在类上标注也可以在方法上标注，如果都标注了，访问的URL即为类上的加上方法上的。  

```java
@Controller
@RequestMapping(value="/login")
public class LoginController{

   @RequestMapping(value="/index")
   public String loginPage(){
     return "/login/index";
   }
}
```

登录url为: xxx/login/index  

### 数据绑定问题(如@RequestParam，@PathVariable)  

解决了url链接的映射问题，下面数据绑定的问题就来了，光有链接不行，很多时候需要传递数据，springMVC传递数据有多种方式。  

1. **@RequestParam绑定单个请求参数值**
2. **@PathVariable绑定URI模板变量值**
3. **@CookieValue绑定Cookie数据值**
4. **@RequestHeader绑定请求头数据**
5. **@ModelAttribute绑定参数到命令对象**
6. **@SessionAttributes绑定命令对象到session**
7. **@RequestBody绑定请求的内容区数据并能进行自动类型转换等**
8. **@RequestPart绑定"multipart/data"数据,除了能绑定@RequestParam能做到的请求参数外，还能绑定上传的文件等**

除了上边提到的注解，还可以通过如HttpServletRequest等API得到请求数据，但推荐使用注解方式，因为使用起来更简单

以下是列子:  

```java
@Controller
@RequestMapping(value="/test")
@SessionAttributes(value="userTemp")
public class TestController {
    
    @RequestMapping(value="/search")
    public String requestParam(@RequestParam("username") String username){
        return "/test";
    }
    
//    @RequestMapping(value="/search")
//    public String requestParam2(@RequestParam(value="username",required=true,defaultValue="dmxiaoshen") String username){
//        return "/test";
//    }
    
    @RequestMapping(value="/search/{username}")
    public String pathVariable(@PathVariable("username") String username){
        return "/test";
    }
    
    @RequestMapping(value="/cookietest")
    public String cookieValue(@CookieValue(value="JSESSIONID", defaultValue="") String sessionId){
        return "/test";
    }
    
    @RequestMapping(value="/requetheader")
    public String requestHeaderTest( @RequestHeader("User-Agent") String userAgent,
            @RequestHeader(value="Accept") String[] accepts){
        return "/test";
    }
    
    @RequestMapping(value="/addUserFirst")
    public String addUserFirst(@ModelAttribute("userTemp") User userTemp){        
        return "redirect:/test/addUserSecond";
    }
    
    @RequestMapping(value="/addUserSecond")
    public String addUserSecond(@ModelAttribute("userTemp") User userTemp,SessionStatus sessionStatus){    
        sessionStatus.setComplete();//创建完成清除session信息
        return "redirect:/";
    }
    
    @ModelAttribute("userTemp")
    public User tempUser(){
        return new User();
    }
}
```

### 方法返回值  

Controller层的方法返回值有：  

**ModelAndView**  

```java
@RequestMapping(value="/index")
    public ModelAndView t(){
        ModelAndView result = new ModelAndView("/test/index");
        result.addObject("key", "value");
        return result;
    }
```

**String**  

```java
@RequestMapping(value="/index")
    public String t(){
        return "/test/index";
    }
```

**void**  

这里要注意，如果是get方法，那么返回的视图名称即为对应的访问地址  

```java
@RequestMapping(value="/index")
    public void t(){
	
    }
```

换句话说，如果视图名称与访问地址相同的话，即可不用返回，默认即可。  


**Object**  

一般用于ajax查询数据，常使用标签@ResponseBody与之配合，一般都转成json数据输出给前台。  

### 数据传递  

后台如何传递数据到前台呢？  

**Model、Map、ModelMap**  

```java
//Spring Web MVC 提供Model、Map或ModelMap让我们能去暴露渲染视图需要的模型数据。
public String createUser(Model model, Map model2, ModelMap model3) {
    model.addAttribute("a", "a");
    model2.put("b", "b");
    model3.put("c", "c");
    System. out.println(model == model2);
    System. out.println(model2 == model3);
    return "success";
}
```

补充：ModelAndView的addObject方法也有类似的功能。


参考网址: <http://blog.sina.com.cn/s/blog_a85398ce0101feek.html>