<!--
 * @Author: your name
 * @Date: 2020-06-02 18:57:30
 * @LastEditTime: 2020-06-02 20:22:12
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\DataBase\MySQL.md
--> 
# MySQL

## 数据库基础
&emsp;&emsp;数据库是一种以某种有组织的方式存储的数据集合。再简单一点，保存有组织的数据的容器

### 表
&emsp;&emsp;每一条数据都存在一个文件中，这个文件在数据库领域中称为表。这是一种结构化的文件，可以用来存储特定类型的数据。通俗点来说，就像是excel里的表格。

&emsp;&emsp;表只能存放一类数据，不应该将不同的数据放在一个表中存储。

&emsp;&emsp;每个表有一个表名，同一个数据库中不允许有两个表名相同。

### 模式
&emsp;&emsp;模式表示表中的一些特性，它们定义了在表中该如何存储，可以存什么数据，各部分如何命名等。

### 列
&emsp;&emsp;每个表都是一列一列组成的，跟excel中的列相似，每一列存的都应该是相同的数据类型，另外每个表所能容许的数据类型不一定一样

### 行
&emsp;&emsp;代表每一个个体。

### 主键
&emsp;&emsp;所谓主键就是代表每一行的一个ID，用于唯一标识每行，一般会存在第一列。例如一张表存的是班级里所有学生的联系方式，第一列存的是每个人的名字，这时每个名字就是每一行的主键。

&emsp;&emsp;当然理论上任意一列都可以是主键，只要满足：
- 任意两行主键值都不相同
- 不能有哪个主键值是NULL

也可以把多个列作为主键，上述条件必须应用到构成主键的所有列，所有列值组合必须唯一。

&emsp;&emsp;主键非常重要，所以需要遵循以下几个原则：
- 不更新主键中的值
- 不重用主键中的值
- 不在主键中存将会被更改的值
  
## SQL
&emsp;&emsp;SQL是结构化查询语言，是一种专门用来和数据库通信的语言。

## MySQL
&emsp;&emsp;数据库的所有存储、检索、管理和处理都是由数据库软件（DBMS）来实现的，MySQL就是其中一种。

&emsp;&emsp;一般来说DBMS分为两类，一种是基于共享文件系统，例如Access，用于桌面用途，不用于高端或者关键应用；另一种是基于客户机-服务器的数据库，例如MySQL，Oracle等。

### 连接
&emsp;&emsp;数据库的连接与连接服务器相似，需要指定IP地址和端口，以及用户名和密码。

### 选择数据库
&emsp;&emsp;使用USE关键字，例如需要使用一个叫wells的数据库：
```
USE wells
```
这时就将当前数据库切换到了wells，在进行操作之前一定要有这一步。

### 数据库列表
&emsp;&emsp;显示所有的数据库：
```
SHOW DATABASES
```

### 当前数据库内可用的表
```
SHOW TABLES
```
返回当前数据库内所有可用的表。

### 某个表内的列
```
SHOW COLUMNS FROM wellst
```
将会返回wellst这个表中所有的列，并且有它们的类型，是否允许NULL，键信息，默认值等信息。也可以使用：
```
DESCRIBE wellst
```
效果相同。

### SHOW的其他命令
```
SHOW STATUS //显示服务器信息
SHOW CREATE DATABASE
SHOW CREATE TABLE //显示用于创建特定数据库或表的语句
SHOW GRANTS //显示授予用户的权限
SHOW ERRORS
SHOW WARNING //显示服务器错误或者警告
```