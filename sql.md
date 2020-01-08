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

## 设计规范
###  基础
  * 所有表必须使用Innodb存储引擎
    * 极少数特殊业务需求除外
    * mysql5.6之后的默认存储引擎是Innodb引擎；
    * mysql5.5之前使用Myisam(默认存储引擎)
    * Innodb优点：支持事务，行级锁，更好的恢复性，高并发下性能更好
  * 数据库和表的字符集统一使用UTF-8
    * 如果要存储一些如表情符号的，还需使用UTF-8的拓展字符集
    * 数据库，表，字段字符集一定要统一，统一字符集可以避免由于字符集转换产生的乱码
    * 在mysql中UTF-8字符集，汉字占3字节，ASCII码占1字节
  * 所有表和字段都需要添加注释
  * 从一开始就进行数据库说明文档既数据字典的维护
  * 尽量控制单表数据量大小，
    * 建议控制在500万以内，
    虽然500万并不是mysql的数据库限制，但是会给修改表结构，备份，恢复带来很大困难。
    * 单表可存储数据量大小取决于存储设置和文件系统
    * 想减少单表数据量：
      * 历史数据归档(常见于日志表)，
      * 分库分表(常见于业务表)，
      * 分区表
      TODO:这里说的是水平分割吗?
      建议不要使用mysql分区表，因为分区表在物理上表现为多个文件，在逻辑上表现为一个表。如果一定要分区，请谨慎选择分区键，跨分区查询效率比查询大数据量的单表查询效率更低
      建议采物理分表的方式管理大数据，但是对应用程序的开发要求和复杂度更高
      TODO: 物理分表和分区表都什么区别?
    * 尽量做到冷热数据分离，减少表的宽度(字段数)
      * 减少磁盘IO，保证热数据的内存缓存命中率，更有效的利用缓存，避免读入无用的冷数据
      * 这样的话，就要对表的列进行拆分，将经常使用的列放到一个表中，可以避免过多的关联操作，也可以提高查询性能
    * 禁止在表中建立预留字段
      预留字段很难做到见名知义，预留字段无法确定存储的数据类型，后期如果修改字段类型，会对全表锁定，严重影响数据库的并发性
      * 修改一个字段的成本要远远大于增加一个字段的成本
    * 禁止在数据库中存储图片，文件等二级制数据
      这类数据如果要存，就得使用blog或者text这样的大字段加以存储，会影响数据库的性能
      文件这种通常所占数据容量很大，会在短时间内造成数据库文件的快速增长，而数据库在读取数据时，会进行大量的随机IO操作，如果数据文件过大，IO操作会非常耗时，从而影响数据库性能
      正确做法是将这类数据存储在文件服务器中，而数据库只村存储地址信息
    * 禁止在线上做数据库压力测试
      会对正常业务造成影响，也会产生很多垃圾数据
      建议建立专门的压力测试数据库，进行测试，然后对比测试服务器和线上服务器的硬件环境，评估线上数据库的性能
    * 禁止从开发环境，测试环境直连生产环境数据库
  * 程序连接不同的数据库使用不同的账号，禁止跨库查询
    * 为数据库迁移和分库分表留出余地
    * 降低业务耦合度
    *避免权限过大而产生的安全风险
### 命名规范
* 库命名:26个英文字母+数字;长度<30;
  * 使用小写字母并用下划线表示，mysql对大小写敏感，
  * 太长会在传输时增加网络开销
* 表名命名:26个英文字母+数字;下划线分隔;
  * 临时表必须以tmp_为前缀并以日期为后缀
  * 备份表必须以bak_为前缀并以日期为后缀
* 表字段名命名:
  * 26个英文字母+数字;
  * 下划线分隔;
  * 每个表中必须有自增主键,add_time(默认系统时间)
    TODO: 不确定是否必须有自增主键?
