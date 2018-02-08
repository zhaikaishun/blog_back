---
title: Centos6编译安装Mysql5.7.18, rpm 安装mysql5.7.18，ubuntu apt安装mysql
date: 2017-08-30 21:25:21
tags: [linux]
categories: [linux]
author: kaishun
id: 102
permalink: mysql5_17_18-install
---

关键字： centos 编译安装mysql5.7.18 rpm安装5.7.18,ubuntu apt -get 安装mysql  
![mysql安装](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1494956471263&di=a8e8616a1c50ff82d6b310f24ceb7350&imgtype=jpg&src=http%3A%2F%2Fimg0.imgtn.bdimg.com%2Fit%2Fu%3D1421471404%2C1366921536%26fm%3D214%26gp%3D0.jpg)  
<!-- more -->
# **一. 编译安装前的准备**
## **1.1 卸载原有的Mysql**
**在root用户下操作**  找出mysql的相关目录  
```shell
[root@master hadoop]# find / -name mysql
```
显示  
```shell
[root@master hadoop]# find / -name mysql
/usr/share/mysql
/usr/lib64/mysql
```  
通过命令rm -rf将mysql有关目录都删除掉，例如我这里只有上述的两个目录  
```shell
[root@master hadoop]# rm -rf /usr/share/mysql
[root@master hadoop]# rm -rf /usr/lib64/mysql
```  
## **1.2 下载自己想要的版本的mysql**  
**官网太慢，不建议去下载，建议sohu Mysql的镜像站中下载**  
http://mirrors.sohu.com/mysql/  
例如我要下载5.7的，那么进入 http://mirrors.sohu.com/mysql/MySQL-5.7/  
**Ctrl+F查找到 tar.gz **   ，**找到mysql-boost-5.7.18.tar.gz进行下载**   

为什么我要找mysql-boost-5.7.18.tar.gz  而不是 mysql-5.7.18.tar.gz, 因为在Mysql5.7版本更新后有很多变化，比如json等，连安装都有变化，他安装必须要BOOST库，所以我们下载 mysql-boost-5.7.18.tar.gz, 如果是5.6版本，那么就下载  mysql-5.6.35.tar.gz 就可以了  5.5 版本，就下载 mysql-5.5.54.tar.gz  反正自己去镜像库中找就可以了

## **1.3 把Mysql的压缩包放在/root下**
```shell
[root@master Downloads]# mv mysql-boost-5.7.18.tar.gz /root/
```  
## **1.4 安装编译源码所需的工具和库**  
```shell
[root@master Downloads]# yum install gcc gcc-c++ ncurses-devel perl
```  
## **1.5 安装cmake编译工具**
```shell
[root@master Downloads]# yum install cmake -y
```  
# **二、设置MySQL用户和组**  
## **2.1新增mysql用户组**
```shell
[root@master Downloads]# groupadd mysql
```
## **2.2新增mysql用户**
```shell
[root@master Downloads]# useradd -r -g mysql mysql
```  
# **三、新建MySQL所需要的目录**
## **3.1新建mysql安装目录**  
```shell
[root@master Downloads]# mkdir -p /usr/local/mysql
```
## **3.2新建mysql数据库数据文件目录** 
```shell
[root@master Downloads]# mkdir -p /data/mysqldb
```

## **3.3解压mysql压缩包**  
```shell
//进入到mysql压缩包的目录下 
[root@master Downloads]# cd /root/
//解压 
[root@master ~]# tar -zxvf mysql-boost-5.7.18.tar.gz  
```

