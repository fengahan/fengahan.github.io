---
title: "我来在Linux上装个Mysql"
date: 2019-07-21T17:09:42+09:15
description: "想想上次装MySQL不知道是什么时候了,这种操作 很少用, 毕竟之前都是用服务器镜像搞定的。本地环境呢 为了省事又是lnmp 一键安装包，最近又自己新买了阿里的服务器，闲着没什么用所以打算装个MySQL 用在博客上,之前的笔记不知道去哪了，所再记录一次"
categories: [mysql, linux]
---

废话不多说，说了没用，我这里是新的虚拟机centos 操作系统版本6.8 
资源地址 https://cdn.mysql.com//Downloads/MySQL-5.6/mysql-5.6.41.tar.gz

1.通过yum 安装cmake 和gcc 以及其他工具（如果之前编译过其他程序 就不需要了)
`$ yum install gcc gcc-c++`
`$ yum install cmake  -y `
`$  yum -y install perl perl-devel`
`$ yum install ncurses-devel -y`
			
2.创建数据存储路径:
`mkdir  -p  /data/mysql `

3.添加mysql组和用户
`$ groupadd mysql`
`$ useradd -M mysql -g mysql -s /sbin/nologin  //创建mysql用户并禁止mysql登陆系统`
`$ chown  -R  mysql.mysql  /data/mysql `
`$ mkdir /usr/local/mysql //创建mysql目录` 

4.下载并编译安装
`$ wget  https://cdn.mysql.com//Downloads/MySQL-5.6/mysql-5.6.41.tar.gz`
`$ tar  -zxvf mysql-5.6.41.tar.gz `
`cd  mysql-5.6.41`
`/usr/bin/cmake \`
`-DCMAKE_INSTALL_PREFIX=/usr/local/mysql   \`
`-DMYSQL_DATADIR=/data/mysql   \`
`-DSYSCONFDIR=/etc   \`
`-DWITH_MYISAM_STORAGE_ENGINE=1 \`
`-DWITH_INNOBASE_STORAGE_ENGINE=1 \`
`-DWITH_MEMORY_STORAGE_ENGINE=1 \`
`-DWITH_READLINE=1 \`
`-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \`
`-DMYSQL_TCP_PORT=3306 \`
`-DENABLED_LOCAL_INFILE=1 \`
`-DWITH_PARTITION_STORAGE_ENGINE=1 \`
`-DEXTRA_CHARSETS=all \`
`-DDEFAULT_CHARSET=utf8 \`
`-DDEFAULT_COLLATION=utf8_general_ci`
5.然后执行
`$ make && make install`
这块执行根据机器配置时间有所不同，请耐心等下，顺便喝杯咖啡，不过你一定会听到风扇的声音
完成后我们进行初始化数据库以及配置

6.切换到`/usr/local/mysql `目录 然后执行

7.`cp support-files/my-default.cnf  /etc/my.cnf  //一般新的服务器在etc 下存在这个配置文件我们需要把之前的改为其他名字如 my_bak.cnf`
 
8.切换到`/usr/local/mysql/scripts`  并执行从而初始化数据库
	`./mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql --defaults-file=/etc/my.cnf //初始化数据库`
	
9.//设置环境变量 方便我们在任何工作目录使用mysql 命令
`PATH=$PATH:/usr/local/mysql/bin;export PATH`

10.我们编辑`/etc/my.cnf` 为下， 
#Remove leading # to turn on a very important data integrity option: logging
#changes to the binary log between backups.
#log_bin
#These are commonly set, remove the # and set as required.
 `basedir =/usr/local/mysql`//新增加部分
`datadir =/data/mysql`//新增加部分
`port = 3306`//新增加部分
#server_id = .....
#socket =/tmp/mysql.sock
`[mysqld_safe]`//新增加部分
`user=mysql`//新增加部分
`tmdir=/tmp`//新增加部分
#Remove leading # to set options mainly useful for reporting servers.
#The server defaults are faster for transactions and fast SELECTs.
#Adjust sizes as needed, experiment to find the optimal values.
#join_buffer_size = 128M
#sort_buffer_size = 2M
#read_rnd_buffer_size = 2M
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

11.设置启动 切换到`/usr/local/mysql/support-files`
`cp  mysql.server   /etc/init.d/mysqld`
`service  mysqld start`//启动

12.顺便解决下远程连接
   
	1.vi  /etc/sysconfig/iptables

   增加 -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT //添加防火墙规则（我一般虚拟机的防火墙直接关掉的大家随意）
	2. 控制台登陆mysql 
	3. GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION  //授权
	4. flush privileges;
	5. 选择mysql库下的user表
	6. update user set Password=Password('123456') where user='root';//设置密码
13.大功告成
14.大家需要自行设置开机启动及服务





 






