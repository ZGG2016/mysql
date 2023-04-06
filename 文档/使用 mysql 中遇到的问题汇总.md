# 使用 mysql 中遇到的问题汇总

[TOC]

### 问题1

启动 mysql 服务，失败，出现 `Job for mysqld.service failed because the control process exited with error code.`

`var/lib/mysql` 这个目标路径已经存在，导致无法初始化，删除即可。

```sh
[root@zgg script]# rm -rf /var/lib/mysql
[root@zgg script]# /bin/systemctl start mysqld.service
[root@zgg script]# /bin/systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2020-10-16 23:32:51 CST; 1min 58s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 6930 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 7005 (mysqld)
   Status: "Server is operational"
   CGroup: /system.slice/mysqld.service
           └─7005 /usr/sbin/mysqld

10月 16 23:32:47 zgg systemd[1]: Starting MySQL Server...
10月 16 23:32:51 zgg systemd[1]: Started MySQL Server.
```


### 问题2

启动mysql8.0，报错，出现`ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)`

解决方法：

1、修改MySQL 登入限制

```sh
# 在/etc/my.cnf文件下的[mysqld]的末尾追加上一句：skip-grant-tables 
[root@zgg ~]# cat /etc/my.cnf
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysql]
default-character-set=utf8

[mysqld]
skip-grant-tables 
....
```

2、重新启动MySQL服务

```sh
[root@zgg script]# /bin/systemctl restart mysqld.service      
```

3、登入MySQL，修改密码设置规则，修改密码

```sh
[root@zgg script]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 8.0.21 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> select host,user,plugin,authentication_string from mysql.user;
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| host      | user             | plugin                | authentication_string                                                  |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | root             | caching_sha2_password | $A$005$
                                                                1(z=SER s1(BQmzoDYXKEZzrJWGMJUVajerO272z57C3YQGhvN/5ou. |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
4 rows in set (0.00 sec)

mysql> ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> SHOW VARIABLES LIKE 'validate_password%'; 
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)

mysql> set global validate_password.policy=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password.length=4;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '1234';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

4、再次修改MySQL 登入限制

去除[mysqld] 中的skip-grant-tables ，重新启动MySQL服务。

```sh
[root@zgg script]# cat mysql_start.py 
import os

# python mysql_start.py
# password:1234
start = "/bin/systemctl start mysqld.service && mysql -uroot -p1234"

os.system(start)
   
[root@zgg script]# python mysql_start.py 
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.21 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> 
```

参考：[1](https://blog.csdn.net/zhouzhiwengang/article/details/87378046?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param) [2](https://blog.csdn.net/weixin_42955916/article/details/104670182?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

### 问题3

安装完 mysql，执行 `mysql -u root -p` 后，出现 `mysql: [ERROR] unknown variable 'datadir=/var/lib/mysql'` 错误。

错误跟 `my.cnf` 文件有关，需要这样：

    [mysql]
    default-character-set=utf8

    [mysqld]

而不是：

    [mysqld]

    [mysql]
    default-character-set=utf8

### 问题5