* 表字段类型规范
  * 所有存储相同数据的列名和列类型必须一致，比如user表中的id和order表中的user_id
  * 用尽量少的存储空间来存数一个字段的数据;
  * IP地址最好使用int类型;
  如将ip存储为数字：inet_aton(‘255.255.255.255’) = 4294967295 ,反之， inet_ntoa(4294967295) = ‘255.255.255.255’
  inet_aton/inet_ntoa是mysql的函数;
  其他语言版本可以实现, 或者已经提供类似的函数;inet*函数不仅能正确处理ip地址,并且还对ip地址的合法性做了验证;
    * ipv4刚好4个字节,用unsigned int更适合
    * ipv6(据说是ipv4的4倍),可以用BINARY
    TODO: BINARY是什么数据类型?长度和int比较如何?
  * 固定长度的类型最好使用char,例如：邮编;
  * 能使用tinyint就不要使用smallint,int;
  * 最好给每个字段一个默认值,最好不能为null;
    * 默认为空需要分配空间;
    * 如果可以默认为""的字符串也比默认为null强, 空字符串不需要占用空间;
    * not in ,!=等负向条件下null列返回的是空结果
    * 对于null column，count(null column)是不计入统计结果的
    * null列会占用多一个字节的空间，来表明是否为空
  * 禁止使用字符串来存储日期型数据。
    * 无法使用日期函数计算比较
    * 字符串存储要占更多的内存空间，
    date保存精度到天，格式为：YYYY-MM-DD，如2016-11-07
    datetime和timestamp精度保存到秒，格式为：YYYY-MM-DD HH:MM:SS,如：2016-11-07 10:58:27
    timestamp会跟随设置的时区变化而变化，而datetime保存的是绝对值不会变化。
    timestamp必须有默认值,
    datetime列可能为空;
    datetime(8字节)和timestamp(本身是以int存储，占4字节,范围:1970-01-01 00:00:01到2038-01-19 03:14:07)
    日期类型填写长度没有意义,不是varchar,正确的长度就是为0;
  * 财务相关数据，使用decimal类型 (精准浮点类型，在计算时不丢失精度)
    * 函数完整格式DECIMAL(M,D)
      M是是只的最大精度数位，1-65
      D是小数点右侧数位0-30
      decimal(3)：999
      decimal(3,2)：9.99
      decimal(10,5)11615.23653
      超出精度范围的数会被强制进位并只显示数据类型定义的格式
* 数据库表索引规范
  * 命名简洁明确,例如：user_login表user_name字段的索引应为user_name_index唯一索引;
  * 限制索引数量,单表不超过5个;
    * 索引多了查询效率高(过多也有可能降低)
    * 索引多了插入和更新的效率降低
  * 为每个表创建一个主键索引;
    * innodb是每个表必须有主键;
    * 如果没有主键将选择第一个非空索引作为主键(性能降低)
  * 每个表创建合理的索引;
    * 不使用更新频繁的列作为主键。
    因为Innodb是一种索引索引组织表，如果主键上的值频繁更新，就意味着数据存储的逻辑顺序频繁变动，必然会带来大量的IO操作，降低数据库性能。
    * 不要使用uuid，md5，hash，字符串列作为主键。
    因为这种主键不能保证主键的值是顺序增长的，如果后来的主键值在已有主键值的中间段，那么这个主键插入的时候，** 会将所有主键值大于它的列都向后移**。
    * 建议用mysql自增id建立主键
    最好选择能保证值的顺序为顺序增长的列为主键。并且数据不能重复，
  * 建立复合索引请慎重;
    * 尽量不使用多列联合主键
    * 如何选择索引列的顺序?
      * 从左到右的顺序来使用的
      * 区分度(列中group by的数目和此列总行数的比值趋近于1)最高的列放在联合索引的最左侧
      * 在区分度差不多的情况下，尽量吧字段长度小的放在联合索引的最左侧，
        因为同样的行数，字段小的文件也小，读取时IO性能更优
      * 使用最频繁的列放在联合索引的左侧，这样的话，可以较少地建立索引就能满足需求
  * 要在哪些列上建立索引?
    在select，delete，update的where从句中的列
    包含在order by，group by，distinct字段中的列
    多表join的关联列：mysql对关联操作的处理方式只有一种，那就是嵌套循环的关联方式，所以这种操作的性能对关联列上的索引的依赖性很大
  * 避免建立冗余索引和重复索引
  TODO:冗余索引和重复索引是什么?
  TODO: 主表中的索引是怎样设置和设计的?和主键的区别?
  * 对于频繁的查询优先使用覆盖索引
    就是包含了所有查询字段的索引，这样可以避免Innodb表进行索引的二次查找，并可以把随机IO变为顺序IO提高查询效率
  * 尽量避免使用外键
    mysql和别的数据库不同，会自动在外键上建立索引，会降低数据库的写性能
    建议不使用外键约束，但是一定要在表与表之间的关联键上建立索引，虽然外键是为了保证数据的完整性，但是最好在代码中去保证
  * 普通索引前缀：idx_索引字段名
  * 唯一索引前缀：uk_索引字段名
