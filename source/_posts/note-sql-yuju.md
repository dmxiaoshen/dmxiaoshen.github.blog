title: SQL常用语句
date: 2014-11-20 16:17:11
categories: note
tags: [sql语句]
---

## SQL常用语句

### 1. 如何更新一张表的数据，来源是另一张表(或者是它自身)?  

```javascript
 update a set a.xxx = b.xxx from 表B b,表A a where a.xxx='条件1' and b.xxx='条件2' and a.xxx=b.xxx
```

### 2. 如何根据已有的表创建新的表，如历史表，备份表等，如果还想保留数据呢?  

```javascript
#只表结构不带数据
create table table_new like table_old  
#表结构和数据都要  
create table table_new as select col1,col2,... from table_old 
```

### 3. 如何给一张表增加一列或删除一列?

```javascript
#增加
alter table tableName add column colName type  
#删除
alter table tableName drop column colName
```

### 4. 如何给一张表插入数据，数据来自其他表的字段?  

```javascript
insert into A(a,b,c) select d,e,f from B
```


参考网址: <http://www.cnblogs.com/yubinfeng/archive/2010/11/02/1867386.html>