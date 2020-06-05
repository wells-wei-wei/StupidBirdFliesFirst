<!--
 * @Author: your name
 * @Date: 2020-06-02 18:57:30
 * @LastEditTime: 2020-06-05 22:33:18
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\DataBase\MySQL.md
--> 
# MySQL

## 数据库基础
&emsp;&emsp;数据库是一种以某种有组织的方式存储的数据集合。再简单一点，保存有组织的数据的容器

###
 表
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

### 检索数据
#### 检索单个列
```
SELECT name FROM wells
```
表示利用SELECT语句从wells的表中检索一个叫name的列，这时会返回此列所有的数据。注意这时的数据如果没有明确排序查询结果则乱序排列。另外命令本身可以空行，所以上述命令也可以表示成：
```
SELECT name 
FROM wells
```

#### 检索多个列
```
SELECT name, age, gender
FROM wells
```

#### 检索所有列
```
SELECT *
FROM wells
```
这里使用通配符*来表示wells表中的所有列。

#### 检索数值不同的行
&emsp;&emsp;以上检索都会把整列数据中每一行都显示出来，但是比如说一群人的年龄有很多数值相同的，现在想要返回不同的所有值，可以使用DIDTINCT命令：
```
SELECT DISTINCT age
FROM wells
```
假如DISTINCT后面跟了多个列的话，那么会将所有不同的组合全部列出来。

#### 限制结果
&emsp;&emsp;可以使用LIMIT命令来限制返回的数量：
```
SELECT name 
FROM wells
LIMIT 5
```
表示返回前5行；
```
SELECT name 
FROM wells
LIMIT 5，15
```
表示返回从第5行开始的15行（从第0行开始）。如果超出了总行数不会报错，有多少行返回多少行

#### 使用完全限定的表名
```
SELECT wells.name 
FROM data.wells
```

以上语句中，SELECT是一个主句，FROM是一个子句

### 数据排序
#### 数据排序
&emsp;&emsp;可以使用SELECT的子句ORDER BY命令来给返回的列排序：
```
SELECT name 
FROM wells
ORDER BY name
```
即使用名字的字母顺序排序，可以以其他的列为基准进行排序。

#### 按多个列排序
```
SELECT name, age,id
FROM wells
ORDER BY id,name
```
这里表示查询了多个列，返回之后将先按照id的顺序进行排序，id相同的再按照name的顺序排序，总之排序顺序和语句的顺序是相对应的。

#### 降序排列
&emsp;&emsp;语句的排序默认是升序排列，如果想要降序排列可以使用DESC语句：
```
SELECT name, age,id
FROM wells
ORDER BY id DESC
```
如果想要升序降序混着用，则可以：
```
SELECT name, age,id
FROM wells
ORDER BY id DESC, name
```
这就是先按照id降序排列，然今相同id的再按照name升序排列。DESC只能用于它前面的那个列，如果多个列都要降序排列的话那就必须都写上DESC。

与DESC相反的是ASC，但是它没什么用，因为默认就是升序排列。

以下命令可以找出年龄最大的人：
```
SELECT name, age,id
FROM wells
ORDER BY age DESC
LIMIT 1
```

### 过滤数据
#### 使用WHERE子句
&emsp;&emsp;WHERE子句可以指定搜索条件进行过滤，在表名之后给出：
```
SELECT name, age,id
FROM wells
WHERE age=10
```
即只返回age为10的行。

在同时使用ORDER BY和WHERE子句时，应该让ORDER BY位于WHERE之后，否则将会产生错误。

#### WHERE子句的操作符
![where](where.jpg)

#### 空值检查
&emsp;&emsp;空值表示当前位置没有存任何值，它与0、空字符、空格都不一样，是真的什么都没有，用NULL表示：
```
SELECT name
FROM wells
WHERE age IS NULL
```
操作符中的等于和不等于操作符都不会匹配到NULL的值。

#### AND操作符
&emsp;&emsp;WHERE还可以写出很多复杂的查询语句，例如加上逻辑操作符：
```
SELECT name, age,id
FROM wells
WHERE age=10 AND id<10
```
AND就是逻辑与，表示返回age=10且id<10的行。

#### OR操作符
```
SELECT name, age,id
FROM wells
WHERE age=10 OR id<10
```
OR就是逻辑或，表示返回age=10或者id<10的行。

#### 计算次序
&emsp;&emsp;WHRER中可以包含任意数目的AND和OR操作符，但是它们的计算顺序是不一样的，通常sql会先计算AND，再计算OR，所以如果要自己指定先算什么的话要用括号：
```
SELECT name, age,id
FROM wells
WHERE (age=10 OR id<10) AND name='musk' //另外这种查询是否相等的操作符在字符上不区分大小写
```

#### IN操作符
&emsp;&emsp;IN操作符用来指定条件范围，范围中的每个条件都可以进行匹配。IN取合法值的由逗号分隔的清单，全都括在圆括号中：
```
SELECT name, age,id
FROM wells
WHERE id IN (1,2,3)
```
意思就是把id等于1，2和3的行都挑出来，其实跟OR是一样的

#### NOT操作符
&emsp;&emsp;NOT就是逻辑中的非，用来否定后面跟的条件：
```
SELECT name, age,id
FROM wells
WHERE id NOT IN (1,2,3)
```
就是把id不等于1，2和3的行都挑出来

### 通配符过滤
&emsp;&emsp;在搜索子句中使用通配符，必须使用LIKE操作符。LIKE指示MySQL，后跟的搜索模式利用通配符匹配而不是直接相等匹配进行比较。
#### %通配符
&emsp;&emsp;%表示任何字符出现任意次数：
```
SELECT name, age,id
FROM wells
WHERE name LIKE 'Jake%'
```
表示找出所有name以Jake开头的行，%告诉MySQL接受Jake之后的任意字符，不管它有多少字符。

如果只是想找name中带有Jake的，可以在Jake前也加上%：
```
SELECT name, age,id
FROM wells
WHERE name LIKE '%Jake%'
```
表示匹配任何位置包含文本anvil的值，而不论它之前或之后出现什么字符。

另外%不能匹配到NULL