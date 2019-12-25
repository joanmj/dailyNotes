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

查看系统配置变量, group_concat的最长长度:
```mysql
 show variables like '%group_concat_max_len%';
```
设置系统变量:
```mysql
SET [GLOBAL | SESSION] group_concat_max_len = val;
```
## 打印注释:

```mysql
show create table docm_comp_doc_td;
```

或者打印效果更好的:

```mysql
show full fields from docm_comp_doc_td
```
## 插入：
### 批量插入技巧
1. 前提：要更新或者忽略的字段必须是主键或者存在唯一索引
2. sql:
* 存在就更新：
~~~sql
REPLACE INTO tablename(field1,field2,...) VALUES(value1, value2,...)
~~~
* 如果存在就忽略：
~~~sql
INSERT IGNORE tablename(field1,field2,...) VALUES(value1, value2,...)
~~~

### 插入前判断是否存在
~~~sql
INSERT INTO table(field1, field2, fieldn) SELECT 'field1',
'field2', 'fieldn' FROM DUAL WHERE NOT EXISTS(SELECT field FROM
table WHERE field = ?)
~~~
### 复制某条记录多次插入,同时每次插入时修改一个值
正确做法:
如果要修改的值不是查询出来的,那么需要动态造表,用DUAL:
```sql
INSERT INTO table (Key, Val1, Val2)
SELECT d.newKey, t.Val1, t.Val2
FROM table t
cross join (select 2 NewKey from dual union all
            select 3 NewKey from dual union all
            select 4 NewKey from dual) d;
```
如果要修改的值可以查询出来,那么用查询结果插入:
```sql
INSERT INTO table (Key, Val1, Val2)
SELECT d.FKey, t.Val1, t.Val2
FROM table t
cross join (select FKey
            from SomeOtherTable
            Where ......) d;
```
其中cross join是将表1和表2相乘;返回的是笛卡尔积
## 查询:

### 分页:

```sql
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 0;
```

如果第4页只有1条记录，最终结果集按实际数量1显示。`LIMIT 3`表示的意思是“最多3条记录”。

可见，分页查询的关键在于，首先要确定每页需要显示的结果数量`pageSize`（这里是3），然后根据当前页的索引`pageIndex`（从1开始），确定`LIMIT`和`OFFSET`应该设定的值：

- `LIMIT`总是设定为`pageSize`；
- `OFFSET`计算公式为`pageSize * (pageIndex - 1)`。

这样就能正确查询出第N页的记录集。

#### 获得总页数:

聚合查询, 每3个一页:

```mysql
 SELECT CEILING(COUNT(*) / 3) FROM students;
```

CEILING函数是获取大于某值的最小整数值，换句话说就是对某一个值进行向上取整;

### 聚合:

通过COUNT/AVG/MAX/MIN/SUM等聚合函数可以计算, 另外用GROUP BY分组, 可以多个column分组, 例如:

```sql
SELECT class_id, gender, COUNT(*) num FROM students GROUP BY class_id, gender;
```

### 多表查询:

```sql
SELECT
    s.id sid,
    s.name,
    s.gender,
    s.score,
    c.id cid,
    c.name cname
FROM students s, classes c
WHERE s.gender = 'M' AND c.id = 1;
```

### 连接查询:

查询语句一般是这样的:

```sql
SELECT ... FROM tableA ??? JOIN tableB ON tableA.column1 = tableB.column2;
```

Join的方式:

1. INNER JOIN是选出两张表都存在的记录：:![inner-join](assets/l.png)

2. LEFT OUTER JOIN是选出左表存在的记录：![left-outer-join](assets/l-1568190246550.png)

   一般都把 LEFT OUTER JOIN 省略掉成 LEFT JOIN;

3. RIGHT OUTER JOIN是选出右表存在的记录：

   ![right-outer-join](assets/l-1568190264026.png)

4. FULL OUTER JOIN则是选出左右表都存在的记录：

   ![full-outer-join](assets/l-1568190274933.png)

### 分类汇总:

1.创建测试表：

```sql
CREATE TABLE test_ROLLUP_1 (
  StateCode  CHAR(6),
  DepCode   CHAR(6),
  SendMoney  INT
);
```

2.插入测试语句：

```sql
INSERT INTO test_ROLLUP_1
SELECT '100001',  '310001',  3000  UNION ALL
SELECT '100001',  '310002',  1500  UNION ALL
SELECT '100002',  '320001',  4200  UNION ALL
SELECT '100003',  '330001',  1800  UNION ALL
SELECT '100003',  '330002',  2100  UNION ALL
SELECT '100004',  '340001',  2500;
```

