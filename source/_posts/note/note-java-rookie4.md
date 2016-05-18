title: Java新手的通病(4)——异常处理使用不当
date: 2015-04-27 14:17:11
categories: note
tags: [java]
---

## Java新手的通病(4)——异常处理使用不当

1.空catch语句块

2.没有使用finally

尤其是对那些资源的释放处理。

3.笼统的catch语句块

4.使用函数返回值进行错误处理

5.不清楚Checked Exception和Runtime Exception的区别

**Runtime Exception**运行时异常，不需要显示的捕获处理，常见的有　　
NullPointException(空指针异常)　　
ClassCastException(类型强制转换异常)　　
IndexOutOfBoundsException(下标越界异常)　　
IllegalArgumentException(非法参数异常)　　
ArithmeticException(算术异常/除零异常)　　
NumberFormatException(数字格式异常)

**Checked Exception**(IOException,SQLException,FileNotFoundException,DataFormatException,)需要显示捕获的异常。

### java异常体系:

![java异常体系](http://images.cnitblog.com/blog/367379/201306/15233816-b0fb36dcf6de4d14af4a0f8206439959.jpg)

### java容器体系:

![java容器体系](http://i1153.photobucket.com/albums/p501/dmxiaoshen/hexo/javacol_zpsxzlqoyay.png)

### java io体系:

![java io体系](http://i1153.photobucket.com/albums/p501/dmxiaoshen/hexo/javaio_zpsujaf4cjd.jpg)







