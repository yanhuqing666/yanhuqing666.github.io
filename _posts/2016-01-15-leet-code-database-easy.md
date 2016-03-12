---
layout:     post
title:      leetcode上的数据库题目(Easy)
category: blog
description: leetcode 上的数据库题目,Easy级别的。
---

leetcode 上的数据库题目不是很多，大都比较简单，先选几道easy级别的题目来开始我的博客之旅吧。SQL语句都是支持MySQL的。
***
<br />

##175. Combine Two Tables

Table: `Person` 

|-----------------+------------|
| Column Name     |Type		   | 
|:---------------:|:----------:|
| PersonId        |int         |
| FirstName       |varchar     |
| LastName        |varchar     | 
|-----------------+------------|


 
PersonId is the primary key column for this table.

Table: `Address`

|---
| Column Name |Type
|:-:|:-:
| AddressId | int
| PersonId |int
| City |varchar
| State| varchar


  
AddressId is the primary key column for this table.


Write a SQL query for a report that provides the following information for each person in the Person table, regardless if there is an address for each of those people:

`FirstName, LastName, City, State`



###175解答
这道题目考察连接的问题，不做细描述，直接给出答案

{% highlight MySQL  %}
#  MySQL 
select a.FirstName,a.LastName,b.City,b.State 
from Person a left join Address b 
on a.PersonId=b.PersonId
{% endhighlight %}
***
<br />

## 176. Second Highest Salary
 Write a SQL query to get the second highest salary from the `Employee` table.

|---
| Id |Salary
|:-:|:-:
| 1 |100
| 2 |200
| 3 |300 



 

For example, given the above Employee table, the second highest salary is `200`. If there is no second highest salary, then the query should return `null`.


###176解答
这道题目考察的是是排序外加分页的功能，见答案

{% highlight MySQL  %}
#  MySQL 
select IFNULL( 
(select e.Salary from Employee e 
group by e.Salary order by e.Salary desc limit 1, 1)
, NULL) SecondHighestSalary;
{% endhighlight %}

***
<br />
##181. Employees Earning More Than Their Managers

 The `Employee` table holds all employees including their managers. Every employee has an Id, and there is also a column for the manager Id. 
 
|---
|Id|Name|Salary|ManagerId
|:-:|:-:|:-:|:-:
| 1 |Joe|70000|3
| 2 |Henry|80000|4
| 3 |Sam   | 60000  | NULL      |
| 4 | Max| 90000| NULL  |

<br /> 
Given the `Employee` table, write a SQL query that finds out employees who earn more than their managers. For the above table, Joe is the only employee who earns more than his manager.

|---
| Employee 
|:-:
| Joe    
 
 <br />
 
###181解答
这道题目考查的是一张表关联自己。  
{% highlight MySQL  %}
#  MySQL 
select a.Name from Employee a
inner join Employee b on a.ManagerId=b.Id
and a.Salary>b.Salary
{% endhighlight %}
 

***
<br />
##182. Duplicate Emails

Write a SQL query to find all duplicate emails in a table named `Person`.
 
|----
| Id | Email 
|:--:+:-------:
| 1  | a@b.com 
| 2  | c@d.com 
| 3  | a@b.com 
 
<br /> 
For example, your query should return the following for the above table:

|---
| Email   
|:-------:
| a@b.com 

Note: All emails are in lowercase.
 <br />
###182解答
这道题目考查的having 的使用。

{% highlight MySQL  %}
#  MySQL 
select Email from Person
group by Email having count(*)>1
{% endhighlight %}

***
<br /> 
##183. Customers Who Never Order

Suppose that a website contains two tables, the `Customers` table and the `Orders` table. Write a SQL query to find all customers who never order anything.
Table: `Customers`.

|----
| Id | Name  
|:--:+:-----:
| 1  | Joe   
| 2  | Henry 
| 3  | Sam   
| 4  | Max   
 
 <br /> 
Table: `Orders`.

|----
| Id | CustomerId 
|:--:+:----------:
| 1  | 3          
| 2  | 1         

<br /> 
Using the above tables as example, return the following: 

|---
| Customers 
|:---------:
| Henry     
| Max      
 
###183解答
这道题目考查的左连接匹配不上的情况。

{% highlight MySQL  %}
#  MySQL 
select a.name from Customers  a
left join Orders b on a.id =b.CustomerId
where b.id is null
{% endhighlight %}

***
<br />
##196. Delete Duplicate Emails

Write a SQL query to delete all duplicate email entries in a table named `Person`, keeping only unique emails based on its _smallest_ **Id**.
 
|---
| Id | Email            |
|:--:+:----------------:
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |

 Id is the primary key column for this table.
 
For example, after running your query, the above `Person` table should have the following rows:


|---
| Id | Email            |
|:--:+:-----------------:
| 1  | john@example.com |
| 2  | bob@example.com  |


<br />
 
###196解答
通过group by 聚合的子查询得到留下来的结果，删掉其他就对了。

{% highlight MySQL  %}
#  MySQL 
delete a from Person a
inner join 
(select Email,min(Id) Id from Person group by Email) b
on a.Email=b.Email and a.Id<>b.Id
{% endhighlight %}

***
<br />
##197. Rising Temperature

Given a `Weather` table, write a SQL query to find all dates' Ids with higher temperature compared to its previous (yesterday's) dates.
 
|--- 
| Id(INT) | Date(DATE) | Temperature(INT) |
|:-------:+:----------:+:----------------: 
|       1 | 2015-01-01 |               10 |
|       2 | 2015-01-02 |               25 |
|       3 | 2015-01-03 |               20 |
|       4 | 2015-01-04 |               30 | 
 
 <br /> 
For example, return the following Ids for the above Weather table: 

|---
| Id |
|:---:
|  2 |
|  4 |

<br /> 
###197解答
这道题目主要是join时候做一个日期的运算，考察日起计算的函数

{% highlight MySQL  %}
#  MySQL 
select b.Id from Weather a inner join Weather b 
on  DATE_ADD(a.Date,INTERVAL 1 DAY)=b.Date and a.Temperature<b.Temperature
{% endhighlight %}

 