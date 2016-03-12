---
layout:     post
title:      leetcode上的数据库题目(Medium)
category: blog
description: leetcode 上的数据库题目,Medium级别的。
---

这篇博客主要梳理一下leetcode 上的Medium级别的数据库题目。SQL语句都是支持MySQL的。
***
<br />

##177. Nth Highest Salary
Write a SQL query to get the nth highest salary from the `Employee` table.  

|---
| Id | Salary |
|:----:+:--------:
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
 
For example, given the above Employee table, the *nth* highest salary where *n* = 2 is `200`. If there is no nth highest salary, then the query should return `null`.



###177解答
这道题目先要排序，然后再拿出第N条数据

{% highlight MySQL  %}
#  MySQL 
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      # Write your MySQL query statement below.
    select IFNULL(Salary, NULL) Salary 
	from ( 
		select @row_num := @row_num+1 Rank, Salary
		from ( 
			select Salary,@row_num := 0  from Employee group by Salary desc 
		)t1
	) x  where Rank=N
  );
END
{% endhighlight %}

***
<br />

##178. Rank Scores
Write a SQL query to rank scores. If there is a tie between two scores, both should have the same ranking. Note that after a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no "holes" between ranks. 

|---
| Id | Score |
|:----:+:--------:
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  | 
 
 <br />
For example, given the above `Scores` table, your query should generate the following report (order by highest score):

|---
| Score | Rank |
|:----:+:--------:
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |

<br />
###178解答
这道题目表自联接，再group by求个数

{% highlight MySQL  %}
#  MySQL 
select  max(a.Score) Score,count(distinct b.Score) rank from Scores a
inner join Scores b
on a.Score<=b.Score
group by a.Id 
order by  Score desc 
{% endhighlight %}

***
<br />

##180. Consecutive Numbers
Write a SQL query to find all numbers that appear at least three times consecutively. 

|---
| Id | Num |
|:----:+:--------:
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  | 

For example, given the above `Logs` table, `1` is the only number that appears consecutively for at least three times.


###180解答
这道题目表自联接2次 

{% highlight MySQL  %}
#  MySQL 
select distinct(log1.Num) from Logs log1 
inner join Logs log2 on log1.Num=log2.Num and log1.id+1=log2.id
inner join Logs log3 on log1.Num=log3.Num and log1.id+2=log3.id
{% endhighlight %}
***
<br />

##184. Department Highest Salary
The `Employee` table holds all employees. Every employee has an Id, a salary, and there is also a column for the department Id.

|---
| Id | Name  | Salary | DepartmentId |
|:--:+:-----:+:------:+:------------:
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |

 <br/>
The `Department` table holds all departments of the company.

|---
| Id | Name     |
|:----:+:--------:
| 1  | IT       |
| 2  | Sales    |

<br/>
 Write a SQL query to find employees who have the highest salary in each of the departments. For the above tables, Max has the highest salary in the IT department and Henry has the highest salary in the Sales department.

|---
| Department | Employee | Salary |
|:--:+:-----:+:------:
| IT         | Max      | 90000  |
| Sales      | Henry    | 80000  |

 <br/>

###184解答
先查出每个部门的最高薪，然后拿到对应的信息并排序

{% highlight MySQL  %}
#  MySQL 
select b.Name,a.Name,a.Salary from  Employee a
inner join  Department b on a.DepartmentId=b.id 
inner join 
(
	select  m.DepartmentId,max(m.Salary) Salary 
	from Employee m
	group by m.DepartmentId
)x 
on a.DepartmentId=x.DepartmentId and a.Salary=x.Salary
order by b.Name ASC ,a.Salary DESC,a.Name ASC
{% endhighlight %}


