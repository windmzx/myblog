---

title: "Sql练习"
date: 2020-06-04T09:40:46+08:00
---

> sql训练

# 表结构如下

![表结构](https://img.0xaa.top//Diagram1.png)

- *查询每门课程被选修的学生数* 

```sql
SELECT
	c_name, count( s_id ) as 选课人数
FROM
	course
	JOIN score ON score.c_id = course.c_id 
GROUP BY
	score.c_id	
```

- 查询出只有两门课程的全部学生的学号和姓名

```
SELECT
	s_id,
	s_name 
FROM
	student 
WHERE
	s_id IN ( SELECT s_id FROM score GROUP BY s_id HAVING count( c_id ) = 2 )
```

- *查询男生、女生人数* 

```
SELECT
	s_sex,
	count( s_id ) 
FROM
	student 
GROUP BY
	s_sex
```



- 查询名字中含有"风"字的学生信息

```sql
SELECT
	s_id,
	s_name 
FROM
	student 
WHERE
	s_name LIKE '%风%'
```



- 查询1990年出生的学生名单

```sql
SELECT
	s_id,
	s_name,
	s_birth 
FROM
	student 
WHERE
	s_birth LIKE '1990%'
```



- 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列

```sql
SELECT
	c_id,
	ROUND( avg( s_score ), 2 ) AS avg_scoce 
FROM
	score 
GROUP BY
	c_id 
ORDER BY
	avg_scoce DESC,
	c_id ASC
```



- 查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩

```sql
SELECT
	student.s_id,
	s_name,
	avg( s_score ) AS avg_score 
FROM
	student
	LEFT JOIN score ON student.s_id = score.s_id 
GROUP BY
	s_id 
HAVING
	avg_score > 85
```



- 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```sql
SELECT
	student.s_id,
	s_name,
	round( avg( s_score ), 2 ) 
FROM
	student
	LEFT JOIN score ON student.s_id = score.s_id 
WHERE
	student.s_id IN ( SELECT s_id FROM score WHERE s_score < 60 GROUP BY s_id HAVING count( 1 ) >= 2 ) 
GROUP BY
	student.s_id,
	student.s_name
```

- *检索"01"课程分数小于60，按分数降序排列的学生信息*

```sql
SELECT
	* 
FROM
	student
	LEFT JOIN score ON student.s_id = score.s_id 
WHERE
	c_id = '01' 
	AND s_score < 60 
ORDER BY
	s_score DESC
```

- 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

```
select s_id from score group by 
```