*数据库范式
  * 第一范式(1NF)：
    * 字段值具有原子性,不能再分(所有关系型数据库系统都满足第一范式);
      例如：姓名字段,其中姓和名是一个整体,如果区分姓和名那么必须设立两个独立字段;
  *  第二范式(2NF)：
    * 一个表必须有主键,即每行数据都能被唯一的区分;
            备注：必须先满足第一范式;
  * 第三范式(3NF)：
    一个表中不能包涵其他相关表中非关键字段的信息,即数据表不能有沉余字段;
    备注：必须先满足第二范式;
    备注：往往我们在设计表中不能遵守第三范式,因为合理的沉余字段将会给我们减少join的查询;
    例如：相册表中会添加图片的点击数字段,在相册图片表中也会添加图片的点击数字段;
### 设计原则
* 核心原则
  * 不在数据库做运算
  * cpu计算务必移至业务层;
    * 减少计算
      * 避免使用函数，将运算转移至易扩展的应用服务器中
        如substr等字符运算，dateadd/datesub等日期运算，abs等数学函数
      * 减少排序，利用索引取得有序数据或避免不必要排序
        如union all代替 union，order by 索引字段等
      *  禁止类型转换，使用合适类型并保证传入参数类型与数据库字段类型绝对一致
        如数字用tiny/int/bigint等，必需转换的在传入数据库之前在应用中转好
      * 简单类型，
        尽量避免复杂类型，降低由于复杂类型带来的附加运算。更小的数据类型占用更少的磁盘、内存、cpu缓存和cpu周期
    * 减少逻辑IO量
      * index，优化索引，减少不必要的表扫描
      如增加索引，调整组合索引字段顺序，去除选择性很差的索引字段等等
      * table，合理拆分，适度冗余
      如将很少使用的大字段拆分到独立表，非常频繁的小字段冗余到“引用表”
      * SQL，调整SQL写法，充分利用现有索引，避免不必要的扫描，排序及其他操作
      如减少复杂join，减少order by，尽量union all，避免子查询等
      * 数据类型，够用就好，减少不必要使用大字段
      TODO: date比timestamp要空间更小吗?
      如tinyint够用就别总是int，int够用也别老bigint，date够用也别总是timestamp
  * 控制列数量(字段少而精,字段数建议在20以内);
  * 平衡范式与冗余(效率优先；往往牺牲范式)
  * 拒绝3B
    * 拒绝大sql语句：big sql、
    * 拒绝大事物：big transaction、
    * 拒绝大批量：big batch;
* 字段类原则
  * 用好数值类型
  (用合适的字段类型节约空间);
  * 字符转化为数字
  (能转化的最好转化,同样节约空间、提高查询性能);
  * 避免使用NULL字段
  (NULL字段很难查询优化、NULL字段的索引需要额外空间、NULL字段的复合索引无效);
  索引null列需要额外的空间来保存，占更多空间
  进行比较和计算时，对null值作特别的处理，可能造成索引失效
  * 少用text类型
  尽量使用varchar代替text字段);
