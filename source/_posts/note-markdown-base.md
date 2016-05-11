title: Markdown语法总结
date: 2014-11-01 22:17:11
categories: note
tags: [markdown]
---

## Markdown语法总结

### <span id="jump">标题</span>  

```java
h1 # 标题 也可以是文字下面加=======  
h2 ## 二级标题 也可以是文字下面加-------（如果上面是空行则变分割线）  
h3 ### 三级标题  
h4 #### 四级标题  
h5 ##### 五级标题  
h6 ###### 六级标题  
```

### 换行与缩进  
尾部加两个空格再换行就是另起一行  
首行缩进要中输入法全角空格  

### 粗体和斜体  

```java
**粗体** 或者 __粗体__  
*斜体* 或者 _斜体_  
```

### 引用、行内代码、嵌套、代码块  
语法|作用|内部标识是否有效  
-----|--------|-----------  
\> 引用 |引用内容 |Y  
\`行内代码\` |在行中显示代码|N  
\`\`嵌套\`\` |嵌套显示|N  
\`\`\`代码块\`\`\` |代码块显示	|N  

### 列表  
* 有序列表  

    ```java
    1. 列表1  
    2. 列表2  
    3. 列表3  
    ```
    
* 无序列表  

    ```java
    *, +,- 都可以，如：  
    * 列表1  
    * 列表2  
    * 列表3  
    ```

### 链接和图片  
#### 直接链接  
`<http://www.baidu.com/>`  
效果  <http://www.baidu.com/>

#### 一般写法  
`[百度一下](http://www.baidu.com/ "百度一下")`  
效果  [百度一下](http://www.baidu.com/ "百度一下")  

#### 图片  
`![img](url "title")`  
效果:  
![img](http://ww2.sinaimg.cn/large/5e8cb366jw1e85r40u55hj20b40b4q2x.jpg "markdown")  

#### 链式引用  

```java
 链接：  
 [文字1][1],[文字2][2]  

 [1]: url "文字1"  
 [2]: url "文字2"  

 图片:  
 ![img][1],![img][2]  

 [1]: url "title1"  
 [2]: url "title2"  
```

效果：  
 [github][1],[开源中国][2]  
 markdown:  
 ![img][3]  

### 其他  

```
 [toc] 显示目录

 脚注：  
 Hello[^hello]  

 [^hello]: 这是一个脚注

 文内跳转:  
 [点击跳转](#jump)  

 <span id="jump">跳转到这里</span>

``` 

[回到文首](#jump)

[1]: https://github.com/ "github"  
[2]: http://www.oschina.net/ "开源中国"  
[3]: http://ww2.sinaimg.cn/large/5e8cb366jw1e85r40u55hj20b40b4q2x.jpg "markdown"  
