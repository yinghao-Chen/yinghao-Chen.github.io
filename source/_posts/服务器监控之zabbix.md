---
title: 服务器监控之zabbix
author: 不好听
img: /2018/11/17/fu-wu-qi-jian-kong-zhi-zabbix/images/timg.jpg
categories: zabbix
tags:
  - zabbix
  - 监控
date: 2018-11-17 18:16:50
toc: true
---

前段时间有监控服务器主机、应用、交换机等的需求，所以对zabbix进行了学习，现在整理记录全过程。
	
## 1.zabbix概述

Zabbix由以下几个组件构成：
1.1 zabbix_server: Zabbix server 是agent程序报告系统可用性、系统完整性和统计数据的核心组件，是所有配置信息、统计信息和操作数据的核心存储器。
1.2 Web界面:由php编写，是Zabbix Server的一部分，通常(使用SQLite需要，其他不是必须)跟Zabbix Server运行在同一台物理机器上。
1.3 Proxy代理服务器
Zabbix proxy 可以替Zabbix Server收集性能和可用性数据。Proxy代理服务器是Zabbix软件可选择部署的一部分；当然，Proxy代理服务器可以帮助单台Zabbix Server分担负载压力。
1.4 zabbix_agent监控代理
Zabbix agents部署在监控目标上，能够主动监控本地资源和应用程序，并将收集到的数据报告给Zabbix Server。
基本步骤: 创建主机（host）-> 创建监控项(item) -> 如果需要：监控项里创建触发器（trigger）-> 告警动作（action）
<!--more-->
	
## 2.环境安装与配置
	
### 2.1 环境准备:
	Centos6.5
	zabbix-3.4.13.tar.gz(server)
	php
	nginx
### 2.2 安装server:
2.2.1 $ tar -zxvf zabbix-3.4.0.tar.gz
2.2.2 Create user account:
	{% codeblock %}
	groupadd --system zabbix
	useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin -c "Zabbix Monitoring System" zabbix
	{% endcodeblock %}
2.2.3 Create Zabbix database
	导入安装包下的数据库脚本（3个）
	source schema.sql, images.sql, data.sql
2.2.4 安装依赖
	net-snmp-devel  libxml2-devel  libcurl-devel ….
2.2.5 Configure the sources
	To configure the sources for a Zabbix server and agent, you may run something like:
	{% codeblock %}
	./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2  --enable-java
	{% endcodeblock %}
	![Example image](images/image2.png)
2.2.6 Make && make install
	Running ‘make install’ will by default install the daemon binaries (zabbix_server, zabbix_agentd, zabbix_proxy) in /usr/local/sbin and the client binaries (zabbix_get, zabbix_sender) in /usr/local/bin.
Review and edit configuration files: /usr/local/etc/zabbix_server.conf
	{% blockquote %}
	#Database host name.
	DBHost=localhost
	#Database name. 必须
	DBName=zabbix
	DBPassword=123456
	DBPort=3306
	DBUser=zabbix
	LogFile=/tmp/zabbix_server.log必须
	{% endblockquote %}
2.2.7 Start up the daemons
	zabbix_server
	{% codeblock %}
	cd /usr/local/sbin/zabbix_java/  
	./startup.sh
	{% endcodeblock %}
### 2.3 zabbix_agent
	a) linux
	1 配置/usr/local/etc/zabbix_agentd.conf
		{% blockquote %}
		Server=121.42.111.220       
		ServerActive=127.0.0.1       
		Hostname=Linux1
		{% endblockquote %}
	2 cp /usr/zabbix/zabbix-3.4.13/misc/init.d/tru64/zabbix_agentd /etc/init.d/
	3 chmod +x /etc/init.d/zabbix_agentd
	4 启动 
		{% codeblock %}
		/etc/init.d/zabbix_agentd start
		{% endcodeblock %}
	b) windows
	1 配置zabbix agent.win.conf
	2 安装
		{% codeblock %}
		E:\zabbix\bin\win64\zabbix_agentd.exe -i -c E:\zabbix\conf\zabbix_agentd.win.conf
		{% endcodeblock %}
	3 启动 
		{% codeblock %}
		E:\zabbix\bin\win64>zabbix_agentd.exe -c E:\zabbix\conf\zabbix_agentd.win.conf –s
		{% endcodeblock %}
	4   netstat -ano|findstr "10050"     
		tasklist|findstr "10268"