* 索引类原则
  * 合理使用索引
  (改善查询,减慢更新,索引一定不是越多越好);
  * 字符字段必须建前缀索引;
  TODO: 前缀索引??
  * 不在索引做列运算;
  TODO: 什么是在索引做列运算
  * innodb主键推荐使用自增列
  TODO: innodb? 聚簇索引?主键还能被修改?字符串能做主键?
  (主键建立聚簇索引,主键不应该被修改,字符串不应该做主键)
  (理解Innodb的索引保存结构就知道了);
  * 不用外键(由程序保证约束);
  TODO: 什么是外键
* sql类原则
  * sql语句尽可能简单(一条sql只能在一个cpu运算,大语句拆小语句,减少锁时间,一条大sql可以堵死整个库);
  * 简单的事务;
  * 建议使用预编译语句(prepareStatment)进行数据库操作
  TODO: 什么是预编译语句? 怎么编写怎么用???
    * 可以同步执行预编译计划，减少预编译时间
    * 可以有效避免动态sql带来的SQL注入的问题
    * 只传参数，一次解析，多次使用，比传递sql语句更高效
  * 避免数据类型的隐式转换
    * 一般出现在where从句中，会导致索引失效，如：select id,name from user where id = ‘12’;
  * 充分利用已存在的索引
    * 避免使用双%的查询条件，不走索引
    * 一个SQL只能利用到复合索引中的一列进行范围查询
    * 使用left join或not exists来优化not in操作
  * 禁止使用子查询
  TODO: 这里说的子查询我理解的是括号条件中使用了selectXXX等语句, 理解的对吗?
    * 虽然可使sql可读性好，但是缺点远远大于优点
    * 子查询返回的结果集无法使用索引，
    结果集会被存储到一个临时表中，结果集越大性能越低
    * 把子查询优化为join操作，
    但是并不是所有的都可以优化为join，一般情况下，只有当子查询是在in字句中，并且子查询是一个简单的sql(不包含union，group by，order by，limit)才能转换为关联查询
  * 避免使用trig/func
  (触发器、函数不用客户端程序取而代之);
  * 不用select *
  (消耗cpu,io,内存,带宽,这种程序不具有扩展性);
  * 减少同数据库的交互次数
    数据库更适合处理批量操作
    合并多个相同的操作到一起，提高处理效率
  * OR改写为IN
  (or的效率是n级别);
    * in的值不要超过500个
    * in 操作可以有效利用索引
  * OR改写为UNION
  (mysql的索引合并很弱智);
    ```MySQL
    select id from t where phone = ’159′ or name = ‘john’;
    =>
    select id from t where phone=’159′
    union
    select id from t where name=’jonh’
    ```
  * 避免负向%;
  TODO: mysql怎么用%?负向??
  * 慎用count(*)
    TODO: count*有没有替代或者优化方案;
  * limit高效分页
  (limit越大，效率越低);
  * 使用union all替代union(union有去重开销);
  * 少用连接join;
    * join一个表会占一部分内存(join_buffer_size)
    * 会产生临时表操作，影响查询效率
    * mysql最多允许关联61个表，建议不超过5个
  * 使用group by;
  * 请使用同类型比较;
  * 禁止在where从句中对列进行函数转换和计算
  TODO: 我之前用year(date)的计算方式有问题吗?
    * 导致无法使用相关列上的索引
    TODO: 如何导致无法使用索引的???
    * where date(create_time)=’20170901’ 写成 where create_time >= ‘20170901’ and create_time < ‘20170902’
  * 打散批量更新;
    TODO:?如何打散
  * 禁止使用order by rand()进行随机排序
    会把表中所有符合条件的数据装载到内存中进行排序
    会消耗大量的cpu和io及内存资源
    推荐在程序中获取随机值
