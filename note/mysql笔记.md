mysql笔记
## 查看事务隔离级别
show variables like '%iso%';

	+---------------+-----------------+
	| Variable_name | Value           |
	+---------------+-----------------+
	| tx_isolation  | REPEATABLE-READ |
	+---------------+-----------------+

## 设置事务级别
	set seesion tx_isolation='read-committed'

## mysql事务
	begin;
	sql
	commit;

## 建表时指定存储引擎
	create database test default character set utf8 collate utf8_general_ci;
	use test;
	create table myIsam(id int,c1 varchar(10)) engine=myisam;

## innodb
1. Innodb使用表空间进行 数据存储

	innodb_file_per_table

	ON:独立表空间：tablename.ibd

	OFF:系统表空间：ibdataX

	show variables like 'innodb_file_per_table';

		+-----------------------+-------+
		| Variable_name         | Value |
		+-----------------------+-------+
		| innodb_file_per_table | ON    |
		+-----------------------+-------+	

	set global innodb_file_per_table=off;

	独立表空间，可以通过optimize table命令收缩系统文件。

2.  系统表空间到独立表空间的转移

	1）使用mysqldump导出所有数据库表数据

	2）停止MySQL服务，修改参数，并删除Innodb相关文件

	3）重启MySQL服务，重建Innodb系统表空间

	4）重新导入数据

3. log缓冲区大小(1秒刷一次到磁盘)

	show variables like 'innodb_log_buffer_size';

		+------------------------+----------+
		| Variable_name          | Value    |
		+------------------------+----------+
		| innodb_log_buffer_size | 16777216 |
		+------------------------+----------+

日志文件个数
show variables like 'innodb_log_files_in_group';

	+---------------------------+-------+
	| Variable_name             | Value |
	+---------------------------+-------+
	| innodb_log_files_in_group | 2     |
	+---------------------------+-------+
 
4. 查看表的定义
show create table tablename;
show create table myInnodb;

5. 锁定表
lock table myInnodb write;
unlock tables;

6.Innodb状态检查
pager more
show engine innodb status;

mysql5.7版本以后，Innodb支持全文索引和空间函数

## CSV存储引擎
#### 特点
	* 以CSV格式进行数据存储
	* 所有列必须都是不能为NULL的
	* 不支持索引
	* 可以对数据文件直接编辑

#### 存储说明 
	* .CSV存储表内容
	* .CSM存储元数据，如表状态和数据量
	* .frm存储表结构信息

#### 编辑完文件，要 flush tables;

#### demo
	create table mycsv(id int not null,c1 char(10) not null) engine=csv;
	insert into mycsv values(1,"aa"),(2,"bb");
	sudo vim /var/lib/mysql/test/mycsv.CSV
	flush tables;
	select * from mycsv;

#### 使用场景
数据交换

## Archive引擎
#### 特点
* 以zlib对表数据进行压缩，磁盘I/O更少
* 数据存储在ARZ为后缀的文件中
* 只能对自增长ID列建索引
* 只支持insert和select操作

#### demo
	create table myarchive(id int auto_increment not null,c1 varchar(10), c2 char(10), key(id)) engine=archive;

#### 使用场景
日志和数据采集类应用

## Memory引擎
####特点
* 也称HEAP存储引擎，所有数据保存在内存中，表结构存储在磁盘
* 支持HASH索引（对等值查询支持好，对范围查询支持不好，默认索引）和Btree索引
* 所有字段都为固定长度varchar(10)=char(10)
* 不支持BLOG和TEXT等大字段
* Memory存储引擎使用表级锁
* 表大小由max_heap_table_size参数决定，默认为16M，该参数对已经存在的表无效

#### demo
	create table mymemory(id int,c1 varchar(10), c2 char(10)) engine=memory;
	create index idx_c1 on mymemory(c1);
	create index idx_c2 using btree on mymemory(c2);

