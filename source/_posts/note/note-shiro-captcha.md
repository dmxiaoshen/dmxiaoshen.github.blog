title: Shiro实现验证码认证
date: 2014-12-08 09:17:11
categories: note
tags: [shiro]
---

## Shiro实现验证码认证

上篇最后的遗留问题就是验证码的问题，其实shiro本身是支持验证码扩展的。这里我们可以有两种方法。  
我们前台页面是表单的提交，针对这个我们有两种思路。一是在表单提交之前去检验验证码，通过之后才能提交，  
二是表单先提交上去，与用户名密码一同处理，只是顺序靠前。  

### 第一种方法  

这种方法，就是普通的表单元素验证。这里用到ajax（类似用户名的重复性验证）。同时用到js的一个验证框架query.validationEngine.js.  
具体也不多说，直接看代码:  

```javascript
<script type="text/javascript">
var ctx = "${ctx}";
$(function(){	

	$("#capt").click(function(){
		$("#imgs").attr("src","${ctx}/login/getCaptcha?time="+(new Date()).getTime());
	});
	
	$("#form").validationEngine({
		validationEventTrigger:"",
		promptPosition : "centerRight",
		scroll : false,
		maxErrorsPerField : 1,
		customFunctions:{
			checkCaptcha:checkCaptcha
		},
	    onValidationComplete : function(form,valid){
			if(valid){
				return true;
			}
		}
	});	
	
	function checkCaptcha($field, rules, i, options){
		 var parmCode = $field.val();
		 
		    var isOk = false;
		    $.ajax({
		    	type:"post",
		    	async:false,
		    	url:ctx+"/login/checkCaptcha",
		    	data:{
		    		captcha:parmCode
		    	},
		    	dataType:"json",
		    	success:function(data){	    		
		    			isOk = data;	    		
		    	}
		    	
		    });
		    
		    if(isOk){		    	
		    	return "* 验证码错误";
		    }
	}
});
</script>

<p>
<label>验证码:</label>
<input type="text" maxlength=4 name="captcha" class="validate[required,funcCall[checkCaptcha]]"/>
<img src="getCaptcha" id="imgs" />&nbsp;<a id="capt" href="#">看不清</a>
</p>
```

后台服务:  

```java
 @RequestMapping(value = "/getCaptcha")
    public void getCaptcha(OutputStream out, HttpSession session) {
        String code = RandomStringUtils.random(4,true,true);
        try {
            CaptchaUtil.writeImage(code, out);
            session.setAttribute(AppConstants.CAPTCHA_CODE, code);
        } catch (IOException e) {
            // 网络异常
            session.removeAttribute(AppConstants.CAPTCHA_CODE);
        }
    }
    
 @RequestMapping(value="/checkCaptcha")
 @ResponseBody
  public boolean checkCaptcha(String captcha,HttpSession session){
	String cap = (String)session.getAttribute(AppConstants.CAPTCHA_CODE);
	return !cap.equalsIgnoreCase(captcha);
}
```

这种方法完全不用修改原先的shiro各种配置。  

### 第二种方法  

需要一个CaptchaUsernamePasswordToken类继承UsernamePasswordToken来扩展验证码  

```java
public class CaptchaUsernamePasswordToken extends UsernamePasswordToken {

    /** */
    private static final long serialVersionUID = -2793965333840597519L;

    private String captcha;

    public CaptchaUsernamePasswordToken(String username, char[] password, boolean rememberMe, String host,
            String captcha) {
        super(username, password, rememberMe, host);
        this.captcha = captcha;
    }

    public String getCaptcha() {
        return captcha;
    }

    public void setCaptcha(String captcha) {
        this.captcha = captcha;
    }

}
```

需要有一个异常IncorrectCaptchaException，方便定位到验证码错误  

```java
public class IncorrectCaptchaException extends AuthenticationException {

    /** */
    private static final long serialVersionUID = 4918785444665781681L;

    public IncorrectCaptchaException() {
        super();
    }

    public IncorrectCaptchaException(String message, Throwable cause) {
        super(message, cause);
    }

    public IncorrectCaptchaException(String message) {
        super(message);
    }

    public IncorrectCaptchaException(Throwable cause) {
        super(cause);
    }
}
```

MyAuthenticatingFilter类里面需要添加验证码的判断  

完整如下:  

```java
public class MyAuthenticatingFilter extends FormAuthenticationFilter {

    public static final String DEFAULT_CAPTCHA_PARAM = "captcha";

    private String captchaParam = DEFAULT_CAPTCHA_PARAM;

    public String getCaptchaParam() {
        return captchaParam;
    }

    public void setCaptchaParam(String captchaParam) {
        this.captchaParam = captchaParam;
    }

    protected String getCaptcha(ServletRequest request) {
        return WebUtils.getCleanParam(request, getCaptchaParam());
    }

    @Override
    protected CaptchaUsernamePasswordToken createToken(ServletRequest request, ServletResponse response) {
        String username = getUsername(request);
        String password = getPassword(request);
        String captcha = getCaptcha(request);
        boolean rememberMe = isRememberMe(request);
        String host = getHost(request);
        return new CaptchaUsernamePasswordToken(username, password.toCharArray(), rememberMe, host, captcha);
    }

    // 验证码校验
    protected void doCaptchaValidate(HttpServletRequest request, CaptchaUsernamePasswordToken token) {
        // session中的图形码字符串
        String captcha = (String) request.getSession().getAttribute(
                AppConstants.CAPTCHA_CODE);
        // 比对
        if (captcha != null && !captcha.equalsIgnoreCase(token.getCaptcha())) {
            throw new IncorrectCaptchaException("验证码错误！");
        }
    }
    
    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        CaptchaUsernamePasswordToken token = createToken(request, response);
        if (token == null) {
            String msg = "createToken method implementation returned null. A valid non-null AuthenticationToken " +
                    "must be created in order to execute a login attempt.";
            throw new IllegalStateException(msg);
        }
        try {
            doCaptchaValidate((HttpServletRequest)request, token);
            Subject subject = getSubject(request, response);
            subject.login(token);
            return onLoginSuccess(token, subject, request, response);
        } catch (AuthenticationException e) {
            return onLoginFailure(token, e, request, response);
        }
    }

    @Override
    protected void setFailureAttribute(ServletRequest request, AuthenticationException ae) {
        /** 放入异常的完整信息，控制层controller需要用到 */
        request.setAttribute(getFailureKeyAttribute(), ae);
    }

}
```

LoginController部分代码:

```java
Object obj = request.getAttribute(FormAuthenticationFilter.DEFAULT_ERROR_KEY_ATTRIBUTE_NAME);
AuthenticationException authExp = (AuthenticationException) obj;
if (authExp != null) {
    if(authExp instanceof IncorrectCaptchaException){
	model.addAttribute("errorCode", "2");  
	model.addAttribute("error", "* 验证码错误");
    }
    else if(authExp instanceof UnknownAccountException || authExp instanceof IncorrectCredentialsException){
	model.addAttribute("errorCode", "1");  
	model.addAttribute("error", "* 用户名或密码错误");
    }     
    
    return loginPage(model);
}
```

参考网址: <http://blog.csdn.net/zhengwei223/article/details/9969831>