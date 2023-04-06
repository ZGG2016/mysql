# [SQL22：统计各个部门的工资记录数](https://www.nowcoder.com/practice/6a62b6c0a7324350a6d9959fa7c21db3?tpId=82&&tqId=29774&rp=1&ru=/ta/sql&qru=/ta/sql/question-ranking)

## 1、题目


统计各个部门的工资记录数，给出部门编码dept_no、部门名称dept_name以及部门在salaries表里面有多少条记录sum

```sql
CREATE TABLE `departments` (
`dept_no` char(4) NOT NULL,
`dept_name` varchar(40) NOT NULL,
PRIMARY KEY (`dept_no`));

CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));

CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

## 2、题解


```sql
select dp.dept_no,dp.dept_name,count(*) sum 
from (dept_emp de join salaries s on de.emp_no=s.emp_no) 
join departments dp on de.dept_no=dp.dept_no 
group by dp.dept_no,dp.dept_name;

-- ---------------------

select a.dept_no,a.dept_name,r.sum
from departments a join 
(
    select w.dept_no,count(*) as sum
    from dept_emp w join salaries e on w.emp_no = e.emp_no
    group by w.dept_no
) r 
on a.dept_no = r.dept_no

-- ---------------------

...
FROM dept_emp AS de, departments AS d, salaries AS s
WHERE de.dept_no = d.dept_no
AND s.emp_no = de.emp_no
...
```

## 3、涉及内容