#### 查看索引语句
mysql> show index from mymemory\G

	*************************** 1. row ***************************
	        Table: mymemory
	   Non_unique: 1
	     Key_name: idx_c1
	 Seq_in_index: 1
	  Column_name: c1
	    Collation: NULL
	  Cardinality: 0
	     Sub_part: NULL
	       Packed: NULL
	         Null: YES
	   Index_type: HASH
	      Comment: 
	Index_comment: 
	*************************** 2. row ***************************
	        Table: mymemory
	   Non_unique: 1
	     Key_name: idx_c2
	 Seq_in_index: 1
	  Column_name: c2
	    Collation: A
	  Cardinality: NULL
	     Sub_part: NULL
	       Packed: NULL
	         Null: YES
	   Index_type: BTREE
	      Comment: 
	Index_comment: 
	2 rows in set (0.00 sec)

#### 查看表状态
mysql> show table status like 'mymemory'\G

	*************************** 1. row ***************************
	           Name: mymemory
	         Engine: MEMORY
	        Version: 10
	     Row_format: Fixed
	           Rows: 0
	 Avg_row_length: 66
	    Data_length: 0
	Max_data_length: 6964122
	   Index_length: 0
	      Data_free: 0
	 Auto_increment: NULL
	    Create_time: 2018-04-30 18:11:11
	    Update_time: NULL
	     Check_time: NULL
	      Collation: utf8_general_ci
	       Checksum: NULL
	 Create_options: 
	        Comment: 
	1 row in set (0.00 sec)

#### 使用场景
* 用于查找或者是映射表，例如邮编和地区的对应表
* 用于保存数据分析中产生的中间表
* 用缓存周期性聚合数据的结果表

注意： Memory数据易丢失，所以要求数据可再生

## 显示所有引擎
show engines;

## mysql重新启动(ubuntu)
	启动方式
	1、使用 service 启动：
	[root@localhost /]# service mysqld start (5.0版本是mysqld)
	[root@szxdb etc]# service mysql start (5.5.7版本是mysql)
	2、使用 mysqld 脚本启动：
	/etc/inint.d/mysqld start
	3、使用 safe_mysqld 启动：
	safe_mysqld&
	b、停止
	1、使用 service 启动：
	service mysqld stop
	2、使用 mysqld 脚本启动：
	/etc/inint.d/mysqld stop
	3、mysqladmin shutdown
	c、重启
	1、使用 service 启动：
	service mysqld restart 
	service mysql restart (5.5.7版本命令)
	2、使用 mysqld 脚本启动：
	/etc/init.d/mysqld restart


## mysql服务器参数
* 命令行参数 
	mysql_safe --datadir=/data/sql_data
* 配置文件
	mysqld --help --verbose | grep -A 1 ‘Default options'

## 参数分全局和session
	show variables where variable_name='wait_timeout' or variable_name='interactive_timeout';
	上面两个值要一起修改，不然取大的
*	set global 参数名=参数值;
	set @@global.参数名=参数值;
*	set [session] 参数名=参数值;
	set @@session. 参数名=参数值;

## 临时表
1. 系统使用临时表（如查询时）
2. create temporary table 建立的临时表

# MySQL服务器参数
## 内存
#### Innodb_buffer_pool_size (缓存池内存)
	总内存=每个线程所需要的内存*连接数）-系统保留内存
#### key_buffer_size (myisam索引内存)
	select sum(index_length) from information_schema.tables where engine='myisam';
	+-------------------+
	| sum(index_length) |
	+-------------------+
	|             45056 |
	+-------------------+
	1 row in set (0.17 sec)


# 测试工具
## 系统整体测试
 apache ab
 http load

## MySQL测试工具
#### mysqlslap 5.1后随MySQL自带
1. 特点：
	* 可以模拟服务器负载，并输出相关统计信息
	* 可以指定也可以自动生成查询语句

2. 常用参数 

