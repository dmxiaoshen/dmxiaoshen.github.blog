title: Java新手的通病(1)——对算法和数据结构不熟悉
date: 2015-04-24 14:17:11
categories: note
tags: [java]
---

## Java新手的通病(1)——对算法和数据结构不熟悉

### 1.什么时候该用数组型容器、什么时候该用链表型容器

内存分配固定的（可确定的）时候用数组型容器，反之则用链表型容器。数组一旦创建，长度是不可变的。

### 2.什么是散列函数? HashMap的实现原理是什么?

散列函数式根据原始输入得出一个散列值（可以想象下md5加密)。

简单来说，HashMap内部是一个数组加链表的形式,Entry[] table是数组，Entry<K,V>是链表（内部有指向下一个元素的引用next)。首先会根据传入的key值计算相应的hash值，根据hash值来确定所对应数组的下标，找到该位置，如果没有存在元素，放在第一个，如已经存在元素，则放在开头位置，也就是说最先加入放链尾，最后加入放链头。确定位置以后，Entry<K,V>当做一个整体放入位置（value类似于key的附属).取数据的时候也是类似，根据key只计算hash值，找到位置，如该位置不止一个（有链表）就需要遍历，判断依据为对key做判断（基本类型==，引用类型equals）。
补充一下，HashMap内部capacity (初始容量，默认16）和loadFactor（负载因子，默认0.75），当实际存储数据size>capacity\*loadFactor时，数组会扩容为原来的2倍（需要消耗性能，原有数据必须重新计算存储位置）。
可提升性能的点：capacity最好为2的n次方。

### 3.什么是递归？尝试写一个递归（比如递归目录树遍历)。

递归是程序调用自身的一种编程技巧。

```java
public static void digui(File f){
	if(f.isFile()){
		System.out.println(f.getAbsolutePath());
		return;
	}

	File[] array = f.listFiles();
	for(File file : array){
		digui(file);
	}
}
```
### 4.什么是算法复杂度

算法复杂度是程序运行时所需要的资源，包括时间资源和内存资源，一个算法的评价主要从时间复杂度和空间复杂度来考虑。

### 5.你是否理解空间换时间的思想？

通过消耗更多的资源（如内存等）来加快程序运行速度。“缓存”就是应用了此思想。

java中多线程的资源共享上，同步机制采用了“以时间换空间”的方式：访问串行化，对象共享化。而ThreadLocal采用了“以空间换时间”的方式：访问并行化，对象独享化。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。

### 6.写一个针对整数数组的冒泡排序函数，看看你要修改几次才能跑通。

```java
public static void bubbleSort(int[] a){
	int length = a.length;
	int temp = 0;
	for(int i=1;i<length;i++){
		for(int j=0;j<length-i;j++){
			if(a[j]>a[j+1]){
				temp = a[j];
				a[j] = a[j+1];
				a[j+1] = temp;

			}
		}
	}
}
```

修改1次。

### 7.写一个针对整数数组的二分查找函数，看看你要修改几次才能跑通。

```java
/** 传进来的数组a是有序数组 */
public static int halfFind(int[] a,int num,int left,int right){
	if(left==right){
		if(a[left]==num){
			return left;
		}
		return -1;
	}
	int pos = (left+right)/2;
	if(a[pos]==num){
		return pos;
	}else if(a[pos]>num){
		return halfFind(a,num,left,pos-1);
	}else{
		return halfFind(a,num,pos+1,right);
	}
}
```

修改0次。




参考链接: [http://zhangshixi.iteye.com/blog/672697](http://zhangshixi.iteye.com/blog/672697)