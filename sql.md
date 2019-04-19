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