```
	--auto-generate-sql 
	--auto-generate-sql-add-autoincrement
	--auto-generate-sql-load-type 	查询类型
	--auto-generate-sql-write-number 	指定初始化数据时生成的数据量
	--concurrency 	指定并发线程数量
	--engines 	指定测试表的存储引擎，可以逗号分割多个存储引擎
	--no-drop	指定不清理测试数据
	--itreations	指定测试运行的次数，和上一个参数冲突
	--number-of-queries	指定每一个线程执行的查询数量
	--debug-info	指定输出额外的内存及CPU统计信息
	--number-int-cols	指定测试表中怨念的INTo类型列的数量
	--number-char-cols	指定测试表中包含的varchar类型的数量
	--create-schema 指定了用于执行测试的数据库的名字
	--query 	用于指定自定义SQL的脚本
	--only-print 并不运行测试脚本，而是把生成的脚本打印出来
```

3. demo

	mysqlslap --concurrency=1,50,100,200 --iterations=3 --number-int-cols=5 --number-char-cols=5 --auto-generate-sql --auto-generate-sql-add-autoincrement --engine=myisam,innodb --number-of-queries=10 --create-schema=sbtest -uroot -p

	mysqlslap: Error when connecting to server: 1040 Too many connections
	max_connections默认值为100

	--only-print 显示实际执行的脚本

#### sysbench
	sysbench --test=cpu --cpu-max-prime=10000 run
	sysbench --test=fileio --file-total-size=1G prepare
	sysbench --test=fileio help
	sysbench --test=fileio --threads=8 --file-total-size=1G --file-test-mode=rndrw --report-interval=1 run

	//something error
	sysbench --test=/home/liuji/share/sysbench-0.5/sysbench/tests/db/oltp.lua --mysql-table-engine=innodb --oltp-table-size=10000 --mysql-db=testdb --mysql-user=root --mysql-password=ji1028 --oltp-tables-10 --mysql-socket=/usr/mysql/data/mysql.sock  prepare

	/var/run/mysqld/mysqld.sock
	先prepare准备数据，再run

## MySQL数据类型
#### 时间
* datetime 时区无关，8个字节
* timestamp 时区相关，4个字节，int存储

#### demo
	set time_zone='-10:00';
	create table time(d1 datetime,d2 timestamp);
	insert into time(d1,d2) values(now(),now());
	select * from time;
	set time_zone='+10:00';
	show variables where variable_name='time_zone';

	存储微秒
	alter table time modify d1 datetime(6);
	alter table time modify d2 timestamp(6);
	select * from time;

	drop table time;
	created table time(id int,d1 )


## 二进制日志
#### 基于段的二进制日志 statement		row 	mixed
* 记录sql语句
* uuid()不确定函数，会复制出错
开启二进制日志
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
log_bin=mysql-bin
server_id=1

show variables like 'log_bin';


	show variables like 'binlog_format';

		+---------------+-------+
		| Variable_name | Value |
		+---------------+-------+
		| binlog_format | ROW   |
		+---------------+-------+
		1 row in set (0.00 sec)	

		set session binlog_format=statement;
		show binary logs;
		flush logs;

	show variables like 'binlog_row_image';
	full 全记录
	minimal 只记录修改的列(建议使用)
	noblob 不记录blob(text)列

## 配置mysql复制
1. 基于日志点的复制配置步骤
	create user 'repl'@'ip段' identified by 'password';
	grant replication slave on *.* to 'repl'@'ip段'

2. 配置主数据库服务器
	bin_log = mysql-bin
	server_id = 100 #可以用set命令动态修改，在复制集群中唯一,可以用ip后几位

3. 配置从数据库服务器
	bin_log = mysql-bin
	server_id = 101
	relay_log = mysql-relay-bin
	log_slave_update = on [可选]
	read_only = on [可选]

4. 从主数据库初始化到从数据库(2种)
	mysqldump --master-data=2 -single-transaction
	xtrabackup --slave-info

5. 启动复制链路(在从服务器上操作)
	change master to master_host = 'master_host_id',
	master_user='repl',
	master_password='password',
	master_log_file='mysql_log_file_name',
	master_log_pos=4;