### 2.4 zabbix_proxy
	1 {% codeblock %}
		./configure --prefix=/usr/local/zabbix_proxy --enable-proxy --enable-agent --with-mysql --with-net-snmp --with-libcurl
		make
		make install
		{% endcodeblock %}
	2 {% codeblock %}
		create database zabbix_proxy;
		grant all on zabbix_proxy.* to 'user'@'host' identified by 'password;
		flush privileges;
		mysql -hhostname -uuser -ppassword zabbix_proxy < zabbix-2.2.1/database/mysql/schema.sql
		{% endcodeblock %}
	3 zabbix_proxy.conf
		{% blockquote %}
		Server=IP　　#zabbix服务端IP
		Hostname=Zabbix_proxy　　#必须和WEB页面添加代理时设置的名称一致
		LogFile=/tmp/zabbix_proxy.log
		DBHost=IP　　#数据库IP
		DBName=zabbix_proxy　　数据库名
		DBUser=user　　#数据库用户名
		DBPassword=password　　#数据库密码
		{% endblockquote %}
	4 启动 /usr/local/zabbix_proxy/sbin/zabbix_proxy
	5 在zabbix web页面添加代理;
		在proxy服务器上面测试,
		{% codeblock %}
		/usr/local/zabbix_agentd/bin/zabbix_get -s IP -k agent.ping
		{% endcodeblock %}
