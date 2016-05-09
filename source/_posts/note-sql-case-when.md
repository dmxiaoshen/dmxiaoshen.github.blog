title: SQL中的case when语句
date: 2015-01-08 15:11:13
categories: note
tags: [sql语句]
---

## SQL中的case when语句

### case when语法

case when语句有两种语法，**简单case函数**和**case搜索函数**。

简单case函数；

```sql
select case sex when '1' then '男' 
when '2' then '女' 
else '其他' end as '性别' 
from employee;
```

case搜索函数:

```sql
select case when salary<600 then 'A' 
when salary>=600 and salary<1000 then 'B' 
else 'C' end as '工资级别' 
from employee;
```

**两者区别是:**简单case函数when后面是值，case搜索函数when后面是条件表达式，明显后者更为灵活，  
就拿例子来说，工资(salary)按范围分等级而不是具体的值。

### 与sum函数结合使用

**问题:如何统计employee表男员工和女员工的数量?**  

```sql
select sum(case sex when '1' then 1 else 0 end) as '男员工数量' ,
sum(case sex when '2' then 1 else 0 end) as '女员工数量' 
from employee;
```

### 典型的行列转换

典型的成绩表

学生号|科目|成绩
------|----|----
1|语文|79
1|数学|88
2|数学|95
3|语文|92
2|语文|89
3|数学|79

要求需要如下展示  

学生号|语文|数学|
------|----|----|
1|79|88
2|89|95
3|92|79

```sql
select 学生号,sum(case 科目 when '语文' then 成绩 end) as '语文',
sum(case 科目 when '数学' then 成绩 end) as '数学' 
from 成绩表 
group by 学生号;
```

**ps: case when还可以用在where语句和group by语句后面。**