连接数据库

```
C:\Users\citizen_lu>mysql -uroot -p123456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.39-log MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```



# 1 DDL:操作数据库

操作数据库主要就是对数据库的增删查操作。

## 1.1 查询

查询所有的数据库

```
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jw                 |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.02 sec)

mysql>
```

## 1.2 创建数据库

```
mysql> CREATE DATABASE db1;
Query OK, 1 row affected (0.00 sec)

mysql>
# 如果不存在则创建
mysql> CREATE DATABASE IF NOT EXISTS db1;
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql>
```

## 1.3 删除数据库

```
mysql> DROP DATABASE db1;
Query OK, 0 rows affected (0.02 sec)

mysql> DROP DATABASE IF EXISTS db1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql>
```

## 1.4 使用数据库

```
mysql> USE jw
Database changed
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| jw         |
+------------+
1 row in set (0.00 sec)

mysql>
```

# 2 DDL:操作表

操作表也就是对表进行增（Create）删（Retrieve）改（Update）查（Delete）。

## 2.1 查询表

```
# 查询当前数据库下所有表名称
mysql> SHOW TABLES;
+--------------+
| Tables_in_jw |
+--------------+
| t_admins     |
| t_classes    |
+--------------+
2 rows in set (0.00 sec)

mysql>
# 查询表结构
mysql> DESC t_admins;
+------------+--------------+------+-----+---------+----------------+
| Field      | Type         | Null | Key | Default | Extra          |
+------------+--------------+------+-----+---------+----------------+
| id         | int(11)      | NO   | PRI | NULL    | auto_increment |
| login_name | varchar(255) | NO   | UNI | NULL    |                |
| pwd        | varchar(255) | YES  |     | NULL    |                |
+------------+--------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)

mysql>
```

## 2.2 创建表

```
mysql> CREATE TABLE t_users(
    -> id int,
    -> username varchar(20),
    -> password varchar(32)
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql>
```

## 2.3 数据类型

MySQL支持多种类型，可以分为3类：

- 1、数值

```sql
tinyint : 小整数型，占一个字节
int	： 大整数类型，占四个字节
	eg ： age int
double ： 浮点类型
	使用格式： 字段名 double(总长度,小数点后保留的位数)
	eg ： score double(5,2)   
```

- 2、日期

```sql
date ： 日期值。只包含年月日
	eg ：birthday date ： 
datetime ： 混合日期和时间值。包含年月日时分秒
```

- 3、字符串

```sql
char ： 定长字符串。
	优点：存储性能高
	缺点：浪费空间
	eg ： name char(10)  如果存储的数据字符个数不足10个，也会占10个的空间
varchar ： 变长字符串。
	优点：节约空间
	缺点：存储性能底
	eg ： name varchar(10) 如果存储的数据字符个数不足10个，那就数据字符个数是几就占几个的空间	
```

## 2.4 删除表

```
mysql> DROP TABLE t_users;
Query OK, 0 rows affected (0.01 sec)

mysql>
mysql> DROP TABLE IF EXISTS t_users;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql>
```

## 2.5 修改表

```
# 修改表名
mysql> ALTER TABLE t_users RENAME to t_user;
Query OK, 0 rows affected (0.01 sec)

mysql>

# 添加一列
mysql> ALTER TABLE t_user ADD address varchar(50);
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql>

# 修改数据类型
mysql> ALTER TABLE t_user MODIFY address char(50);
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql>

# 修改列名和数据类型
mysql> ALTER TABLE t_user CHANGE address addr varchar(50);
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql>

# 删除列
mysql> ALTER TABLE t_user DROP addr;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql>
```









