* 性能分析工具
TODO: 这些都是什么工具?
  * show profile;
  * mysqlsla;
  * mysqldumpslow;
  * explain;
  * show slow log;
  * show processlist;
## mysql 数据类型长度
int(M)这样的描述并不代表存储的宽度,int(3)和int(11)能够存储的数据是一样的;
M的作用在于提示开发者长度
只有联合zerofill参数才能有意义，否则int（3）和int（11）没有任何区别。
当使用zerofill 时，默认会自动加unsigned（无符号），zerofill默认为int(10)
但int(3)和int(100)都是只能存储4个字节32位,指定宽度作用仅在于设置了zerofilll的时候进行补0;

* 字符串
  * char(n):
  n 字节长度
  char(M)类型的数据列里，每个值都占用M个字节，如果某个长度小于M，MySQL就会在它的右边用空格字符补足．（在检索操作中那些填补出来的空格字符将被去掉）
  * varchar(n):
  如果是 utf8 编码, 则是 3 n + 2字节; 如果是 utf8mb4 编码, 则是 4 n + 2 字节.
  varchar(M)类型的数据列里，每个值只占用刚好够用的字节再加上一个用来记录其长度的字节（即总长度为L+1字节）
  但创建临时表或者排序时,会悲观地给varchar类型分配最大的长度;
* 数值类型:
  * TINYINT: 1字节
  boolean类型在mysql就是tinyint(1);mysql有4个常量:true,false,TRUE,FALSE分别代表1,0,1,0。

  * SMALLINT: 2字节
  * MEDIUMINT: 3字节
  * INT: 4字节
  * BIGINT: 8字节
  数值类型是2的位数次方;eg,8字节*8(1字节=8位)=64, 可存范围是-$2^{N-1}$~$2^{N-1}$, N=64,如果设为unsign类型表示正数,可以使存储的上限变成$2^N-1$,将近扩大了一半。
  int(3)和int(100)都是只能存储4个字节32位,指定宽度作用仅在于设置了zerofilll的时候进行补0;
  * float和double
   实数精度要求不高情况，优先使用浮点类型float，double,如果精度要求高可考虑用整型转化或直接使用decimal类型
* 时间类型
  * DATE: 3字节
  * TIMESTAMP: 4字节
  * DATETIME: 8字节

* 字段属性: NULL 属性 占用一个字节.
  如果一个字段是 NOT NULL 的, 则没有此属性.
## 操作行为规范
* 主要面向手动操作数据库的行为
  * 超过100万的批量写操作，要分批多次进行操作
    * 主从复制中：
    大批量操作可能会造成严重的主从延迟，因为当主库执行完成后，才会在从库执行
    * binlog日志为row格式时会产生大量的日志
    TODO: 什么是binlog??
    * 避免产生大量事务，产生阻塞，占满可用连接
  * 对大表数据结构的修改一定要谨慎
    * 可能会造成严重的锁表操作，尤其是生产环境，是不能忍受的
    * 对于大表使用pt-online-schema-change修改表结构：
    TODO: 什么是pt-online-schema-change
      * 首先会建立一个与原表结构相同的新表
      * 然后在新表上进行表结构的修改
      * 然后把原表中的数据复制到新表中，并且增加一些触发器，以便把原表中即时新增的数据也复制到新表中
      * 在行的所有数据复制完成之后，会在原表上增加一个很准的时间锁，同时把新表命名为原表，把原表删掉
      [实际上是把一个原子的DDL操作分解成多批次进行]
      [避免大表修改产生的主从延迟问题]
      [避免在对表字段进行修改时进行锁表]
    * 禁止为程序使用的账号赋予super权限
      * 当数据库连接数达到最大限制时，允许1个有super权限的用户连接
      * super权限只能留给DBA处理问题的账号使用
    * 对于程序连接数据库账号，遵循权限最小原则
      * 程序使用的数据库账号只能在一个DB下使用，不准跨库
      * 程序使用的账号原则上不准有drop权限
