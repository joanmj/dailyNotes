# mysql

配置文件位置:

进入sql的命令行界面, 查看sql的安装路径:

```mysql
select @@basedir;
```

查看数据所在路径:

```mysql
select @@datadir;
```

数据所在路径是:

```bash
mysql> select @@datadir;
+---------------------------------------------+
| @@datadir                                   |
+---------------------------------------------+
| C:\ProgramData\MySQL\MySQL Server 5.7\Data\ |
+---------------------------------------------+
1 row in set
```

那么my.ini就在data目录的上一层: C:\ProgramData\MySQL\MySQL Server 5.7\

## 打印注释:

```mysql
show create table docm_comp_doc_td;
```

或者打印效果更好的:

```mysql
show full fields from docm_comp_doc_td
```



# Centos安装

官方安装指南地址:<https://dev.mysql.com/doc/refman/5.6/en/binary-installation.html>

安装包:mysql-5.6.37-linux-glibc2.12-x86_64.tar.gz

进CentOs后安装:

```bash
#
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
cd /usr/local
tar zxvf /path/to/mysql-VERSION-OS.tar.gz
ln -s full-path-to-mysql-VERSION-OS mysql
cd mysql
scripts/mysql_install_db --user=mysql
bin/mysqld_safe --user=mysql &
# Next command is optional
cp support-files/mysql.server /etc/init.d/mysql.server
```

执行后, 安装脚本结果如下:

```bash
[root@hadoop-data-27 mysql]# scripts/mysql_install_db --user=mysql
Installing MySQL system tables...2019-04-20 13:48:54 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2019-04-20 13:48:54 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2019-04-20 13:48:54 0 [Note] ./bin/mysqld (mysqld 5.6.37) starting as process 2198 ...
2019-04-20 13:48:54 2198 [Note] InnoDB: Using atomics to ref count buffer pool pages
2019-04-20 13:48:54 2198 [Note] InnoDB: The InnoDB memory heap is disabled
2019-04-20 13:48:54 2198 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2019-04-20 13:48:54 2198 [Note] InnoDB: Memory barrier is not used
2019-04-20 13:48:54 2198 [Note] InnoDB: Compressed tables use zlib 1.2.3
2019-04-20 13:48:54 2198 [Note] InnoDB: Using Linux native AIO
2019-04-20 13:48:54 2198 [Note] InnoDB: Using CPU crc32 instructions
2019-04-20 13:48:54 2198 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2019-04-20 13:48:54 2198 [Note] InnoDB: Completed initialization of buffer pool
2019-04-20 13:48:54 2198 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
2019-04-20 13:48:54 2198 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
2019-04-20 13:48:54 2198 [Note] InnoDB: Database physically writes the file full: wait...
2019-04-20 13:48:55 2198 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
2019-04-20 13:48:55 2198 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
2019-04-20 13:48:56 2198 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2019-04-20 13:48:56 2198 [Warning] InnoDB: New log files created, LSN=45781
2019-04-20 13:48:56 2198 [Note] InnoDB: Doublewrite buffer not found: creating new
2019-04-20 13:48:57 2198 [Note] InnoDB: Doublewrite buffer created
2019-04-20 13:48:57 2198 [Note] InnoDB: 128 rollback segment(s) are active.
2019-04-20 13:48:57 2198 [Warning] InnoDB: Creating foreign key constraint system tables.
2019-04-20 13:48:57 2198 [Note] InnoDB: Foreign key constraint system tables created
2019-04-20 13:48:57 2198 [Note] InnoDB: Creating tablespace and datafile system tables.
2019-04-20 13:48:57 2198 [Note] InnoDB: Tablespace and datafile system tables created.
2019-04-20 13:48:57 2198 [Note] InnoDB: Waiting for purge to start
2019-04-20 13:48:57 2198 [Note] InnoDB: 5.6.37 started; log sequence number 0
2019-04-20 13:48:57 2198 [Note] Binlog end
2019-04-20 13:48:57 2198 [Note] InnoDB: FTS optimize thread exiting.
2019-04-20 13:48:57 2198 [Note] InnoDB: Starting shutdown...
2019-04-20 13:48:58 2198 [Note] InnoDB: Shutdown completed; log sequence number 1625977
OK

Filling help tables...2019-04-20 13:48:58 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2019-04-20 13:48:58 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2019-04-20 13:48:58 0 [Note] ./bin/mysqld (mysqld 5.6.37) starting as process 3975 ...
2019-04-20 13:48:58 3975 [Note] InnoDB: Using atomics to ref count buffer pool pages
2019-04-20 13:48:58 3975 [Note] InnoDB: The InnoDB memory heap is disabled
2019-04-20 13:48:58 3975 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2019-04-20 13:48:58 3975 [Note] InnoDB: Memory barrier is not used
2019-04-20 13:48:58 3975 [Note] InnoDB: Compressed tables use zlib 1.2.3
2019-04-20 13:48:58 3975 [Note] InnoDB: Using Linux native AIO
2019-04-20 13:48:58 3975 [Note] InnoDB: Using CPU crc32 instructions
2019-04-20 13:48:58 3975 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2019-04-20 13:48:58 3975 [Note] InnoDB: Completed initialization of buffer pool
2019-04-20 13:48:58 3975 [Note] InnoDB: Highest supported file format is Barracuda.
2019-04-20 13:48:58 3975 [Note] InnoDB: 128 rollback segment(s) are active.
2019-04-20 13:48:58 3975 [Note] InnoDB: Waiting for purge to start
2019-04-20 13:48:58 3975 [Note] InnoDB: 5.6.37 started; log sequence number 1625977
2019-04-20 13:48:59 3975 [Note] Binlog end
2019-04-20 13:48:59 3975 [Note] InnoDB: FTS optimize thread exiting.
2019-04-20 13:48:59 3975 [Note] InnoDB: Starting shutdown...
2019-04-20 13:49:00 3975 [Note] InnoDB: Shutdown completed; log sequence number 1625987
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

  ./bin/mysqladmin -u root password 'new-password'
  ./bin/mysqladmin -u root -h hadoop-data-27 password 'new-password'

Alternatively you can run:

  ./bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:

  cd . ; ./bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl

  cd mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/

The latest information about MySQL is available on the web at

  http://www.mysql.com

Support MySQL by buying support/licenses at http://shop.mysql.com

New default config file was created as ./my.cnf and
will be used by default by the server when you start it.
You may edit this file to change server settings

WARNING: Default config file /etc/my.cnf exists on the system
This file will be read by default by the MySQL server
If you do not want to use this, either remove it, or use the
--defaults-file argument to mysqld_safe when starting the server

```

