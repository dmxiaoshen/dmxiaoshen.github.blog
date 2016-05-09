title: CSS常用样式
date: 2015-04-17 13:17:11
categories: note
tags: [css]
---

## CSS常用样式

![标准盒子模型](http://i1153.photobucket.com/albums/p501/dmxiaoshen/hexo/hzmx_zpsdkg1ltlb.jpg)  

### 盒子模型距离

body {**margin**: 1em 2em 3em 4em}  

等同于("上右下左")  

body {**margin-top**:1em; **margin-right**:2em; **margin-bottom**:3em; **margin-left**:4em;}

类似的  **padding** 和 **border** 也是如此用法。  

### 边框

p {**border**: thin solid #ddffdd} 边框三个参数（border-width,border-style,border-color）

**border-width**: thin medium thick  (细 中 粗)

**border-style**:dotted solid double dashed; （点 实 双 虚）

**border-color**: #fff

**border-collapse**边框是否合并 separate（分开） collapse（合并一个边框） inherit（继承父亲）

### 背景

div {**background**: transparent url(img.jpg) 50% 0 fixed no-repeat}

对应5个属性：**background-color,background-image,background-position,background-attachment,background-repeat**  

css3里面有一个属性 **background-size**:100% 0 可以用来设置背景图片拉伸。

### 字体

**line-height**: 15px  行间距　　
**font-weight**: bold 字体加粗　　
**font-size**: 120% 字体大小　　
**color**: #dd3f37 字体颜色　　
**font-family**: Georgia 何种字体 

### a标签  

a{**text-decoration**:none}  underline  overline  line-through blink  超链接下划线

a:link{} 未访问　　
a:visited{} 已访问　　
a:hover{**cursor**:pointer} 鼠标悬停　　
a:active{} 被选择　　

鼠标变小手 cursor:pointer

### ul li标签

横排 去点 

ul{**list-style**:none; margin:0px; padding:0px; width:auto;}

ul li{**float**:left;}

### 浮动float

float:right/left是子块级元素流集合面向父级元素的定位，定位的关键词使用**margin/padding**。兄弟块元素之间进行相对的定位均基于移动后的新位置进行重新渲染，可以重叠，原位置被清空。

##伪元素:before和:after

一般是用来清除浮动的，如 .clearfix:after{content:","; **display**:block; height:0; visibility:hidden; **clear**:both;}

### position

相对定位 **postion:relative** 是子块级元素面向父级元素的相对定位，定位关键字使用**left/right/top/bottom**。兄弟块元素之间相对进行定位，但是position移动后，原位置依然保留。而且随后的兄弟块元素定位基于被移走前的位置。

绝对定位 **position:absolute** 元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。

**z-index**属性，表示元素显示的层次关系（可被用于将在一个元素放置于另一元素之后）。