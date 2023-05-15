# DML

DML主要是对数据进行增（insert）删（delete）改（update）操作。

## 1 添加数据

```mysql
-- 给指定列添加数据
INSERT INTO stu(id,name) VALUES(1,'张三');

-- 给全部列添加数据
INSERT INTO stu(id,`name`,gender,birthday,score,email,tel,`status`) VALUES(2,'李四','男','1998-3-5',90,'abc@outlook.com',13112345678,1);

-- 可以省略列名的列表
INSERT INTO stu VALUES(3,'王五','男','1990-9-9',99,'wang@outlook.com',13112348765,1);

-- 批量添加数据
INSERT INTO stu VALUES
(3,'王五','男','1990-9-9',99,'wang@outlook.com',13112348765,1),
(4,'王五','男','1990-9-9',99,'wang@outlook.com',13112348765,1),
(5,'王五','男','1990-9-9',99,'wang@outlook.com',13112348765,1);
```

## 2 修改数据

```mysql
-- 将张三的性别改为女
UPDATE stu SET gender='女' WHERE `name`='张三';

-- 将张三的生日改为 1999-12-12 分数改为99.99
UPDATE stu SET birthday='1999-12-12',score=99.99 WHERE `name`='张三';

-- 如果update语句没有加where条件，则会将表中所有数据全部修改
UPDATE stu SET gender='女';
```

## 3 删除数据

```mysql
-- 删除张三记录
DELETE FROM stu WHERE `name`='张三';

-- 删除stu表中所有的数据
DELETE FROM stu;
```

# DQL

```mysql
-- 填充一些信息
CREATE TABLE stu (
 id int, -- 编号
 name varchar(20), -- 姓名
 age int, -- 年龄
 sex varchar(5), -- 性别
 address varchar(100), -- 地址
 math double(5,2), -- 数学成绩
 english double(5,2), -- 英语成绩
 hire_date date -- 入学时间
);


INSERT INTO stu(id,NAME,age,sex,address,math,english,hire_date) 
VALUES 
(1,'马运',55,'男','杭州',66,78,'1995-09-01'),
(2,'马花疼',45,'女','深圳',98,87,'1998-09-01'),
(3,'马斯克',55,'男','香港',56,77,'1999-09-02'),
(4,'柳白',20,'女','湖南',76,65,'1997-09-05'),
(5,'柳青',20,'男','湖南',86,NULL,'1998-09-01'),
(6,'刘德花',57,'男','香港',99,99,'1998-09-01'),
(7,'张学右',22,'女','香港',99,99,'1998-09-01'),
(8,'德玛西亚',18,'男','南京',56,65,'1994-09-02');
```



## 1 基础查询

```mysql
-- 查询多个字段
SELECT * FROM stu;

SELECT address FROM stu;

-- 去除重复结果
SELECT DISTINCT address FROM stu;

-- 起别名
SELECT name,math 数学成绩,english as 英语成绩 FROM stu;
```

## 2 条件查询

```mysql
-- 1.查询年龄大于20岁的学员信息
select * from stu where age > 20;

-- 2.查询年龄大于等于20岁的学员信息
select * from stu where age >= 20;

-- 3.查询年龄大于等于20岁 并且 年龄 小于等于 30岁 的学员信息
select * from stu where age >= 20 &&  age <= 30;
select * from stu where age >= 20 and  age <= 30;
select * from stu where age BETWEEN 20 and 30;

-- 4.查询入学日期在'1998-09-01' 到 '1999-09-01'  之间的学员信息
select * from stu where hire_date BETWEEN '1998-09-01' and '1999-09-01';

-- 5. 查询年龄等于18岁的学员信息
select * from stu where age = 18;

-- 6. 查询年龄不等于18岁的学员信息
select * from stu where age != 18;
select * from stu where age <> 18;

-- 7. 查询年龄等于18岁 或者 年龄等于20岁 或者 年龄等于22岁的学员信息
select * from stu where age = 18 or age = 20 or age = 22;
select * from stu where age in (18,20 ,22);

-- 8. 查询英语成绩为 null的学员信息  
-- 注意： null值的比较不能使用 = != 。需要使用 is  is not
select * from stu where english = null; -- 不行的
select * from stu where english is null;
select * from stu where english is not null;

-- 模糊查询 like
-- _:代表单个任意字符
-- %:代表任意个数字符

-- 1. 查询姓'马'的学员信息
select * from stu where name like '马%';

-- 2. 查询第二个字是'花'的学员信息   
select * from stu where name like '_花%';

-- 3. 查询名字中包含 '德' 的学员信息
select * from stu where name like '%德%';
```

## 3 排序查询

```mysql
-- ASC：升序排列（默认值）
-- DESC：降序排列

-- 1.查询学生信息，按照年龄升序排列 
select * from stu order by age ;

-- 2.查询学生信息，按照数学成绩降序排列
select * from stu order by math desc ;

-- 3.查询学生信息，按照数学成绩降序排列，如果数学成绩一样，再按照英语成绩升序排列
select * from stu order by math desc , english asc ;
```

## 4 聚合函数

将一列数据作为一个整体，进行纵向计算。null值不参与所有聚合函数运算。

```mysql
-- 计算总人数
SELECT COUNT(id) FROM stu;
```



## 5 分组查询

```mysql
-- 1. 查询男同学和女同学各自的数学平均分
select sex, avg(math) from stu group by sex;

-- 注意：分组之后，查询的字段为聚合函数和分组字段，查询其他字段无任何意义
-- name字段没有意义
select name, sex, avg(math) from stu group by sex;

-- 2. 查询男同学和女同学各自的数学平均分，以及各自人数
select sex, avg(math),count(*) from stu group by sex;

-- 3. 查询男同学和女同学各自的数学平均分，以及各自人数，要求：分数低于70分的不参与分组
select sex, avg(math),count(*) from stu where math > 70 group by sex;

-- 4. 查询男同学和女同学各自的数学平均分，以及各自人数，要求：分数低于70分的不参与分组，分组之后人数大于2个的。
select sex, avg(math),count(*) from stu where math > 70 group by sex having count(*)  > 2;
```

**where 和 having 区别：**

* 执行时机不一样：where 是分组之前进行限定，不满足where条件，则不参与分组，而having是分组之后对结果进行过滤。

* 可判断的条件不一样：where 不能对聚合函数进行判断，having 可以。

## 6 分页查询

```mysql
SELECT 字段列表 FROM 表名 LIMIT  起始索引 , 查询条目数;

-- 1. 从0开始查询，查询3条数据
select * from stu limit 0 , 3;

-- 2. 每页显示3条数据，查询第1页数据
select * from stu limit 0 , 3;

-- 3. 每页显示3条数据，查询第2页数据
select * from stu limit 3 , 3;

-- 4. 每页显示3条数据，查询第3页数据
select * from stu limit 6 , 3;

-- 起始索引 = （当前页码 - 1） * 每页显示的条数
```

























