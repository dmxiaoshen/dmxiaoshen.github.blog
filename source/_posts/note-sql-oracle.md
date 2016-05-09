title: Oracle中merge into用法
date: 2015-01-04 14:17:11
categories: note
tags: [oracle]
---

## Oracle中merge into用法

merge命令可以用来处理一个表中的数据插入或更新到另一个表，也可以对单表处理，存在则修改不存在则插入。  

### 语法  

```sql
merge into table a
using table b
on (a.xxx=b.xxx)
when matched then
update set a.col=b.col
when not matched then
insert(col1,col2,col3) values(b.col1,b.col2,b.col3)
```

以上是最基础的语法,简单分析下:  

* table a  插入或修改数据的目标表
* table b  数据源，可以是**表、视图、子查询** 
* on 条件 满足则修改，不满足插入


### 实战  

#### 场景一  

有一张产品表production,已有一些数据,现在又有一张production_new表，并且从外部导入了一些数据，现在要把production_new表的数据整合到production表中去，  
原有的production数据当然是不能删的，因为其主键id已被很多地方依赖, 这个时候 merge into就能发挥作用了

```sql
merge into production a
using production_new b
on (a.产品编号=b.产品编号)
when matched then 
update set a.x=b.x,a.xx=b.xx,a.xxx=b.xxx
when not matched then
insert(x,xx,xxx) values(b.x,b.xx,b.xxx)
```

#### 场景二  

有一张表production,已有数据,但是程序中会不时的获取一些信息插入该表，对已经存在相同编号(字段为pro_no)的产品只需update，如是新产品则insert

```sql
merge into production a
using (select '100001' as pro_no from dual) b
on (a.pro_no=b.pro_no)
when matched then
update set a.x=新值1,a.xx=新值2,a.xxxx=新值3
when not matched then 
insert(pro_no,x,xx,xxx) values('100001',新值1,新值2,新值3)
```

如果存在编号为100001的产品则更新，否则新增。

### 千万注意  

b中select出来的数据，每一条都跟a进行on中的条件比较，如果满足则更新，不满足就插入。  
换句话说**更新与插入的总记录数，就是b中select出来的记录数**  
所以如果想要语句按你的逻辑走，第一步就要检查b中是否能select出数据。