## **3.4进入到mysql目录**
```shell
[root@master ~]# cd mysql-5.7.18
```  
# **四、编译安装MySQL**  
## **4.1 设置编译参数**
```shell
[root@master mysql-5.7.18]# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock -DDEFAULT_CHARSET=utf8 \
-DWITH_BOOST=boost -DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 \ 
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DMYSQL_DATADIR=/data/mysqldb \
-DMYSQL_TCP_PORT=3306 -DENABLE_DOWNLOADS=1 \
```    
当最后有以下显示说明成功， 可能上面有一些 Failed 的报错，但是貌似是正常情况，具体情况有待考究
```
-- Configuring done
-- Generating done
-- Build files have been written to: /root/mysql-5.7.18
```
参数说明：  
**在mysql-5.7中，多了一个  -DWITH_BOOST=boost**  
在其他的版本，不需要 -DWITH_BOOST=boost
```
-DCMAKE_INSTALL_PREFIX=dir_name	设置mysql安装目录
-DMYSQL_UNIX_ADDR=file_name	设置监听套接字路径，这必须是一个绝对路径名。默认为/tmp/mysql.sock
-DDEFAULT_CHARSET=charset_name	设置服务器的字符集。
缺省情况下，MySQL使用latin1的（CP1252西欧）字符集。cmake/character_sets.cmake文件包含允许的字符集名称列表。
-DDEFAULT_COLLATION=collation_name	设置服务器的排序规则。
-DWITH_INNOBASE_STORAGE_ENGINE=1 
-DWITH_ARCHIVE_STORAGE_ENGINE=1
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1	存储引擎选项：

MyISAM，MERGE，MEMORY，和CSV引擎是默认编译到服务器中，并不需要明确地安装。

静态编译一个存储引擎到服务器，使用-DWITH_engine_STORAGE_ENGINE= 1

可用的存储引擎值有：ARCHIVE, BLACKHOLE, EXAMPLE, FEDERATED, INNOBASE  
(InnoDB), PARTITION (partitioning support), 和PERFSCHEMA (Performance Schema)
-DMYSQL_DATADIR=dir_name	设置mysql数据库文件目录
-DMYSQL_TCP_PORT=port_num	设置mysql服务器监听端口，默认为3306
-DENABLE_DOWNLOADS=bool	
```
**注**：重新运行配置，需要删除CMakeCache.txt文件  
## **4.2编译源码** 
建议执行完下面的命令，去喝杯茶，休息休息，因为5.7的编译需要好几十分钟（我是看了一集多的权力的游戏才编译完），5.6的可能几分钟就ok了
```shell
[root@master mysql-5.7.18]# make
```
## **4.3 安装**
```shell
[root@master mysql-5.7.18]# make install
```  
# **五、将mysql目录得所有权交给mysql用户和mysql用户组**  
### **5.1修改mysql安装目录**
进入到mysql安装目录(上面已经定义了)
```shell
[root@master mysql-5.7.18]# cd /usr/local/mysql/

```
赋予当前目录下所有文件的所有权为mysql, 注意别忘了后面的 **.** 号
```shell
[root@master mysql]# chown -R mysql:mysql .
```

### **5.2修改mysql数据库文件目录**
```shell
[root@master mysql]# cd /data/mysqldb/  
[root@master mysqldb]# chown -R mysql:mysql .
```
## **六、初始化mysql数据库**
进入到mysql安装目录
```shell
[root@master mysqldb]# cd /usr/local/mysql/
```
初始化mysql数据库
```shell
[root@master mysql]# ./bin/mysql_install_db  --user=mysql --datadir=/data/mysqldb/  
```  
注意： 如果是5.7以下的版本，应该这样 // scripts/mysql_install_db  --user=mysql --datadir=/data/mysqldb/  