# 数据库基础

## 数据抽象

### 三个层次：

* 物理层、
* 逻辑层(使用的数据模型包括两类)
  * 概念数据模型，
    1. 有实体-联系模型（ERM）
  * 逻辑数据模型
* 视图层。

### 关系模型

1. 面向对象模型、

2. 层次模型和网状模型。

设计师构建数据库模式的方法通常是首先使用E-R模型在高层对数据建模，然后再将其转变成关系模型。在物理层使用的数据模型称为物理数据模型。

数据模型通常由数据结构、数据操作和完整性约束三部分组成。

##  数据库语言

- 1）数据定义语言（DDL），用于定义数据库模式；
- **2）****数据操纵语言****（DML），用于对数据库进行查询和更新；**
- **3）****数据控制语言****（DCL），用于对数据进行权限管理**





*T*  数据库模式

- 在物理层描述数据库中全体存储结构和存取方法，
- 在逻辑层描述数据库中全体数据的逻辑结构和特征。
- 描述了数据库用户能够看见和使用的局部数据的逻辑结构和特征。
- **物理模式（内模式）**
- **辑模式（概念模式）**
- **在视图层也可分为若干模式，称为子模式（外模式）**
- 一个数据库只有一个物理模式和一个逻辑模式，但是子模式有若干个。





*T*  E-R图

- **1）矩形，代表实体型；**
- **2）椭圆，代表属性**
- **3）菱形，代表联系**
- **4）线段，将属性和实体性相连，或将实体型和联系相连。**





*T*关系模型

- 关系模型中常用的关系操作包括：选择、投影、连接、除、并、交、差等查询操作和增加、删除、修改两大部分。关系操作的特点是集合操作方式，即操作的对象和结果都是集合。关系操作可以使用两种方式定义：基于代数的定义称为关系代数；基于逻辑的定义称为关系演算。由于使用变量的不同，关系演算又分为元组关系演算和域关系演算。



关系模型允许定义三类完整性约束：实体完整性、参照完整性和用户定义完整性。其中实体完整性和参照完整性是关系模型必须满足的完整性约束条件。实体完整性规则是：关系的主码不能取空值。参照完整性规则是：外码必须是另一个表中主码的有效值，或者是“空值”。



*T* 连接运算

- 自然连接要求两个关系中进行比较的分量必须是相同的属性组，并且在结果中把重复的属性列去掉。





*T*SQL

- 1. What

  2.  

  3. is it

  4. SQL的数据定义功能
- 模式定义
- 表定义、
- 视图定义
- 索引定义。
- SQL通常不提供修改模式定义、修改视图定义和修改索引定义。用户如果想修改这些对象，只能先将它们删除，然后再重建。
- **基本对象有****表、视图和索引**





*T*    基本表的操作

- *Q*1)       创建表

  - *A*create table 基本表名

    

    

    (列名类型，

    

    ……

    

    完整性约束，

    

    ……

    



)



*Q* 修改表

- *A*alter  table  <基本表名>  add  <列名>  <类型>
- *A*alter  table  <基本表名>  drop  <列名>  <类型>  [cascade | restrict]（cascade表示所有引用到该列的视图和约束也要一起自动删除；restrict表示在没有视图或约束引用该属性时，才能在本表中删除该列，否则拒绝删除。）
- *A*alter  table  <基本表名>  modify  <列名>  <类型>





*Q*撤销表

- *A*drop  table  <基本表名>  [cascade | restrict]





*Q*where子句中可以使用下列运算符：

- *A*l  算术运算符

  

  l  逻辑运算符

  

  l  字符串匹配运算符，包括like，not like

  

  l  集合成员资格运算符，包括in，not in

  

  l  谓词，包括exists，all，some，unique

  

  l  聚合函数，包括avg，min，max，sum和count

  



l  还可以是另一个select语句



*Q*select语句完整语法：

- *A*  select  目标表的列名或列表达式序列

  

  ​        from 基本表名和（或）视图序列

  

  ​        [where 行条件表达式]

  

  ​        [group by  列名序列]

  

  ​               [having  组条件表达式]

  



​        [order by 列名[asc | desc]]



*Q*创建视图：

- *A*  create view <视图名> [<列名> <列名>…]

  

  ​        as <子查询>

  



​        []



*Q*with checkoption表示

- *A*对视图进行增删改是要保证操作的行满足视图定义中的谓词条件（即子查询中的条件表达式）。





*Q*视图最终是定义在基本表之上的，对视图的一切操作最终也要转换为对基本表的操作。视图的好处：

- *A*l  视图能够简化用户的操作

  

  l  视图是用户能以多种角度看待同一数据

  

  l  视图对重构数据库提供了一定程度的逻辑独立性

  



l  视图能够对机密数据提供安全保护