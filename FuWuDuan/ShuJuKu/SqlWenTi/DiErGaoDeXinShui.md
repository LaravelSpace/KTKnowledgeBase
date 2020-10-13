#### 第二高的薪水

SQL架构

```sql
Create table If Not Exists Employee (Id int, Salary int)
Truncate table Employee
insert into Employee (Id, Salary) values ('1', '100')
insert into Employee (Id, Salary) values ('2', '200')
insert into Employee (Id, Salary) values ('3', '300')
```

编写一个 SQL 查询，获取 `Employee` 表中第二高的薪水（Salary） 。

```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

例如上述 `Employee` 表，SQL查询应该返回 `200` 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 `null`。

```
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

#### MySQL

```sql
# Write your MySQL query statement below

select IFNULL((select DISTINCT `Salary` from `employee` order by `Salary` DESC limit 1 offset 1),null) as SecondHighestSalary
```

这里需要用到 MySQL 的 **IFNULL()** 函数，如果 Sql 这么写的话 `select Salary from Employee order by Salary DESC limit 1 offset 1`，当数据库里只有一条数据时就会报错。

**DISTINCT** 的作用是防止数据库中所有的数据都是一样的，这样就不存在第二高，所以要做去重处理。如果 Sql 这么写的话 `select IFNULL((select DISTINCT Salary from employee order by Salary DESC limit 1 offset 1),null) as SecondHighestSalary`，当数据库里的数据都是一样的时候，依然会输出一样的那个值。