## JDBC与mysql的映射
### 日期:
LocalTime 对应 **time **只包括时间
LocalDate 对应 **date ** 只包括日期
LocalDateTime 对应 **timestamp datetime **包括日期和时间

## 打印列Comment

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



# 数据库设计原则
  ## 原始单据与实体之间的关系
    * 一般是一对一的关系
  　　可以是一对一、一对多、多对多的关系。在一般情况下，它们是一对一的关系：即一张原始单据对应且只对应一个实体。
  在特殊情况下，它们可能是一对多或多对一的关系，即一张原始单证对应多个实体，或多张原始单证对应一个实体。
  这里的实体可以理解为基本表。明确这种对应关系后，对我们设计录入界面大有好处。

  　　〖例1〗：一份员工履历资料，在人力资源信息系统中，就对应三个基本表：员工基本情况表、社会关系表、工作简历表。
  　　　　　　  这就是“一张原始单证对应多个实体”的典型例子。

## 主键与外键
TODO: 主键与外键区别?
一般而言，一个实体不能既无主键又无外键。
在E—R 图中, 处于叶子部位的实体, 可以定义主键，也可以不定义主键(因为它无子孙), 但必须要有外键(因为它有父亲)。
主键与外键的设计，在全局数据库的设计中，占有重要地位。当全局数据库的设计完成以后，有个美国数据库设计专家说：“键，到处都是键，除了键之外，什么也没有”，这就是他的数据库设计经验之谈，也反映了他对信息系统核心(数据模型)的高度抽象思想。因为：主键是实体的高度抽象，主键与外键的配对，表示实体之间的连接。
## 基本表的性质
TODO: 基本表?中间表、临时表?
　　基本表与中间表、临时表不同，因为它具有如下四个特性：
　　 (1) 原子性。基本表中的字段是不可再分解的。
　　 (2) 原始性。基本表中的记录是原始数据（基础数据）的记录。
　　 (3) 演绎性。由基本表与代码表中的数据，可以派生出所有的输出数据。
　　 (4) 稳定性。基本表的结构是相对稳定的，表中的记录是要长期保存的。
　　理解基本表的性质后，在设计数据库时，就能将基本表与中间表、临时表区分开来。
## 范式标准
　　基本表及其字段之间的关系, 应尽量满足第三范式。但是，满足第三范式的数据库设计，往往不是最好的设计。
　　为了提高数据库的运行效率，常常需要降低范式标准：适当增加冗余，达到以空间换时间的目的。
eg:有一张存放商品的基本表，如表1所示。“金额”这个字段的存在，表明该表的设计不满足第三范式，
　　因为“金额”可以由“单价”乘以“数量”得到，说明“金额”是冗余字段。但是，增加“金额”这个冗余字段，
　　可以提高查询统计的速度，这就是以空间换时间的作法。
## 通俗地理解三个范式
  * 第一范式：1NF是对属性的原子性约束，要求属性具有原子性，不可再分解；
　* 第二范式：2NF是对记录的惟一性约束，要求记录有惟一标识，即实体的惟一性；
  * 第三范式：3NF是对字段冗余性的约束，即任何字段不能由其他字段派生出来，它要求字段没有冗余。
  有时为了提高运行效率，就必须降低范式标准，适当保留冗余数据。具体做法是：在概念数据模型设计时遵守第三范式，降低范式标准的工作放到物理数据模型设计时考虑。降低范式就是增加字段，允许冗余。
  * 两个实体多对多的关系要消除