### 2.5 安装nginx:
	1 {% codeblock %}
	./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre
	{% endcodeblock %}
	![Example image](images/image3.png)
	2 {% codeblock %}
		make 
		make install
		{% endcodeblock %}
	3 启动  /usr/local/nginx/sbin/nginx
	4 访问 curl -s http://localhost | grep nginx.com
	5 停止 /usr/local/nginx/sbin/nginx -s stop
	6 	 /usr/local/nginx/sbin/nginx -s reload
		![Example image](images/image4.png)
		![Example image](images/image5.png)
	7 修改nginx的配置文件，在server里加上：
		{% codeblock %}
		location /ngx_status {
			stub_status on;
			access_log off;
			allow 127.0.0.1;
			allow 192.168.137.1;
		}
		location ~ \.php$ {
            root           /data/site/monitor.ttlsa.com/zabbix;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  zabbix.php;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
        location ~* ^.+\.(ico|gif|jpg|jpeg|png|html|css|htm|bmp|js|svg)$ {
            root           /data/site/monitor.ttlsa.com/zabbix;
        }
		{% endcodeblock %}
	8 http://192.168.137.128/ngx_status
		![Example image](images/image6.png)
### 2.6 安装Php:
	1 依赖 gd-devel libjpeg-devel libpng-devel libxml2-devel bzip2-devel libcurl-devel
		tar –zxvf php-7.2.9.tar.gz
		./configure --prefix=/usr/local/php \
			--with-config-file-path=/usr/local/php/etc --with-bz2 --with-curl \
			--enable-ftp --enable-sockets --disable-ipv6 --with-gd \
			--with-jpeg-dir=/usr/local --with-png-dir=/usr/local \
			--with-freetype-dir=/usr/local --enable-gd-native-ttf \
			--with-iconv-dir=/usr/local --enable-mbstring --enable-calendar \
			--with-gettext --with-libxml-dir=/usr/local --with-zlib \
			--with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mysql=mysqlnd \
			--enable-dom --enable-xml --enable-fpm --with-libdir=lib64 --enable-bcmath \
			--with-ldap
		![Example image](images/image7.png)
	2 配置：
		{% blockquote %}		
		#复制php配置文件到安装目录
		cp php.ini-production /usr/local/php/etc/php.ini
		ln -s /usr/local/php/etc/php.ini  /etc/php.ini
		ls -l /etc/php.ini
		#复制模板文件为php-fpm配置
		cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
		ln -s /usr/local/php/etc/php-fpm.conf  /etc/php-fpm.conf
		#复制php-fpm到启动目录
		cp /usr/php/php-7.2.9/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
		ls -l /etc/init.d/php-fpm
		#赋予php-fpm执行权限
		chmod 755 /etc/init.d/php-fpm
		#设置php-fpm开机启动
		chkconfig php-fpm on
		chkconfig --list php-fpm
		#编辑php配置文件php.ini
		vim /usr/local/php/etc/php.ini
		disable_functions=passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd,posix_getegid,posix_geteuid,posix_getgid,posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid,posix_getppid,posix_getpwnam,posix_getpwuid,posix_getrlimit,posix_getsid,posix_getuid,posix_isatty,posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid,posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname
		启动 /usr/local/php/sbin/php-fpm
		(注意：需要 php-fpm.d下 cp www.conf.default www.conf)
		eg: whereis php
		{% endblockquote %}
	![Example image](images/image8.png)
	![Example image](images/image9.png)
### 2.7 zabbix前端
	{% blockquote %}
	1 拷贝前端文件
		mkdir /data/logs/nginx
		mkdir /data/site/monitor.ttlsa.com/zabbix
		cp -rp frontends/php/* /data/site/monitor.ttlsa.com/zabbix
		php 配置http://192.168.137.128/info.php
		zabbix前端：http://192.168.137.128/setup.php 出错直接到php源码把抛出异常的地方注释掉貌似没什么问题？
		编译时参数加入了php配置路径，Loaded Configuration File还是找不到，然后通过跟踪strace /usr/local/php/sbin/php-fpm -i 2>1.log
		mv php.ini /usr/local/php-5.5.0/etc/  解决，，
		nginx配置（后面那个不加的话，页面访问只有纯文字）

	2 编译安装php的过程中报错：
		第一个报错：
		configure: error: Cannot find ldap.h
		yum install openldap openldap-devel  -y
		第二个报错：
		configure: error: Cannot find ldap libraries in /usr/lib
		cp -frp /usr/lib64/libldap* /usr/lib/
		然后再 ./configure  --with-php-config=/usr/local/php/bin/php-config  --with-ldap
	{% endblockquote %}
	![Example image](images/image10.png)
	![Example image](images/image11.png)
### 2.8 调用api获取数据：
	curl -X POST -H "Content-Type":application/json-rpc --data '{"jsonrpc":"2.0", "method":"apiinfo.version", "id":1, "auth":null, "params":{}}' http://192.168.137.128/api_jsonrpc.php

## 3 tomcat监控
	在tomcat目录下放入catalina-jmx-remote-xx.jar
	在bin目录下创建文件setenv.bat：
	set "JAVA_OPTS=-Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=12345  -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=192.168.3.131"
	
## 4 Windows mysql监控
	mysql.ping.vbs
	{% codeblock %}
	Set objFS =CreateObject("Scripting.FileSystemObject")
	Set objArgs = WScript.Arguments
	str1 = getCommandOutput("C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqladmin-uroot -p'123456' ping") 
	 
	If Instr(str1,"alive") > 0 Then
	WScript.Echo 1
	Else
	WScript.Echo 0
	End If
	 
	Function getCommandOutput(theCommand)
	 
	Dim objShell, objCmdExec
	Set objShell =CreateObject("WScript.Shell")
	Set objCmdExec = objshell.exec(thecommand)
	getCommandOutput =objCmdExec.StdOut.ReadAll
	end Function	
	{% endcodeblock %}
	MYSQL-status.vbs
	{% codeblock %}
	Set objFS = CreateObject("Scripting.FileSystemObject")
	Set objArgs = WScript.Arguments
	str1 = getCommandOutput("C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqladmin-uroot -p'123456' extended-status") 
	Arg = objArgs(0)
	str2 = Split(str1,"|")
	 
	For i = LBound(str2) to UBound(str2)
	 
	If Trim(str2(i)) = Arg Then   
	WScript.Echo TRIM(str2(i+1))
	Exit For
	End If
	next
 
	Function getCommandOutput(theCommand)
	 
	Dim objShell, objCmdExec
	Set objShell =CreateObject("WScript.Shell")
	Set objCmdExec = objshell.exec(thecommand)
	getCommandOutput =objCmdExec.StdOut.ReadAll
	 
	end Function
	{% endcodeblock %}
	mysql_version.vbs
	{% codeblock %}
	Set objFS = CreateObject("Scripting.FileSystemObject")
	Set objArgs = WScript.Arguments
	str1 = getCommandOutput("C:\Program Files\MySQL\MySQL Server 5.7\bin\mysql.exe -V")

	WScript.Echo str1

	Function getCommandOutput(theCommand)
	Dim objShell, objCmdExec
	Set objShell = CreateObject("WScript.Shell")
	Set objCmdExec = objshell.exec(thecommand)
	getCommandOutput = objCmdExec.StdOut.ReadAll
	end Function
	{% endcodeblock %}
	修改zabbix_agentd.win.conf
	{% codeblock %}
	UnsafeUserParameters=1
	UserParameter=mysql.version, cscript /nologo  E:\zabbix_agent\scripts\mysql_version.vbs
	UserParameter=mysql.status[*], cscript /nologo E:\zabbix_agent\scripts\MYSQL-status.vbs $1 
	UserParameter=mysql.ping, cscript /nologo E:\zabbix_agent\scripts\mysql_ping.vbs
	{% endcodeblock %}

## 5 linux mysql 监控：
	找到mysql目录：find / -name mysqladmin    (userparameter_mysql.conf)
	cd /usr/local/zabbix/etc  vi .my.cnf
	{% codeblock %}
	[client]
	user=root
	host=192.168.3.230
	password=123456
	{% endcodeblock %}
	vi /usr/local/zabbix/etc/zabbix_agentd.conf
	{% codeblock %}
	#UserParameter=mysql.status[*],echo "show global status where Variable_name='$1';" | HOME=/var/lib/zabbix mysql -N | awk '{print $$2}'
	UserParameter=mysql.status[*],/usr/local/zabbix/etc/chk_mysql.sh $1 UserParameter=mysql.size[*],bash -c 'echo "select sum($(case "$3" in both|"") echo "data_length+index_length";; data|index) echo "$3_length";; free) echo "data_free";; esac)) from information_schema.tables$([[ "$1" = "all" || ! "$1" ]] || echo " where table_schema=\"$1\"")$([[ "$2" = "all" || ! "$2" ]] || echo "and table_name=\"$2\"");" | /usr/bin/mysql -uroot -p123456 -N'
	UserParameter=mysql.ping, /usr/bin/mysqladmin -uroot -p123456 ping | grep -c alive
	UserParameter=mysql.version, /usr/bin/mysql -uroot -p123456 –V
	{% endcodeblock %}
	将以上mysql路径改为HOME=/usr/local/zabbix/etc, 例如：UserParameter=mysql.ping,HOME=/var/lib/zabbix mysqladmin ping | grep -c alive
	Vi chk_mysql.sh
	{% codeblock %}
	#!/bin/bash
	# 用户名
	MYSQL_USER='root'
	# 密码
	MYSQL_PWD='123456'
	# 主机地址/IP
	MYSQL_HOST='192.168.3.230'
	# 端口
	MYSQL_PORT='3306'
	# 数据连接
	MYSQL_CONN="/usr/bin/mysqladmin -u${MYSQL_USER} -p${MYSQL_PWD} -h${MYSQL_HOST} -P${MYSQL_PORT}"
	# 参数是否正确
	if [ $# -ne "1" ];then 
		echo "arg error!" 
	fi 
	# 获取数据
	case $1 in 
    Uptime) 
        result=`${MYSQL_CONN} status 2>/dev/null |cut -f2 -d":"|cut -f1 -d"T"` 
        echo $result 
        ;; 
    Com_update) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Com_update"|cut -d"|" -f3` 
        echo $result 
        ;; 
    Slow_queries) 
        result=`${MYSQL_CONN} status 2>/dev/null |cut -f5 -d":"|cut -f1 -d"O"` 
        echo $result 
        ;; 
    Com_select) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Com_select"|cut -d"|" -f3` 
        echo $result 
                ;; 
    Com_rollback) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Com_rollback"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Questions) 
        result=`${MYSQL_CONN} status 2>/dev/null |cut -f4 -d":"|cut -f1 -d"S"` 
                echo $result 
                ;; 
    Com_insert) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Com_insert"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_delete) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Com_delete"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_commit) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Com_commit"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Bytes_sent) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Bytes_sent" |cut -d"|" -f3` 
                echo $result 
                ;; 
    Bytes_received) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Bytes_received" |cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_begin) 
        result=`${MYSQL_CONN} extended-status 2>/dev/null |grep -w "Com_begin"|cut -d"|" -f3` 
                echo $result 
                ;; 

        *) 
        echo "Usage:$0(Uptime|Com_update|Slow_queries|Com_select|Com_rollback|Questions|Com_insert|Com_delete|Com_commit|Bytes_sent|Bytes_received|Com_begin)" 
        ;; 
	esac
	{% endcodeblock %}
	chmod +x chk_mysql.sh

## 6 附：我监控用的一些脚本
	
	linux(centos6.5x64):
		vi zabbix_agentd.conf
		UserParameter=mysql.status[*],/usr/local/zabbix/etc/chk_mysql.sh $1
		UserParameter=mysql.Threads,HOME=/usr/local/zabbix/etc mysqladmin status|cut -f3 -d":"|cut -f1 -d"Q"
		UserParameter=mysql.Qps,HOME=/usr/local/zabbix/etc mysqladmin  status|cut -f9 -d":"
		UserParameter=mysql.Key_buffer_size,HOME=/usr/local/zabbix/etc mysql  -e "show variables like 'key_buffer_size';"| grep -v Value |awk '{print $2/1024^2}'
		UserParameter=mysql.Key_cache_miss_rate,echo $(HOME=/usr/local/zabbix/etc mysql  -e "show status like 'key_reads';"| grep -v Value |awk '{print $2}') $(HOME=/usr/local/zabbix/etc mysql  -e "show status like 'key_read_requests';"| grep -v Value |awk '{print $2}')| awk '{printf("%1.4f\n",$1/$2*100)}'
		UserParameter=mysql.Key_blocks_used_rate,echo $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'key_blocks_used';"| grep -v Value |awk '{print $2}') $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'key_blocks_unused';"| grep -v Value |awk '{print $2}')| awk '{printf("%1.4f\n",$1/($1+$2)*100)}'
		UserParameter=mysql.Innodb_buffer_pool_size,HOME=/usr/local/zabbix/etc mysql -e "show variables like 'innodb_buffer_pool_size';"| grep -v Value |awk '{print $2/1024^2}'
		UserParameter=mysql.Threads_connected,HOME=/usr/local/zabbix/etc mysql -e "show status like 'Threads_connected';"| grep -v Value |awk '{print $2}'
		UserParameter=mysql.Table_open_cache_used_rate,echo $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'open_tables';"| grep -v Value |awk '{print $2}') $(HOME=/usr/local/zabbix/etc mysql -e "show variables like 'table_open_cache';"| grep -v Value |awk '{print $2}')| awk '{printf("%1.4f\n",$1/($1+$2)*100)}'
		UserParameter=mysql.Qcache_used_rate,echo $(HOME=/usr/local/zabbix/etc mysql -e "show variables like 'query_cache_size';"| grep -v Value |awk '{print $2}') $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'Qcache_free_memory';"| grep -v Value |awk '{print $2}')| awk '{printf("%1.4f\n",($1-$2)/$1*100)}'
		UserParameter=mysql.Qcache_fragment_rate,echo $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'Qcache_free_blocks';"| grep -v Value |awk '{print $2}') $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'Qcache_total_blocks';"| grep -v Value |awk '{print $2}')| awk '{printf("%1.4f\n",$1/$2*100)}'
		UserParameter=mysql.Max_used_connections,HOME=/usr/local/zabbix/etc mysql -e "show status like 'Max_used_connections';"| grep -v Value |awk '{print $2}'UserParameter=mysql.Qcache_hits_rate,echo $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'Qcache_hits';"| grep -v Value |awk '{print $2}') $(HOME=/usr/local/zabbix/etc mysql -e "show status like 'Qcache_inserts';"| grep -v Value |awk '{print $2}')| awk '{printf("%1.4f\n",($1-$2)/$1*100)}'UserParameter=mysql.Open_files_rate, $(HOME=/usr/local/zabbix/etc mysql -e "show global status like 'open_files';"| grep -v Value |awk '{print $2}') $(HOME=/usr/local/zabbix/etc mysql -e "show variables like 'open_files_limit';"| grep -v Value |awk '{print $2}')| awk '{printf("%1.4f\n",$1/$2*100)}'
		UserParameter=mysql.Ping,HOME=/usr/local/zabbix/etc mysqladmin  ping|grep alive|wc -l
		UserParameter=mysql.version, HOME=/usr/local/zabbix/etc mysql -V

		UserParameter=discovery.processlist,/usr/local/zabbix/etc/discovery_process.sh
		UserParameter=discovery.process,/usr/local/zabbix/etc/discovery_process.sh
		UserParameter=process.check[*],/usr/local/zabbix/etc/process_check.sh $1 $2 $3

		vi .my.cnf
		[client]
		user=root
		host=192.168.3.230
		password=123456

		vi chk_mysql.sh
		#!/bin/bash
		# 用户名
		MYSQL_USER='root'

		# 密码
		MYSQL_PWD='123456'

		# 主机地址/IP
		MYSQL_HOST='192.168.3.230'

		# 端口
		MYSQL_PORT='3306'

		# 数据连接
		MYSQL_CONN="/usr/bin/mysqladmin -u${MYSQL_USER} -p${MYSQL_PWD} -h${MYSQL_HOST} -P${MYSQL_PORT}"

		# 参数是否正确
		if [ $# -ne "1" ];then
			echo "arg error!"
		fi

		# 获取数据
		case $1 in
			Uptime)
				result=`${MYSQL_CONN} status 2>/dev/null|cut -f2 -d":"|cut -f1 -d"T"`
				echo $result
				;;
			Com_update)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Com_update"|cut -d"|" -f3`
				echo $result
				;;
			Slow_queries)
				result=`${MYSQL_CONN} status 2>/dev/null|cut -f5 -d":"|cut -f1 -d"O"`
				echo $result
				;;
			Com_select)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Com_select"|cut -d"|" -f3`
				echo $result
						;;
			Com_rollback)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Com_rollback"|cut -d"|" -f3`
						echo $result
						;;
			Questions)
				result=`${MYSQL_CONN} status 2>/dev/null|cut -f4 -d":"|cut -f1 -d"S"`
						echo $result
						;;
			Com_insert)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Com_insert"|cut -d"|" -f3`
						echo $result
						;;
			Com_delete)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Com_delete"|cut -d"|" -f3`
						echo $result
						;;
			Com_commit)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Com_commit"|cut -d"|" -f3`
						echo $result
						;;
			Bytes_sent)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Bytes_sent" |cut -d"|" -f3`
						echo $result
						;;
			Bytes_received)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Bytes_received" |cut -d"|" -f3`
						echo $result
						;;
			Com_begin)
				result=`${MYSQL_CONN} extended-status 2>/dev/null|grep -w "Com_begin"|cut -d"|" -f3`
						echo $result
						;;

				*)
				echo "Usage:$0(Uptime|Com_update|Slow_queries|Com_select|Com_rollback|Questions|Com_insert|Com_delete|Com_commit|Bytes_sent|Bytes_received|Com_begin)"
				;;
		esac

		vi discovery_process.sh
		#!/bin/bash
		#system process discovery script
		top -b -n 1 > /tmp/.top.txt && chown zabbix. /tmp/.top.txt
		proc_array=(`tail -n +8 /tmp/.top.txt | awk '{a[$NF]+=$10}END{for(k in a)print a[k],k}'|sort -gr|head -10|cut -d" " -f2`)
		length=${#proc_array[@]}

		printf "{\n"
		printf '\t'"\"data\":["
		for ((i=0;i<$length;i++))
		do
			printf "\n\t\t{"
			printf "\"{#PROCESS_NAME}\":\"${proc_array[$i]}\"}"
			if [ $i -lt $[$length-1] ];then
				printf ","
			fi
		done
			printf "\n\t]\n"
		printf "}\n"

		vi process_check.sh
		#!/bin/bash
		#system process CPU&MEM use information
		#mail: mail@huangming.org
		mode=$1
		name=$2
		process=$3
		mem_total=$(cat /proc/meminfo | grep "MemTotal" | awk '{printf "%.f",$2/1024}')
		cpu_total=$(( $(cat /proc/cpuinfo | grep "processor" | wc -l) * 100 ))

		function mempre {
			mem_pre=`tail -n +8 /tmp/.top.txt | awk '{a[$NF]+=$10}END{for(k in a)print a[k],k}' | grep "\b${process}\b" | cut -d" " -f1`
			echo "$mem_pre"
		}

		function memuse {
			mem_use=`tail -n +8 /tmp/.top.txt | awk '{a[$NF]+=$10}END{for(k in a)print a[k]/100*'''${mem_total}''',k}' | grep "\b${process}\b" | cut -d" " -f1`
			echo "$mem_use" | awk '{printf "%.f",$1*1024*1024}'
		}

		function cpuuse {
			cpu_use=`tail -n +8 /tmp/.top.txt | awk '{a[$NF]+=$9}END{for(k in a)print a[k],k}' | grep "\b${process}\b" | cut -d" " -f1`
			echo "$cpu_use"
		}

		function cpupre {
			cpu_pre=`tail -n +8 /tmp/.top.txt | awk '{a[$NF]+=$9}END{for(k in a)print a[k]/('''${cpu_total}'''),k}' | grep "\b${process}\b" | cut -d" " -f1`
			echo "$cpu_pre"
		}


		case $name in
			mem)
				if [ "$mode" = "pre" ];then
					mempre
				elif [ "$mode" = "avg" ];then
					memuse
				fi
			;;
			cpu)
				if [ "$mode" = "pre" ];then
					cpupre
				elif [ "$mode" = "avg" ];then
					cpuuse
				fi
			;;
			*)
				echo -e "Usage: $0 [mode : pre|avg] [mem|cpu] [process]"
		esac


		windows(win10x64):
		zabbix_agentd.win.conf ->
		#mysql状态数据
		UserParameter=mysql.version, cscript /nologo  E:\zabbix_agent\scripts\mysql_version.vbs
		UserParameter=mysql.status[*], cscript /nologo E:\zabbix_agent\scripts\MYSQL-status.vbs $1 
		#mysql性能数据
		UserParameter=mysql.xn[*], cscript /nologo E:\zabbix_agent\scripts\mysql_xn.vbs $1
		#系统进程列表
		UserParameter=process.list, cscript /nologo E:\zabbix_agent\scripts\process_list.vbs

		mysql_version.vbs ->
		Set objFS = CreateObject("Scripting.FileSystemObject")
		Set objArgs = WScript.Arguments
		str1 = getCommandOutput("C:\Program Files\MySQL\MySQL Server 5.7\bin\mysql.exe -V")

		WScript.Echo str1

		Function getCommandOutput(theCommand)
		Dim objShell, objCmdExec
		Set objShell = CreateObject("WScript.Shell")
		Set objCmdExec = objshell.exec(thecommand)
		getCommandOutput = objCmdExec.StdOut.ReadAll
		end Function

		MYSQL-status.vbs ->
		Set objFS = CreateObject("Scripting.FileSystemObject")
		Set objArgs = WScript.Arguments
		str1 = getCommandOutput("C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqladmin-uroot -p123456 extended-status") 
		Arg = objArgs(0)
		str2 = Split(str1,"|")
		 
		For i = LBound(str2) to UBound(str2)
		 
		If Trim(str2(i)) = Arg Then   
		WScript.Echo TRIM(str2(i+1))
		Exit For
		End If
		next
		 
		 
		Function getCommandOutput(theCommand)
		 
		Dim objShell, objCmdExec
		Set objShell =CreateObject("WScript.Shell")
		Set objCmdExec = objshell.exec(thecommand)
		getCommandOutput =objCmdExec.StdOut.ReadAll

		end Function

		mysql_xn.vbs ->
		Set objFS = CreateObject("Scripting.FileSystemObject")
		Set objArgs = WScript.Arguments
		Arg = objArgs(0)
		'已创建线程数
		If "Threads" = Arg Then
			str1 = getCommandOutput1(" status") 
			str2 = Split(str1,"  ") 
			For i = LBound(str2) to UBound(str2)
			str3 = Split(Trim(str2(i)),":")
			If Trim(str3(0)) = "Threads" Then   
			WScript.Echo CINT(TRIM(str3(1)))
			Exit For
			End If
			next
		'索引缓存未命中率
		ElseIf "Key_cache_miss_rate" = Arg Then
			str1 = getCommandOutput("-e ""show status like 'key_reads';""") 
			str1_1 = Split(CStr(str1), "Key_reads")
			str1_2 = CINT(Trim(str1_1(1)))	
			str2 = getCommandOutput("-e ""show status like 'key_read_requests';""")
			str2_1 = Split(CStr(str2), "Key_read_requests")
			str2_2 = CINT(Trim(str2_1(1)))
			WScript.Echo formatnumber(str1_2/str2_2*100,4,true) 
		'缓存簇使用率
		ElseIf "Key_blocks_used_rate" = Arg Then
			str1 = getCommandOutput("-e ""show status like 'key_blocks_used';""") 
			str1_1 = Split(CStr(str1), "Key_blocks_used")
			str1_2 = CINT(Trim(str1_1(1)))
			str2 = getCommandOutput("-e ""show status like 'key_blocks_unused';""") 
			str2_1 = Split(CStr(str2), "Key_blocks_unused")
			str2_2 = CINT(Trim(str2_1(1)))
			WScript.Echo formatnumber(str1_2/(str1_2+str2_2)*100,4,true)
		'当前链接数
		ElseIf "Threads_connected" = Arg Then
			str1 = getCommandOutput("-e ""show status like 'Threads_connected';""") 
			str1_1 = Split(CStr(str1), "Threads_connected")
			str1_2 = CINT(Trim(str1_1(1)))
			WScript.Echo str1_2
		'表缓存利用率
		ElseIf "Table_open_cache_used_rate" = Arg Then
			str1 = getCommandOutput("-e ""show status like 'open_tables';""") 
			str1_1 = Split(CStr(str1), "Open_tables")
			str1_2 = CINT(Trim(str1_1(1)))
			str2 = getCommandOutput("-e ""show variables like 'table_open_cache';""") 
			str2_1 = Split(CStr(str2), "table_open_cache")
			str2_2 = CINT(Trim(str2_1(1)))
			WScript.Echo formatnumber(str1_2/(str1_2+str2_2)*100,4,true)
		'查询缓存利用率
		ElseIf "Qcache_used_rate" = Arg Then
			str1 = getCommandOutput("-e ""show variables like 'query_cache_size';""") 
			str1_1 = Split(CStr(str1), "query_cache_size")
			str1_2 = CINT(Trim(str1_1(1)))
			If str1_2 = 0 Then 
			WScript.Echo 0
			Else 
			str2 = getCommandOutput("-e ""show status like 'Qcache_free_memory';""") 
			str2_1 = Split(CStr(str2), "Qcache_free_memory")
			str2_2 = CINT(Trim(str2_1(1)))
			WScript.Echo formatnumber((str1_2-str2_2)/str1_2*100,4,true)
			End If
		'查询缓存碎片率
		ElseIf "Qcache_fragment_rate" = Arg Then
			str1 = getCommandOutput("-e ""show status like 'Qcache_free_blocks';""") 
			str1_1 = Split(CStr(str1), "Qcache_free_blocks")
			str1_2 = CINT(Trim(str1_1(1)))
			If str1_2 = 0 Then
			WScript.Echo 0
			Else 
			str2 = getCommandOutput("-e ""show status like 'Qcache_total_blocks';""") 
			str2_1 = Split(CStr(str2), "Qcache_total_blocks")
			str2_2 = CINT(Trim(str2_1(1)))
			WScript.Echo formatnumber(str1_2/str2_2*100,4,true)
			End If
		'最大响应链接数
		ElseIf "Max_used_connections" = Arg Then
			str1 = getCommandOutput("-e ""show status like 'Max_used_connections';""") 
			str1_1 = Split(CStr(str1), "Max_used_connections")
			str1_2 = CINT(Trim(str1_1(1)))
			WScript.Echo str1_2
		'查询缓存命中率
		ElseIf "Qcache_hits_rate" = Arg Then
			str1 = getCommandOutput("-e ""show status like 'Qcache_hits';""") 
			str1_1 = Split(CStr(str1), "Qcache_hits")
			str1_2 = CINT(Trim(str1_1(1)))
			If str1_2 = 0 Then
			WScript.Echo 0
			Else 
			str2 = getCommandOutput("-e ""show status like 'Qcache_inserts';""") 
			str2_1 = Split(CStr(str2), "Qcache_inserts")
			str2_2 = CINT(Trim(str2_1(1)))
			WScript.Echo formatnumber((str1_2-str2_2)/str1_2*100,4,true)
			End If
		'文件使用率
		ElseIf "Open_files_rate" = Arg Then
			str1 = getCommandOutput("-e ""show global status like 'open_files';""") 
			str1_1 = Split(CStr(str1), "Open_files")
			str1_2 = CINT(Trim(str1_1(1)))
			str2 = getCommandOutput("-e ""show variables like 'open_files_limit';""") 
			str2_1 = Split(CStr(str2), "open_files_limit")
			str2_2 = CINT(Trim(str2_1(1)))
			WScript.Echo formatnumber(str1_2/str2_2*100,4,true)
		ElseIf "Ping" = Arg Then
			str1 = getCommandOutput1(" ping") 
			If Instr(str1, "alive") Then 
			WScript.Echo 1
			Else 
			WScript.Echo 0
			End If
		End If

		Function getCommandOutput1(theCommand)
			Dim objShell, objCmdExec
			Set objShell =CreateObject("WScript.Shell")
			Set objCmdExec = objshell.exec("C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqladmin -uroot -p123456 " & thecommand)
			getCommandOutput1 =objCmdExec.StdOut.ReadAll
		end Function

		Function getCommandOutput(theCommand)
			Dim objShell, objCmdExec
			Set objShell =CreateObject("WScript.Shell")
			Set objCmdExec = objshell.exec("C:\Program Files\MySQL\MySQL Server 5.7\bin\mysql -uroot -p123456 " & thecommand)
			getCommandOutput =objCmdExec.StdOut.ReadAll
		end Function

		process_list.vbs ->
		Dim WMI,Objs,Process
		Set WMI=GetObject("WinMgmts:")
		Set Objs=WMI.InstancesOf("Win32_Process")
		Process=""
		For Each Obj In Objs
		  Process=Process & Obj.Description & Chr(13) & Chr(10)
		Next
		WScript.Echo Process
		
## 7 监控硬件的一些脚本

		windows:
		利用SpeedFan软件的日志监控；
		cp_cpufan.vbs:
		'硬件信息
		set fs =createobject("Scripting.FileSystemObject")
		Set objArgs = WScript.Arguments
		Arg = objArgs(0)

		month1 = "" & Month(Now)
		if Len(month1)<2 Then month1 = "0"&month1 End If
		day1 = day(Now)
		if Len(day1)<2 Then day1 = "0"&day1 End If
		dates = year(Now)& month1 & day1
		set ts=fs.opentextfile("C:\Program Files (x86)\SpeedFan\SFLog"& dates &".csv",1,true)
		line=CStr(ts.read(100))
		ts.close
		'日志记录为如下格式
		'Seconds GPU     CPU     Fan1    Fan2
		'48968   32.0    41.0    853     984
		'48970   32.0    41.0    853     984
		ta = Split(line, vbcrlf)
		tt = Split(ta(1), "	")
		If Arg = "GPU_TEMP" Then 
			WScript.Echo tt(1)
		ElseIf Arg = "CPU_TEMP" Then 
			WScript.Echo tt(2)
		ElseIf Arg = "Fan1" Then 
			WScript.Echo tt(3)
		ElseIf Arg = "Fan2" Then 
			WScript.Echo tt(4)	
		End If

		zabbix_agentd.win.conf:
		UserParameter=hardware[*], cscript /nologo E:\zabbix_agent\scripts\cp_cpufan.vbs $1

		Linux:
		利用yum install lm_sensors；运行  sensors-detect　检测内核模块，在引导下直接enter，使用默认选项
		检测结束后运行 sensors ，可以看到每颗CPU每个核心的温度
		vi zabbix_agentd.conf:
		UserParameter=get_temp_cpu[*],sensors|grep "Physical id $1"|cut -c 17-20