#### demo
	create user repl@'192.168.3.%' identified by 'pass';
	create user 'repl'@'*' identified by 'pass';
	grant replication slave on *.* to 'repl'@'*';

	bin_log = mysql-bin
	server_id = 1

	如果主从mysql版本不一致，不要备份系统数据库
	mysqldump --single-transaction --master-data --triggers --routines --all-databases -uroot -p >> /home/liuji/all.sql

	mysqldump -h192.168.43.160 --single-transaction --master-data --triggers --routines --databases exhibition test -uroot -p >> /home/liuji/all.sql
	

	mysql -uroot -p < all.sql

	change master to master_host='192.168.3.100',
	master_user='repl',
	master_password='pass',
	master_port=3307,
	#从all.sql中找
	#CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=154;
	MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=154;

	start slave;

	主服务器
	mysql> show processlist \G;
	Binlog Dump 线程

	从服务器2个进程
	mysql> show processlist \G;

#### 基于GTID的复制，是从mysql 5.6开始支持的复制方式
1. 基于日志点的复制配置步骤
	create user 'repl'@'ip段' identified by 'password';
	grant replication slave on *.* to 'repl'@'ip段'
2. 配置主数据库服务器
	bin_log = mysql-bin
	server_id = 100
	gtid_mode=on
	enforce-gtid-consistency=on
	log-slave-updates = on #5.7不需要，之前版本需要

3. 配置从数据库服务器
	server_id = 101
	relay_log = /usr/local/mysql/log/relay_log
	gtid_mode = on
	enforce-gtid-consistency=on
	log-slave-updates = on
	read_only=on [建议]
	master_info_repository=TABLE [建议]
	relay_log_info_repository=TABLE [建议]

4. 初始化从服务器数据
	mysqldump --master-data=2 -single-transaction
	xtrabackup --slave-info

5. 启动基于GTID的复制
	change master to master_host = 'master_host_id',
	master_user='repl',
	master_password='password',
	master_auto_position = 1

#### 查看数据库用户
	mysql> use mysql
	Database changed
	mysql> select user,host from user;

	查看授权
	show grants for repl@'192.168.3.%'

## 多线程复制配置（5.7版本开始有的功能）
	show processlist;
	stop slave;
	show slave status\G;
	mysql> show variables like 'slave_parallel_type';
	+---------------------+----------+
	| Variable_name       | Value    |
	+---------------------+----------+
	| slave_parallel_type | DATABASE |
	+---------------------+----------+
	1 row in set (0.00 sec)
	set global slave_parallel_type='logical_clock';
	show variables like 'slave_parallel_workers';
	set global slave_parallel_workers=4;
	start slave;
	show slave status \G;


## MMM配置
	查看用户
	select user,host from mysql.user;

	show master status \G

	wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
	wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

	rpm -ivh epel-release-latest-6.noarch.rpm
	rpm -ivh remi-release-6.rpm

	vim /etc/yum.repos.d/remi.repo
	enabled=1
	vim /etc/yum.repos.d/epel.repo
	#baseurl 的#去掉
	mirrorlist 前加上#

	搜索可以安装的包
	sudo yum search mmm
	sudo apt search mmm

	yum install mysql-mmm-agent.noarch -y

  	监控的要安装所有的
  	yum -y install mysql-mmm*

  	ip addr 也可以查看ip

## MHA (Master High Availability
	select user,host from mysql.user;
	show grants for 'repl'@'*';
	grant all privileges on *.* to mha@'192.168.3.%' identified by '123456';
	show variables like '%log%';

## 读写分离插件 MaxScale

## 索引
演示数据库
http://downloads.mysql.com/docs/sakila-db.tar.gz

查找重复索引
pt-duplicate-key-checker h=127.0.0.1

查找未被使用过的索引
select object_schema,object_name,index_name,b.TABLE_ROWS
from performance_schema.table_io_waits_summary_by_index_usage a
join information_schema.tables b on
	a.object_schema=b.TABLE_SCHEMA AND
	a.OBJECT_NAME=b.TABLE_NAME
WHERE index_name IS NOT NULL
AND count_star=0
ORDER BY object_schema,object_name;


慢查询日志：
动态开启
set global slow_query_log=on;