即使是5.7 上述执行后还是报错
```shell
2017-04-30 21:57:19 [WARNING] mysql_install_db is deprecated. Please consider switching to mysqld --initialize
2017-04-30 21:57:48 [WARNING] The bootstrap log isn't empty:
2017-04-30 21:57:48 [WARNING] 2017-04-30T13:57:20.320409Z 0 [Warning] --bootstrap is deprecated. Please consider using --initialize instead
2017-04-30T13:57:20.329799Z 0 [Warning] Changed limits: max_open_files: 1024 (requested 5000)
2017-04-30T13:57:20.329842Z 0 [Warning] Changed limits: table_open_cache: 431 (requested 2000)
```  
原因是 之前版本mysql_install_db是在mysql_basedir/script下，5.7放在了mysql_install_db/bin目录下,且已被废弃
应该这样
```shell
[root@master mysql]# bin/mysqld --initialize --user=mysql --datadir=/data/mysqldb/
```  
下面会显示
```shell
2017-04-30T14:28:37.187266Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2017-04-30T14:28:40.862777Z 0 [Warning] InnoDB: New log files created, LSN=45790
2017-04-30T14:28:42.292320Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2017-04-30T14:28:42.589111Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 4f97ab23-2db1-11e7-9084-000c293c28f1.
2017-04-30T14:28:42.591907Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2017-04-30T14:28:42.594730Z 1 [Note] A temporary password is generated for root@localhost: OxxO=A0:lB_T
```  
由上面的提示可以知道，5.7初始化会自动生成一个初始化密码，我这里的密码是OxxO=A0:lB_T,一定要把密码给记住，以后有用  若是5.5， 5.6版本的，初始化的密码是空  
注： 如果需要重新初始化，需要先删除掉数据库data文件夹, 例如我这里
```shell
[root@master mysqldb]# rm -rf /data/mysqldb
```  
# **七、复制mysql服务启动配置文件**  
**5.7和5.6，5.7的不一样，不能像以前一样cp /usr/local/mysql/support-files/my-small.cnf /etc/my.cnf了**。  
**需要进入到/etc/my.cnf*  修改datadir=/data/mysqldb  socket=/usr/local/mysql/mysql.sock
注： /data/mysqldb为前面定义的数据存放目录，/usr/local/mysql/ 是数据安装目录， log-error， pid-file 我暂时没有修改
```shell
[root@master hadoop]# vim /etc/my.cnf  
最终my.cnf如下  
[mysqld]
# datadir=/var/lib/mysql
datadir=/data/mysqldb
#socket=/var/lib/mysql/mysql.sock
socket=/usr/local/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```  
# **八、复制mysql服务启动脚本**
```shell
[root@master hadoop]# cd /usr/local/mysql/
[root@master hadoop]# cp support-files/mysql.server /etc/init.d/mysqld
```  
# **九、启动mysql服务并加入开机自启动**  
启动service  mysqld start， 出现OK说明启动成功
```shell
[root@master mysql]# service  mysqld start
Starting MySQL                                             [  OK  ]
```  
加入开机自启动
```shell
[root@master mysql]# chkconfig --level 35 mysqld on
```  
# **十、检查mysql服务是否启动**  
## **1.  查看端口有没有被占用**
```
[root@master mysql]# netstat -tulnp | grep 3306
tcp        0      0 :::3306                     :::*                        LISTEN      2474/mysqld  
```
## **2. 查看数据库版本** 

```shell
[root@master mysql]# [root@master etc]# mysql --version
有可能出现
bash: mysql: command not found
```  
这是由于系统默认会查找/usr/bin下的命令，如果这个命令不在这个目录下，当然会找不到命令，我们需要做的就是映射一个链接到/usr/bin目录下，相当于建立一个链接文件, /usr/local/mysql是我们的安装目录 
```shell
[root@master mysql]# ln -s /usr/local/mysql/bin/mysql /usr/bin/
```    

## **3. 登陆数据库**  
账号为-uroot, 密码为前面初始化数据库的时候生成的密码
```shell
[root@master etc]# mysql -uroot -pOxxO=A0:lB_T
```  
登陆成功
```shell
[root@master etc]# mysql -uroot -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
[root@master etc]# mysql -uroot -pOxxO=A0:lB_T
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.18

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```  
查看数据库
```
mysql> show databases;
出现You must reset your password using ALTER USER statement before executing this statement.提示需要重置密码
```
重置密码，我这里设置成shun
```
mysql> SET PASSWORD = PASSWORD('shun');
```
退出重新登陆，成功

# **rpm 安装mysql**
编译安装实在是太麻烦了，编译按照的好处在于可以自己定制想要的功能，rpm就方便很多
## 先下载rpm包，同样的去http://mirrors.sohu.com/mysql/  去找，也可以去官方网站下载，大概有400兆，然后解压
```

[root@master hadoop]# wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.18-1.el6.x86_64.rpm-bundle.tar

```

