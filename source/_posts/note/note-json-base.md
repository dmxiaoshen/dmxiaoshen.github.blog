title: Json简单分析
date: 2014-11-23 20:17:11
categories: note
tags: [json]
---

## Json简单分析

> JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。它基于JavaScript（Standard ECMA-262 3rd Edition - December 1999）的一个子集。 JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C, C++, C#, Java, JavaScript, Perl, Python等）。这些特性使JSON成为理想的数据交换语言。易于人阅读和编写，同时也易于机器解析和生成(网络传输速度快)。

### json语法规则  

#### 1. 数据在名称/值对中

书写格式如下:  

```javascript
"name":"charles"
```

**注意**: 这里名称必须用双引号，值如果是字符串也必须用双引号。 

**json值可以是:**  

* 数字(整数或浮点数)
* 字符串(在双引号中)
* 逻辑值(true或false)
* 数组(在方括号中)
* 对象(在花括号中)
* null


#### 2. 数据由逗号分隔  

书写格式如下:  

```javascript
{"name":"charels" , "sex":"M" , "age":100}
```

#### 3. 花括号保存对象  

书写格式如下:  

```javascript
{"firstName":"John" , "lastName":"Doe"}
```

#### 4. 方括号保存数组  

**数组可以包含多个对象**  

书写如下:  

```javascript
{
"employees": [
{ "firstName":"John" , "lastName":"Doe" },
{ "firstName":"Anna" , "lastName":"Smith" },
{ "firstName":"Peter" , "lastName":"Jones" }
]
}
```

### json在javascript中的使用  

```javascript
var jsonObject = {"firstName":"John" , "lastName":"Doe" };//定义一个json对象
var jsonStr = '{"firstName":"John" , "lastName":"Doe" }';//定义一个json字符串
var employees = [
{ "firstName":"Bill" , "lastName":"Gates" },
{ "firstName":"George" , "lastName":"Bush" },
{ "firstName":"Thomas" , "lastName": "Carter" }
];//定义一个json对象数组  
//访问也很方便  employees[0].firstName  返回 Bill
//修改数据也很方便  employees[0].lastName="Jobs"  
```

### js中json字符串和json对象的相互转化

```javascript
//json字符串转json对象 
1. var jsonObj = eval('('+jsonStr+')');
2. var jsonObj = (new Function("return"+jsonStr))();
3. var jsonObj = JSON.parse(jsonStr);

//json对象转json字符串
var jsonStr = JSON.stringify(jsonObj);
```


这里有一篇关于java端bean与json字符串之间转化的文章:  
<http://www.cnblogs.com/hoojo/archive/2011/04/22/2024628.html>