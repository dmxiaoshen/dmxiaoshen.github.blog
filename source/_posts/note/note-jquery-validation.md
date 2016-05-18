title: Jquery ValidationEngine验证插件
date: 2014-12-09 14:17:11
categories: note
tags: [jquery, validationEngine]
---

## Jquery ValidationEngine验证插件

在开发前台页面的时候，免不了要用到验证，js有很多验证插件，这里主要讲Jquery ValidationEngine这款插件，  
使用简单，功能强大。[jQuery.validationEngine](http://www.position-relative.net/creation/formValidator/)

### form表单验证  

一般我们用到form标签的时候，都需要对表单进行验证，简单使用如下：  

```javascript
<form id="formId" action="add" method="post">
<p>
<label>用户名:</label><input type="text" name="username" class="validate[required]"  />
</p>
<p>
<label>密码:</label><input type="password" name="password" class="validate[required]"  />
</p>
<p><input type="submit" value="新增" /></p>
</form>
```  

调用：  

```javascript
$("#formId").validationEngine();
```  

以上是最简单的使用方式`class="validate[required]"`这个就是验证的规则，这里只是对必填项做了约束，其他一些规则文末会给出详细。  

那么问题来了

如果我要新增自己的验证规则呢，还有就是有后台ajax验证呢，比如上面新增用户这个例子，如果我有2个需求  

1. 有一个确认密码输入框，验证规则为跟密码输入框必须输入一致。
2. 用户名不能重复。  

这个时候框架本身没有这种规则，但是我们可以自己写，页面如下:  

```javascript
<form id="formId" action="${ctx}/user/add" method="post">
<p>
<label>用户名:</label><input type="text" name="username" class="validate[required,funcCall[checkUsername]]"  />
</p>
<p>
<label>密码:</label><input type="password" name="password" class="validate[required]"  />
</p>
<p>
<label>确认密码:</label><input type="password" name="password1" class="validate[required,funcCall[checkPwd]]" />
</p>
<p><input type="submit" value="新增" /></p>
</form>
```  

验证的时候:  

```javascript
$("#formId").validationEngine({
	validationEventTrigger:"",
	promptPosition:"centerRight",
	scroll:false,
	maxErrorsPerField:1,
	customFunctions:{
		checkUsername:checkUsername,
		checkPwd:checkPwd
	},
	onValidationComplete:function(form,valid){
		if(valid){
			return true;
		}
	}
});

function checkUsername($field,rules,i,options){
	var userName = $field.val();
	
	var isOk = true;
	$.ajax({
		url:ctx+"/user/checkUsername",
		type:"post",
		dataType:"json",
		async:false,
		data:{
			username:userName
		},success:function(data){
			isOk = data;
		}
	});
	
	if(!isOk){
		return "* 用户名已经存在";
	}
}

function checkPwd($field,rules,i,options){
	var pwd = $field.val();
	var pwdPre = $("input[name='password']").val();
	if(pwd!=pwdPre){
		return "* 两次输入的密码不一致";
	}
}
```  

后台checkUsername:  

```java
@RequestMapping(value="/checkUsername")
    @ResponseBody
    public boolean checkUsername(String username){
        User u = userService.findUserByName(username);
        return u==null;
    }
```  

对表单的验证就差不多这样了，以上应该能解决大部分的问题了。

### 非表单的验证  

除了用到表单，有时候还会用到ajax提交，这个时候也需要验证，比如说一些修改操作，这里以修改用户为例，  
这时候我们不用表单，但是我们要用一个div把区域包起来，并且该div需要有一个class为**validationEngineContainer**。  

页面如下:  

```javascript
<div class="validationEngineContainer">
<p>
<input type="hidden" value="${user.id}" id="userId" />
<label>用户名:</label><input type="text" id="username" value="${user.username}" class="validate[required]"  />
</p>
</div>
<p><input type="button" value="新增" id="update"/></p>
```  

调用验证如下:  

```javascript
$("#update").click(function(){
	if(!$(".validationEngineContainer").validationEngine('validate')){
		return  ;
	}
	var userId = $("#userId").val();
	var username = $("#username").val();
	$.ajax({
		async:false,
		type:"post",
		url:ctx+"/user/update",
		dataType:"text",
		data:{
			id:userId,
			username:username
		},success:function(data){
			if(data=='success'){
				alert("修改成功");
			}else{
				alert("修改失败");
			}
		}
	});
});
```  

如果有自定义的规则呢？

页面如下:  

```javascript
div class="validationEngineContainer">
<p>
<input type="hidden" value="${user.id}" id="userId" />
<label>用户名:</label><input type="text" id="username" value="${user.username}" class="validate[required,funcCall[checkUsername]]"  />
</p>
</div>
<p><input type="button" value="新增" id="update"/></p>
```  

调用如下:  

```javascript
function checkUsername($field,rules,i,options){
	var userName = $field.val();
	
	var isOk = true;
	$.ajax({
		url:ctx+"/user/checkUsername",
		type:"post",
		dataType:"json",
		async:false,
		data:{
			username:userName
		},success:function(data){
			isOk = data;
		}
	});
	
	if(!isOk){
		return "* 用户名已经存在";
	}
}
	
$("#update").click(function(){
	var validator = $(".validationEngineContainer").validationEngine({
		validationEventTrigger:"",
		promptPosition:"centerRight",
		scroll:false,
		maxErrorsPerField:1,
		customFunctions:{
			checkUsername:checkUsername
		}
	});
	
	if(!validator.validationEngine('validate')){
		return  ;
	}
	var userId = $("#userId").val();
	var username = $("#username").val();
	$.ajax({
		async:false,
		type:"post",
		url:ctx+"/user/update",
		dataType:"text",
		data:{
			id:userId,
			username:username
		},success:function(data){
			if(data=='success'){
				alert("修改成功");
			}else{
				alert("修改失败");
			}
		}
	});
});
```  

### 总结  

至此，对于表单和非表单的验证都已经加上，并且可以自定义规则，下面补充下框架自带的一些规则。  

[验证规则及API查看](http://blog.csdn.net/amohan/article/details/17528705)