消除的办法是，在两者之间增加第三个实体。这样，原来一个多对多的关系，现在变为两个一对多的关系。要将原来两个实体的属性合理地分配到三个实体中去。这里的第三个实体，实质上是一个较复杂的关系，它对应一张基本表。一般来讲，数据库设计工具不能识别多对多的关系，但能处理多对多的关系。
　　〖例3〗：在“图书馆信息系统”中，“图书”是一个实体，“读者”也是一个实体。这两个实体之间的关系，是一个典型的多对多关系：一本图书在不同时间可以被多个读者借阅，一个读者又可以借多本图书。为此，要在二者之增加第三个实体，该实体取名为“借还书”，它的属性为：借还时间、借还标志(0表示借书，1表示还书)，另外，它还应该有两个外键(“图书”的主键，“读者”的主键)，使它能与“图书”和“读者”连接。
## 主键PK的取值方法
　　 PK是供程序员使用的表间连接工具，可以是一无物理意义的数字串, 由程序自动加1来实现。也可以是有物理意义的字段名或字段名的组合。不过前者比后者好。当PK是字段名的组合时，建议字段的个数不要太多，多了不但索引占用空间大，而且速度也慢。
## 数据冗余
  * 主键与外键在多表中的重复出现, 不属于数据冗余;
  * 非键字段的重复出现, 才是数据冗余！
  而且是一种低级冗余，即重复性的冗余。
  * 高级冗余不是字段的重复出现，而是字段的派生出现。
　 〖例4〗：商品中的“单价、数量、金额”三个字段，“金额”就是由“单价”乘以“数量”派生出来的, 是一种高级冗余。
  冗余的目的是为了提高处理速度。只有低级冗余才会增加数据的不一致性，因为同一数据，可能从不同时间、地点、角色上多次录入。因此，我们提倡高级冗余(派生性冗余)，反对低级冗余(重复性冗余)。
## E--R图没有标准答案
　* 好的E—R图的标准是：
    * 结构清晰、
    * 关联简洁、
    * 实体个数适中、
    * 属性分配合理、
    * 没有低级冗余。
## 视图技术
  TODO: 视图设计办法, navicat操作过程;
　与基本表、代码表、中间表不同，视图是一种虚表，它依赖数据源的实表而存在。
  视图是供程序员使用数据库的一个窗口，是基表数据综合的一种形式, 是数据处理的一种方法，是用户数据保密的一种手段。为了进行复杂处理、提高运算速度和节省存储空间, 视图的定义深度一般**不得超过三层**。
  TODO: 临时表?作用
  若三层视图仍不够用, 则应在视图上定义临时表,在临时表上再定义视图。这样反复交迭定义, 视图的深度就不受限制了。
  对于某些与国家政治、经济、技术、军事和安全利益有关的信息系统，视图的作用更加重要。这些系统的基本表完成物理设计之后，立即在基本表上建立第一层视图，这层视图的个数和结构，与基本表的个数和结构是完全相同。并且规定，所有的程序员，一律只准在视图上操作。只有数据库管理员，带着多个人员共同掌握的“安全钥匙”，　　才能直接在基本表上操作。请读者想想：这是为什么？
  TODO: 为什么??
## 中间表、报表和临时表
  TODO: 报表?
  * 中间表是存放统计数据的表，它是为数据仓库、输出报表或查询结果而设计的，有时它没有主键与外键(数据仓库除外)。
  * 临时表是程序员个人设计的，存放临时记录，为个人所用。
  TODO: dba怎么维护数据库?
  * 基表和中间表由DBA维护，临时表由程序员自己用程序自动维护。
## 完整性约束表现在三个方面
  TODO: 三个完整性约束怎么实现?
  TODO: 域完整性的Check怎么实现?
　* 域的完整性：用Check来实现约束，在数据库设计工具中，对字段的取值范围进行定义时，有一个Check按钮，通过它定义字段的值城。
　* 参照完整性：用PK、FK、表级触发器来实现。
  * 用户定义完整性：它是一些业务规则，用存储过程和触发器来实现。
