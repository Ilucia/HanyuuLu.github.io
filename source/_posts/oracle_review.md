---
title: Oracle数据库复习提纲
date: 2020-06-20 00:00:00
tag:
  - Oracle
  - database
---

# Oracle数据库结构 

它由至少一个表空间和数据库模式对象组成。这里，模式是对象的集合，而模式对象是直接引用数据库数据的逻辑结构。模式对象包括这样一些结构:表、视图、序列、存储过程、同义词、索引、簇和数据库链等。逻辑存储结构包括表空间、段和范围，用于描述怎样使用数据库的物理空间。

总之,逻辑结构由逻辑存储结构(表空间,段,范围,块)和逻辑数据结构(表、视图、序列、存储过程、同义词、索引、簇和数据库链等)组成,而其中的模式对象(逻辑数据结构)和关系形成了数据库的关系设计。

段(Segment):

是表空间中一个指定类型的逻辑存储结构，它由一个或多个范围组成，段将占用并增长存储空间。

数据库的物理存储结构是由一些多种物理文件组成，主要有数据文件、控制文件、重做日志文件、归档日志文件、参数文件、口令文件、警告文件等。

控制文件:存储实例、数据文件及日志文件等信息的二进制文件。alter system set control_files='路径'。V$CONTROLFILE。

数据文件:存储数据，以.dbf做后缀。一句话:一个表空间对多个数据文件，一个数据文件只对一个表空间。dba_data_files/v$datafile。

# Oracle 的 oracle sql*plus常用命令

一、sys用户和system用户
Oracle安装会自动的生成sys用户和system用户
(1)、sys用户是超级用户，具有最高权限，具有sysdba角色，有create database的权限，该用户默认的密码是change_on_install
(2)、system用户是管理操作员，权限也很大。具有sysoper角色，没有create database的权限，默认的密码是manager
(3)、一般讲，对数据库维护，使用system用户登录就可以拉
注意：也就是说sys和system这两个用户最大的区别是在于有没有create database的权限。
          
二、sql\*plus工具简介
sql\*plus是oracle自带的工具软件，主要用于执行sql语句，pl\sql块。
操作如下：
(1)、在D:\dev\oracle\product\10.2.0\db_1\bin\目录下的sqlplusw.exe。(D:\dev\oracle\为oracle安装目录)
(2)、在运行栏中输入“sqlplusw”即可

三、sqlplus dos工具简介
(1)、概述：sqlplus是在dos下操作oracle的工具，其功能和sql*plus相似。
(2)、操作如下：在运行栏中输入“sqlplus”即可

四、PLSQL Developer工具，需要自己安装，推荐大家使用

五、sql\*plus常用命令
1)、连接命令
1.conn[ect]
用法：conn 用户名/密码@网络服务名 [as sysdba/sysoper]
说明：当用特权用户身份连接时，必须带上as sysdba或是as sysoper
eg、
SQL> show user
USER 为 "SCOTT"
SQL> conn system/oracle@orcl
已连接。
SQL> show user
USER 为 "SYSTEM"
SQL>
以上命令实现类似切换用户的效果
2.disc/disconn/disconnect
说明: 该命令用来断开与当前数据库的连接
3.pssw[ord]
说明: 该命令用于修改用户的密码，如果要想修改其它用户的密码，需要用sys/system登录。
eg、
SQL> conn scott/oracle
已连接。
SQL> passw
更改 SCOTT 的口令
旧口令:
新口令:
重新键入新口令:
口令已更改
SQL>
4.show user
说明: 显示当前用户名
5.exit
说明: 该命令会断开与数据库的连接，同时会退出sql*plus
5.clear screen
清空屏幕

2)、文件操作命令
1.start和@
说明: 运行sql脚本
案例: sql>@ d:\a.sql或是sql>start d:\a.sql
2.edit
说明: 该命令可以编辑指定的sql脚本
案例: sql>edit d:\a.sql,这样会把d:\a.sql 这个文件打开
3.spool
说明: 该命令可以将sql*plus屏幕上的内容输出到指定文件中去。
案例: sql>spool d:\b.sql并输入sql>spool off
eg、
sql>spool d:\b.sql;
sql>select * from emp;
sql>spool off;

3)、交互式命令
1.&
说明：可以替代变量，而该变量在执行时，需要用户输入。
select * from emp where job='&job'；

4)、显示和设置环境变量
概述：可以用来控制输出的各种格式，set show 如果希望永久的保存相关的设
置，可以去修改glogin.sql 脚本
1.linesize
说明：设置显示行的宽度，默认是80个字符
show linesize
set linesize 90
2.pagesize说明：设置每页显示的行数目，默认是14
用法和linesize 一样
至于其它环境参数的使用也是大同小异