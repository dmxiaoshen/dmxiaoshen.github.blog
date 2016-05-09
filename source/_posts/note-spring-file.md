title: SpringMVC文件上传下载
date: 2015-09-10 11:20:30
categories: note
tags: [Spring, SpringMVC, 文件]
---

## SpringMVC文件上传下载

web开发难免会碰到文件上传下载这样的功能。

点[这里](https://github.com/hmkcode/Spring-Framework) 有很多Spring相关的项目demo。

回到正题，文件上传无非就是前台选择文件，后台Controller接收文件然后处理。

页面需要引入以下js及css

```javascript
<script src="js/jquery.1.9.1.min.js"></script>

<script src="js/vendor/jquery.ui.widget.js"></script>
<script src="js/jquery.iframe-transport.js"></script>
<script src="js/jquery.fileupload.js"></script>

<!-- bootstrap just to have good looking page -->
<script src="bootstrap/js/bootstrap.min.js"></script>
<link href="bootstrap/css/bootstrap.css" type="text/css" rel="stylesheet" />
```

上传按钮可以这么写:

```javascript
<input style="display:none" id="fileupload" type="file" name="fileName" data-url="rest/controller/upload">
<input type="button" value="上传文件" id="upButton" />
```

相应的js这样写:

```javascript
	$("#upButton").click(function(){
	     $('#fileupload').click();//这里把fileupload隐藏起来了，通过别的按钮来出发点击事件
	});

	$('#fileupload').fileupload({
        dataType: 'json',
        drop: function (e, data) {
	        return true;
	    },change: function (e, data) {	    	
	        return true;    		      
	    },add: function (e, data) {
	    	maxFileSize = 10000000; // 10MB
			minFileSize = 1;
			acceptFileTypes=/(\.|\/)(gif|jpe?g|png)$/i;
			file = data.files[0];
			if(file.size<minFileSize){
				alert("文件不能为空或是文件夹");
				return false;
			}
			if(file.size>maxFileSize){
				alert("文件大小不能大于10M");
				return false;
			}
			if (!acceptFileTypes.test(file.type) && !acceptFileTypes.test(file.name)){
				alert("只能上传图片");
				return false;
			}

			data.submit().success(function (result, textStatus, jqXHR) {
				if(result=="success"){
					alert("文件上传成功");
				}else{
					alert("文件上传失败");
				 }

			}).error(function (jqXHR, textStatus, errorThrown) {
				alert(errorThrown);
			});
		 }

    });
```

后台代码开始前，需要配置spring-MVC.xml,要增加对文件上传的支持：

```xml
<bean id="multipartResolver"  class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

然后需要jar包(spring那些就不贴了):

```xml
    <dependency>
		<groupId>commons-io</groupId>
		<artifactId>commons-io</artifactId>
		<version>2.4</version>
    </dependency>
   <dependency>
		<groupId>commons-codec</groupId>
		<artifactId>commons-codec</artifactId>
		<version>1.6</version>
  </dependency>
```

后台接收文件:

```cpp
	@RequestMapping(value = "/upload", method = RequestMethod.POST)
	public String upload(MultipartFile fileName, HttpServletRequest request,
			HttpServletResponse response) {
		try {

			FileUtils.copyInputStreamToFile(fileName.getInputStream(),
					new File("D:/temp/files/", fileName.getOriginalFilename()));

		} catch (Exception e) {
			e.printStackTrace();
		}

		return "success";

	}

```

下载文件就比较简单了，前台:

```javascript
<a href='rest/controller/get/1'>Click</a>
```

Controller处理:

```cpp
	@RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
	public void download(HttpServletRequest request, HttpServletResponse response,
			@PathVariable String id) {
		 
		String fileName = "a.txt";//通过id去获取文件名,这里简单处理写死
		String filePath = "D:/temp/files/a.txt";//通过id去获取文件保存路径,这里简单处理写死
		try {
			response.setContentType("application/octet-stream; charset=utf-8");
			String agent = request.getHeader("User-Agent");
			if (agent != null) {
				agent = agent.toLowerCase();
				if (agent.indexOf("firefox") != -1) {
					// 解决空格会截断问题
					fileName = "=?UTF-8?B?"
							+ (new String(Base64.encodeBase64(fileName
									.getBytes("UTF-8")))) + "?=";
				} else if (agent.indexOf("msie") != -1) {
					fileName = URLEncoder.encode(fileName, "UTF-8");
					fileName = fileName.replaceAll("\\+", "%20"); // 解决编码后空格变+号的情况
				} else if (agent.indexOf("chrome") != -1) {
					fileName = "=?UTF-8?B?"
							+ (new String(Base64.encodeBase64(fileName
									.getBytes("UTF-8")))) + "?=";
				} else {
					fileName = "=?UTF-8?B?"
							+ (new String(Base64.encodeBase64(fileName
									.getBytes("UTF-8")))) + "?=";
				}
			}
			response.setHeader("Content-disposition", "attachment; filename=\""
					+ fileName + "\"");
			FileCopyUtils.copy(new FileInputStream(new File(filePath)),
					response.getOutputStream());
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
```

至此，简单的springMVC文件上传下载功能就完成了。