---
layout:     post
title:      leetcode上的数据库题目(Hard)
category: blog
description: leetcode 上的数据库题目,Hard级别的。
---

这篇博客主要梳理一下leetcode 上的Hard级别的数据库题目。SQL语句都是支持MySQL的。
***
<br />
  

##185. Department Top Three Salaries
The `Employee` table holds all employees. Every employee has an Id, a salary, and there is also a column for the department Id.

|---
| Id | Name  | Salary | DepartmentId |
|:--:+:-----:+:------:+:------------:
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |

 <br/>
The `Department` table holds all departments of the company.

|---
| Id | Name     |
|:--:+:-----:
| 1  | IT       |
| 2  | Sales    |

<br/>
 Write a SQL query to find employees who earn the top three salaries in each of the department. For the above tables, your SQL query should return the following rows.

|---
| Department | Employee | Salary |
|:--:+:-----:+:------:
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |

 <br/>

###185解答
先查出每个人关联到所在部门薪水大于自己的人，约束为小于等于3人，再关联其他信息。

{% highlight MySQL  %}
#  MySQL 
select Department,Employee,Salary from 
(
	select  m.Id,max(n.Name)  Department ,
	max(m.Name) Employee,max(m.Salary) Salary 
	from Employee m
	inner join Department n on m.DepartmentId=n.id
	inner join Employee b on m.DepartmentId=b.DepartmentId  
	and m.Salary<=b.Salary
	group by m.Id  having count(DISTINCT b.Salary)<=3 
)x
ORDER BY Department ASC ,Salary DESC,Employee ASC
{% endhighlight %}


##262. Trips and Users
The `Trips` table holds all taxi trips. Each trip has a unique Id, while Client_Id and Driver_Id are both foreign keys to the Users_Id at the `Users` table. Status is an ENUM type of (‘completed’, ‘cancelled_by_driver’, ‘cancelled_by_client’).

|---
| Id | Client_Id | Driver_Id | City_Id |        Status      |Request_at|
|:--:+:-----:+:------:+:--:+:-----:+:------:
| 1  |     1     |    10     |    1    |     completed      |2013-10-01|
| 2  |     2     |    11     |    1    | cancelled_by_driver|2013-10-01|
| 3  |     3     |    12     |    6    |     completed      |2013-10-01|
| 4  |     4     |    13     |    6    | cancelled_by_client|2013-10-01|
| 5  |     1     |    10     |    1    |     completed      |2013-10-02|
| 6  |     2     |    11     |    6    |     completed      |2013-10-02|
| 7  |     3     |    12     |    6    |     completed      |2013-10-02|
| 8  |     2     |    12     |    12   |     completed      |2013-10-03|
| 9  |     3     |    10     |    12   |     completed      |2013-10-03| 
| 10 |     4     |    13     |    12   | cancelled_by_driver|2013-10-03|

 <br/>
The `Users` table holds all users. Each user has an unique Users_Id, and Role is an ENUM type of (‘client’, ‘driver’, ‘partner’).


|---
| Users_Id | Banned |  Role  |
|:--:+:-----:+:------:
|    1     |   No   | client |
|    2     |   Yes  | client |
|    3     |   No   | client |
|    4     |   No   | client |
|    10    |   No   | driver |
|    11    |   No   | driver |
|    12    |   No   | driver |
|    13    |   No   | driver | 

 <br/>
Write a SQL query to find the cancellation rate of requests made by unbanned clients between **Oct 1, 2013** and **Oct 3, 2013**. For the above tables, your SQL query should return the following rows with the cancellation rate being rounded to two decimal places.

|---
|     Day    | Cancellation Rate |
|:--:+:-----:
| 2013-10-01 |       0.33        |
| 2013-10-02 |       0.00        |
| 2013-10-03 |       0.50        |

<br/> 


###262解答
这道题目有点疑问，先说正确答案。

{% highlight MySQL  %}
#  MySQL 
select t.Request_at as Day, 
round(
	sum(case when t.Status='completed' then 0 else 1 end )/count(*)
,2) as Cancellation_Rate 
from Trips t
inner join Users u on t.Client_Id=u.Users_Id and u.Banned='NO'
where t.Request_at>='2013-10-01' and t.Request_at<='2013-10-03'
group by t.Request_at
{% endhighlight %}

但我第一次做的时候是这样写的

{% highlight MySQL  %}
#  MySQL 
select t.Request_at as Day, 
round(
sum(case when t.Status='cancelled_by_client' then 1 else 0 end )/count(*)
,2) as Cancellation_Rate 
from Trips t
inner join Users u on t.Client_Id=u.Users_Id and u.Banned='NO'
where t.Request_at>='2013-10-01' and t.Request_at<='2013-10-03'
group by t.Request_at
{% endhighlight %}


所以焦点在于made by unbanned clients 是指未禁止的乘客所涉及到的所有取消的trips，还是未禁止的乘客所涉及到的由乘客的取消的trips，有英语达人可以适当讨论下