## 防止数据库设计打补丁的方法是“三少原则”
  * 一个数据库中表的个数越少越好。
  只有表的个数少了，才能说明系统的E--R图少而精，去掉了重复的多余的实体，形成了对客观世界的高度抽象，进行了系统的数据集成，防止了打补丁式的设计；
  * 一个表中组合主键的字段个数越少越好。
  因为主键的作用，一是建主键索引，二是做为子表的外键，所以组合主键的字段个数少了，不仅节省了运行时间，而且节省了索引存储空间；
  * 一个表中的字段个数越少越好。
  只有字段的个数少了，才能说明在系统中不存在数据重复，且很少有数据冗余，更重要的是督促读者学会“列变行”，这样就防止了将子表中的字段拉入到主表中去，在主表中留下许多空余的字段。所谓“列变行”，就是将主表中的一部分内容拉出去，另外单独建一个子表。这个方法很简单，有的人就是不习惯、不采纳、不执行。
## 数据库设计的实用原则是：在数据冗余和处理速度之间找到合适的平衡点。
  “三少”是一个整体概念，综合观点，不能孤立某一个原则。该原则是相对的，不是绝对的。“三多”原则肯定是错误的。试想：若覆盖系统同样的功能，一百个实体(共一千个属性) 的E--R图，肯定比二百个实体(共二千个属性) 的E--R图，要好得多。
  提倡“三少”原则，是叫读者学会利用数据库设计技术进行系统的数据集成。数据集成的步骤是将文件系统集成为应用数据库，将应用数据库集成为主题数据库，将主题数据库集成为全局综合数据库。集成的程度越高，数据共享性就越强，信息孤岛现象就越少，整个企业信息系统的全局E—R图中实体的个数、主键的个数、属性的个数就会越少。
  提倡“三少”原则的目的，是防止读者利用打补丁技术，不断地对数据库进行增删改，使企业数据库变成了随意设计数据库表的“垃圾堆”，或数据库表的“大杂院”，最后造成数据库中的基本表、代码表、中间表、临时表杂乱无章，不计其数（即动态创表而增加表数量），导致企事业单位的信息系统无法维护而瘫痪。
　 “三多”原则任何人都可以做到，该原则是“打补丁方法”设计数据库的歪理学说。“三少”原则是少而精的原则，它要求有较高的数据库设计技巧与艺术，不是任何人都能做到的，因为该原则是杜绝用“打补丁方法”设计数据库的理论依据。
## 提高数据库运行效率的办法
  在给定的系统硬件和系统软件条件下，提高数据库系统的运行效率的办法是：
  * 在数据库物理设计时，降低范式，增加冗余, 少用触发器, 多用存储过程。
  * 当计算非常复杂、而且记录条数非常巨大时(例如一千万条)，复杂计算要先在数据库外面，以文件系统方式用C++语言计算处理完成之后，最后才入库追加到表中去。这是电信计费系统设计的经验。
  * 发现某个表的记录太多，例如超过一千万条，则要对该表进行水平分割。
    * 水平分割:
    以该表主键PK的某个值为界线，将该表的记录水平分割为两个表（即可以表维护 表行数过大 手动分割为两个  建个两表union的视图对程序透明）。若发现某个表的字段太多，例如超过八十个，则
    * 垂直分割:
    将原来的一个表分解为两个表。
  * 对数据库管理系统DBMS进行系统优化，即优化各种系统参数，如缓冲区个数。
  * 在使用面向数据的SQL语言进行程序设计时，尽量采取优化算法。
  TODO: 优化算法是什么?
  总之，要提高数据库的运行效率，必须从数据库系统级优化、数据库设计级优化、程序实现级优化，这三个层次上同时下功夫。
## denormalization, 逆规范化
辞典:反向规格化, 阻碍正常化
就是我们通常所说的逆规范化.
比如在一个表里设置两个主键.
比如在两个表之间的关系为多对多关系.
等等都是违反标准范式的
无论是规范化还是逆规范化都是为了提高数据库性能.
初学者还是尽量做到规范化的好.

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