3.使用rollup实现分类汇总功能：



```
SELECT
    IFNULL(StateCode, '合计:') AS StateCode,
    IFNULL(DepCode, '小计:') AS DepCode,
    SUM(SendMoney) AS SendMoney
FROM
    test_ROLLUP_1
GROUP BY
    StateCode,
    DepCode WITH ROLLUP;
```

效果如图所示：

![img](assets/548763-20180105152304799-1274206956.png)

### 查询时插入指定类型的列:

```sql
select *,cast(0 as decimal(18,3)) as pickqty  from gb
```
### 行转列查询
举例代码：
```sql
SELECT user_name ,
    MAX(CASE course WHEN '数学' THEN score ELSE 0 END ) 数学,
    MAX(CASE course WHEN '语文' THEN score ELSE 0 END ) 语文,
    MAX(CASE course WHEN '英语' THEN score ELSE 0 END ) 英语
FROM test_tb_grade
GROUP BY USER_NAME;
```
说明：
1. course是表中的列，根据这列的值，查询时创建新的列，因此
```sql
 MAX(CASE course WHEN '数学' THEN score ELSE 0 END ) 数学,
```
  内容为数学则创建数学列;
2. 想让多列内容依据user name分组到一行,则需要使用group by;
以上就完成行转列了



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



# ibatis相关:

[入门基础推荐](https://www.cnblogs.com/estellez/p/4120605.html)

## 读取配置得到**SqlMapClient：**

```java
private static SqlMapClient sqlMapClient = null;
static{
  try{
       Reader reader = com.ibatis.common.resource.Resources.getResourceAsReader(“Ibatis总配置文件路径”);
       sqlMapClient = com.ibatis.sqlMap.client.SqlMapClientBuilder.builderSqlMapClient(reader);
       reader.close();
　　}catch(IOException e){
       异常处理…….
　　}
}
```

## 查询相关

Ibatis只能传递一个参数，如果又多个参数需要封装在一个对象中。

1. 模糊查询:

   如name like‘%c%’。在Ibatis中有两种写法：



   写法1：在java方法中传递参数时写成：”%字段名%”。

   写法2：在Ibatis的sql语句中可以写成如：`where name=’%$字段名$%’`。

2. 查下语句:

   ```xml
   <select id=”Ibatis查询实体操作Id” resultClass=”查询结果类型” >
          select * from 实体对应的表名;
   </select>
   ```



3. 内嵌参数:

   语法为：`#参数值：数据库中数据类型：内嵌参数#`

   例如:

   ```xml
   <statement id=”insertProduct” parameterClass=”Product”>
         Insert into Product(PRD_ID,PRD_DESC)  values(#id:Number:-999999#,#desc:varchar:noEntry#);
   </statement>
   ```



4. 定义查询的参数和返回结果:

   ```xml
   <parameterMap class="commonj.sdo.DataObject" id="parameterMap">
       <parameter javaType="date" jdbcType="DATE" property="dateType"/>
   </parameterMap>
   <resultMap class="commonj.sdo.DataObject" id="resultMap">
       <result column="TYPEID" javaType="string" property="typeId"/>
   </resultMap>
   ```

   也可以定义参数类名的Alias:

   ```xml
    <typeAlias alias="User" type="com.pojo.User" />
   ```

   定义返回的resultMap的属性然后:

   ```xml
   <resultMap id="UserResult" class="User">
       <result property="id" column="ID" />
       <result property="username" column="USERNAME" />
       <result property="password" column="PASSWORD" />
   </resultMap>
   ```



5. 参数:

   1. #xxx#

       代表xxx是属性值, 是map里面的key或者是你的pojo对象里面的属性,

   ibatis会自动在它的外面加上引号,表现在sql语句是这样的:

   `where xxx = 'xxx' `;

   2. `$xxx$`

      则是把xxx作为字符串拼接到你的sql语句中, 比如order by topicId , 语句这样写:

      ```sql
       order by #xxx#
      ```

       ibatis 就会把他翻译成order by 'topicId' （这样就会报错）

      语句这样写:

      ```sql
       order by $xxx$
      ```

      ibatis 就会把他翻译成order by topicId



# 工具

# Navicat快捷键:

**快捷键**

**Navicat 主窗口**

| **键**                  | **动作**                 |
| ----------------------- | ------------------------ |
| CTRL+G                  | 设置位置文件夹           |
| CTRL+#（# 代表 0 至 9） | 从收藏夹列表打开对象窗口 |
| F6                      | 命令列界面               |
| CTRL+H                  | 历史日志                 |
| CTRL+Q                  | 新建查询                 |
| F12                     | 仅显示活跃对象           |

**常规**

| **键**                        | **动作**       |
| ----------------------------- | -------------- |
| CTRL+N                        | 新建对象       |
| SHIFT+CTRL+#（# 代表 0 至 9） | 添加收藏夹     |
| F8                            | Navicat 主窗口 |
| CTRL+TAB 或 SHIFT+CTRL+TAB    | 下一个窗口     |
| F1                            | 说明           |
| CTRL+F1                       | 在线文件       |

**表设计器**

| **键**   | **动作**       |
| -------- | -------------- |
| CTRL+O   | 打开表         |
| CTRL+F   | 查找字段       |
| F3       | 查找下一个字段 |
| SHIFT+F3 | 查找上一个字段 |

**表查看器或视图查看器**

| **键**                         | **动作**                 |
| ------------------------------ | ------------------------ |
| CTRL+D                         | 设计表或设计视图         |
| CTRL+Q                         | 查询表或查询视图         |
| CTRL+F                         | 查找文本                 |
| F3                             | 查找下一个文本           |
| CTRL+G                         | 前往行                   |
| CTRL+ 左箭头                   | 当前记录的第一个数据列   |
| CTRL+ 右箭头                   | 当前记录的最后一个数据列 |
| CTRL+HOME                      | 当前列的第一个数据行     |
| CTRL+END                       | 当前列的最后一个数据行   |
| CTRL+PAGE UP 或 CTRL+ 上箭头   | 当前窗口的第一个数据行   |
| CTRL+PAGE DOWN 或 CTRL+ 下箭头 | 当前窗口的最后一个数据行 |
| CTRL+R                         | 在筛选向导套用筛选       |
| SHIFT+ 箭头                    | 选择单元格               |
| CTRL+ENTER                     | 打开编辑器来编辑数据     |
| INSERT 或 CTRL+N               | 插入记录                 |
| CTRL+DELETE                    | 删除记录                 |
| CTRL+S                         | 应用记录改变             |
| ESC                            | 取消记录改变             |
| CTRL+T                         | 停止加载数据             |

**视图或查询**

| **键**       | **动作**             |
| ------------ | -------------------- |
| CTRL+O       | 加载视图或加载查询   |
| CTRL+/       | 注释行               |
| SHIFT+CTRL+/ | 取消注释行           |
| CTRL+E       | 视图定义或查询编辑器 |
| CTRL+R       | 运行                 |
| SHIFT+CTRL+R | 运行已选择的         |
| F7           | 从这里运行一个语句   |
| CTRL+T       | 停止                 |

**SQL 编辑器**

| **键**                      | **动作**       |
| --------------------------- | -------------- |
| CTRL+F                      | 查找文本       |
| F3                          | 查找下一个文本 |
| CTRL+= 或 CTRL+鼠标滚轮向上 | 放大           |
| CTRL+- 或 CTRL+鼠标滚轮向下 | 缩小           |
| CTRL+0                      | 重设缩放       |

**调试器**

| **键**   | **动作** |
| -------- | -------- |
| F9       | 运行     |
| F8       | 逐过程   |
| F7       | 逐语句   |
| SHIFT+F7 | 跳过     |

**报表**

| **键**    | **动作**             |
| --------- | -------------------- |
| CTRL+O    | 在报表设计中打开报表 |
| CTRL+P    | 在报表设计中打印     |
| CTRL+G    | 在报表设计中组       |
| PAGE DOWN | 下一页               |
| PAGE UP   | 上一页               |
| END       | 第尾页               |
| HOME      | 第一页               |

**模型**

| **键**                      | **动作**                               |
| --------------------------- | -------------------------------------- |
| CTRL+D                      | 在模型中新建图表                       |
| CTRL+P                      | 打印                                   |
| ESC                         | 选择                                   |
| H                           | 移动图表                               |
| T                           | 新建表                                 |
| V                           | 新建视图                               |
| L                           | 新建层                                 |
| A                           | 新建标签                               |
| N                           | 新建笔记                               |
| I                           | 新建图像                               |
| R                           | 新建外键                               |
| CTRL+B                      | 显示已选择的表、视图、外键或形状为粗体 |
| CTRL+= 或 CTRL+鼠标滚轮向上 | 放大                                   |
| CTRL+- 或 CTRL+鼠标滚轮向下 | 缩小                                   |
| CTRL+0                      | 重设缩放                               |

# 在线练习工具:

http://sqlfiddle.com/