## 删除已有的mysql
```
[root@node01 Downloads]# rpm -qa|grep mysql
mysql-libs-5.1.71-1.el6.x86_64
[root@node01 Downloads]# rpm -e --nodeps mysql-libs-5.1.71-1.el6.x86_64
```
## rpm安装，最好按照我这个顺序来  
这些全部是上面下载的包解压得到的

```

[root@node01 Downloads]# rpm -ivh  mysql-community-common-5.7.18-1.el6.x86_64.rpm
[root@node01 Downloads]# rpm -ivh mysql-community-libs-5.7.18-1.el6.x86_64.rpm
[root@node01 Downloads]# rpm -ivh mysql-community-libs-compat-5.7.18-1.el6.x86_64.rpm
[root@node01 Downloads]# rpm -ivh mysql-community-client-5.7.18-1.el6.x86_64.rpm  
[root@node01 Downloads]# rpm -ivh  mysql-community-server-5.7.18-1.el6.x86_64.rpm
[root@node01 Downloads]# rpm -ivh   mysql-community-devel-5.7.18-1.el6.x86_64.rpm
[root@node01 Downloads]# rpm -ivh  mysql-community-embedded-5.7.18-1.el6.x86_64.rpm
[root@node01 Downloads]# rpm -ivh  mysql-community-embedded-devel-5.7.18-1.el6.x86_64.rpm
[root@node01 Downloads]# rpm -ivh mysql-community-test-5.7.18-1.el6.x86_64.rpm

```
安装过程中如果报下面的错误，说明缺少一些包
```
warning: mysql-community-test-5.7.18-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
error: Failed dependencies:
        perl(JSON) is needed by mysql-community-test-5.7.18-1.el6.x86_64
        perl(Time::HiRes) is needed by mysql-community-test-5.7.18-1.el6.x86_64 
        
```

**yum 安装依赖包**
```
[root@node01 Downloads]# yum install perl -y
[root@node01 Downloads]# yum install perl-JSON.noarch
yum install perl-Time-HiRes
```

## 初始化数据库
```
[root@node01 Downloads]# mysqld --initialize  --user=mysql
如果要自定义目录
 --datadir=你要设置的目录
```  

## 查看生成的密码
```
[root@node01 log]# vim /var/log/mysqld.log 
显示如下，我的用户名是root, 密码是 0a.efyjoekuF
2017-05-15T07:30:10.907363Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2017-05-15T08:11:33.040718Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2017-05-15T08:11:34.346242Z 0 [Warning] InnoDB: New log files created, LSN=45790
2017-05-15T08:11:34.547761Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2017-05-15T08:11:34.760728Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 1c8d4b8e-3946-11e7-a555-000c29ad881f.
2017-05-15T08:11:34.836260Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2017-05-15T08:11:34.838073Z 1 [Note] A temporary password is generated for root@localhost: kwjoAoUmF1!e
```  
如果有问题，想重新初始化数据库，需要删除datadir 的目录(没设置就是默认目录)和/var/lib/mysql 目录

## 启动数据库
```
[root@node01 log]# service  mysqld start
Initializing MySQL database:                               [  OK  ]
Installing validate password plugin:                       [  OK  ]
Starting mysqld:                                           [  OK  ]

``` 


## 查看端口
```
[root@node01 log]# netstat -tulnp | grep 3306
tcp        0      0 :::3306                     :::*                        LISTEN      5255/mysqld
```

## 登录  
```
[root@node01 mysql]# mysql -uroot -p0a.efyjoekuF

```
## 改密码
```
SET PASSWORD = PASSWORD('你的密码');  
```



# **Ubuntu apt 按照mysql5.6, 5.7**  
很久没用ubuntu了，貌似ubuntu的还不支持5.7.18，但是apt 有5.7以上的版本了，ubuntu 安装mysql 非常的简单
参考 http://blog.csdn.net/t1dmzks/article/details/52079791
```shell
sudo apt-get install mysql-5.6
更新软件源 
sudo apt-get update
```  
按照提示一路走即可
**安装Mysql5.7**
```
sudo apt-get install mysql-server-5.7
```

---

> 希望各位都能顺利的安装成mysql  
由于作者水平有限,本文章错漏缺点在所难免,希望读者批评指正。