# 第一节 FTP 服务器

FTP 意为文件传输协议。使用 FTP 服务搭建服务器可以快速传输文件。

## pure-ftpd（以 MySQL 支持为例）

> **RFC 2640 的支持已经被移除，所以 Windows 下的 FTP 文件会乱码，见** [**https://www.pureftpd.org/project/pure-ftpd/news/**](https://www.pureftpd.org/project/pure-ftpd/news/) **无法解决，同时不建议把 Windows 的系统编码改为 UTF8 ，会造成更多乱码的发生，比如 zip 文件。**
>
> **注意：本示例以 mysql 5.x 为例。**

### 安装

由于 pkg 包不带有数据库支持功能，所以需要通过 ports 来安装该软件:

```
# /usr/ports/ftp/pure-ftpd
# make config-recursive
```

选中 mysql，其余保持默认选项回车即可：

![](../.gitbook/assets/在FreeBsd中安装PureFTPD（MySQL）.jpg)

```
# make install clean
```

> **注意：关于 mysql 的基本设置请看 第十七章**
>
> **请自行安装 mysql，理论上兼容 mysql 5.x、8.x**

### 配置 /usr/local/etc/pure-ftpd.conf 文件

#### 生成配置文件：

```
# cp /usr/local/etc/pure-ftpd.conf.sample /usr/local/etc/pure-ftpd.conf
# cp /usr/local/etc/pureftpd-mysql.conf.sample /usr/local/etc/pureftpd-mysql.conf
```

#### 编辑配置文件并增加 mysql 的支持：

```
#兼容 ie 等非正规化的 ftp 客户端

BrokenClientsCompatibility yes

# 被动连接响应的端口范围。
PassivePortRange 30000 50000

# 认证用户允许登陆的最小组 ID（UID） 。
MinUID 2000

# 仅允许认证用户进行 FXP 传输。
AllowUserFXP yes

# 用户主目录不存在的话，自动创建。
CreateHomeDir yes

# MySQL configuration file (see README.MySQL)

MySQLConfigFile /usr/local/etc/pureftpd-mysql.conf
```

### 配置 mysql

#### 创建数据库

```
create database pureftp;
use pureftp;
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
   `User` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
   `Password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
   `Uid` int(11) NOT NULL DEFAULT -1 COMMENT '用户ID',
   `Gid` int(11) NOT NULL DEFAULT -1 COMMENT '用户组ID',
   `Dir` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
   `quotafiles` int(255) NULL DEFAULT 500,
   `quotasize` int(255) NULL DEFAULT 30,
   `ulbandwidth` int(255) NULL DEFAULT 80,
    `dlbandwidth` int(255) NULL DEFAULT 80,
   `ipaddress` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT'*',
   `comment` int(255) NULL DEFAULT NULL,
   `status` tinyint(4) NULL DEFAULT 1,
   `ulratio` int(255) NULL DEFAULT 1,
   `dlratio` int(255) NULL DEFAULT 1,
   PRIMARY KEY (`User`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
INSERT INTO `users` VALUES ('demo', 'demo&2022*', 2002, 2000, '/home/www/demo', 500, 30, 80, 80, '*', NULL, NULL, 1, 1);
```

#### 创建登录数据库用户及设置密码

```
grant select,insert,update,delete on pureftp.* to pftp@localhost identified by "Ab123456&";
```

### 配置 /usr/local/etc/pureftpd-mysql.conf

```
##############################################
# #
# Sample Pure-FTPd Mysql configuration file. #
# See README.MySQL for explanations. #
# #
##############################################


# Optional : MySQL server name or IP. Don't define this for unix sockets.

# MYSQLServer 127.0.0.1
MYSQLServer localhost


# Optional : MySQL port. Don't define this if a local unix socket is used.

MYSQLPort 3306


# Optional : define the location of mysql.sock if the server runs on this host.

MYSQLSocket /var/run/mysqld/mysqld.sock


# Mandatory : user to bind the server as.

MYSQLUser pftp


# Mandatory : user password. You must have a password.

MYSQLPassword Ab123456&


# Mandatory : database to open.

MYSQLDatabase pureftpd


# Mandatory : how passwords are stored
# Valid values are : "cleartext", "argon2", "scrypt", "crypt", "sha1", "md5",password" and "any"

# ("password" = MySQL password() function, which is sha1(sha1(password)))

#MYSQLCrypt scrypt
MYSQLCrypt cleartext


# In the following directives, parts of the strings are replaced at
# run-time before performing queries :
#
# \L is replaced by the login of the user trying to authenticate.
# \I is replaced by the IP address the user connected to.
# \P is replaced by the port number the user connected to.
# \R is replaced by the IP address the user connected from.
# \D is replaced by the remote IP address, as a long decimal number.
#
# Very complex queries can be performed using these substitution strings,
# especially for virtual hosting.

# Query to execute in order to fetch the password

MYSQLGetPW SELECT Password FROM users WHERE User='\L'


# Query to execute in order to fetch the system user name or uid
MYSQLGetUID SELECT Uid FROM users WHERE User='\L'


# Optional : default UID - if set this overrides MYSQLGetUID

MYSQLDefaultUID 2000


# Query to execute in order to fetch the system user group or gid

MYSQLGetGID SELECT Gid FROM users WHERE User='\L'


# Optional : default GID - if set this overrides MYSQLGetGID

MYSQLDefaultGID 2000


# Query to execute in order to fetch the home directory

MYSQLGetDir SELECT Dir FROM users WHERE User='\L'


# Optional : query to get the maximal number of files
# Pure-FTPd must have been compiled with virtual quotas support.

# MySQLGetQTAFS SELECT QuotaFiles FROM users WHERE User='\L'


# Optional : query to get the maximal disk usage (virtual quotas)
# The number should be in Megabytes.
# Pure-FTPd must have been compiled with virtual quotas support.

# MySQLGetQTASZ SELECT QuotaSize FROM users WHERE User='\L'


# Optional : ratios. The server has to be compiled with ratio support.

# MySQLGetRatioUL SELECT ULRatio FROM users WHERE User='\L'
# MySQLGetRatioDL SELECT DLRatio FROM users WHERE User='\L'


# Optional : bandwidth throttling.
# The server has to be compiled with throttling support.
# Values are in KB/s .

# MySQLGetBandwidthUL SELECT ULBandwidth FROM users WHERE User='\L'
# MySQLGetBandwidthDL SELECT DLBandwidth FROM users WHERE User='\L'


# Enable ~ expansion. NEVER ENABLE THIS BLINDLY UNLESS :
# 1) You know what you are doing.
# 2) Real and virtual users match.

# MySQLForceTildeExpansion 1


# If you're using a transactionnal storage engine, you can enable SQL
# transactions to avoid races. Leave this commented if you are using the
# traditional MyIsam engine.

# MySQLTransactions On
```

### 添加 ftp 组和用户

```
# pw groupadd ftpgroup -g 2000
# pw useradd ftpuser -u 2001 -g 2000
```

或

```
# pw useradd ftpuser -u 2001 -g 2000 -s /sbin/nologin -w no -d /home/vftp -c "VirtualUser Pure-FTPd" -m
```

### 配置 FTP 目录

```
# mkdir /home/www/pureftp
# chown -R ftpuser /home/www/
# chgrp -R ftpgroup /home/www/
```

### 服务操作

```
# sysrc pureftpd_enable="YES"
# service pure-ftpd start   #启动服务器
# service pure-ftpd stop    #停止服务
# service pure-ftpd restart #重启服务

## proftpd（以 mysql 支持为例）
```

## 安装 proftpd（以 mysql 支持为例）

```
# pkg install proftpd proftpd-mod_sql_mysql
```

### 编辑配置文件 /usr/local/etc/proftpd.conf

```
# cat /usr/local/etc/proftpd.conf
ServerName "Test Ftp Server"
ServerType standalone
DefaultServer on
ServerIdent on "FTP Server ready"
DeferWelcome off
Port 21
Umask 022
TimeoutLogin 300
TimeoutIdle 36000
TimeoutNoTransfer 36000
TimeoutStalled 36000
TimeoutSession 0
User proftpd
Group proftpd
MaxInstances 100
MaxClientsPerHost 100
AllowRetrieveRestart on
AllowStoreRestart on
AllowOverwrite on
AllowOverride off
RootLogin off
IdentLookups off
UseReverseDNS off
DenyFilter \*.*/
TimesGMT off
DefaultRoot ~
#RLimitCPU 1200 1200
RLimitMemory 256M 256M
RLimitOpenFiles 1024 1024
PassivePorts 50000 60000
LogFormat default "%h %l %u %t \"%r\" %s %b"
LogFormat auth "%v [%P] %h %t \"%r\" %s"
LogFormat write "%h %l %u %t \"%r\" %s %b"
SystemLog /var/log/proftpd/proftpd.log
TransferLog /var/log/proftpd/xfer.log
ExtendedLog /var/log/proftpd/access.log WRITE,READ write
ExtendedLog /var/log/proftpd/auth.log AUTH auth
LoadModule mod_sql.c
LoadModule mod_sql_mysql.c
<Global>
   SQLConnectInfo proftpd@localhost proftpd proftpd_password
   SQLAuthTypes Crypt
   SQLUserInfo users username password uid gid homedir NULL
   SQLDefaultGID 2000
   SQLDefaultUID 2000
   RequireValidShell off
   SQLAuthenticate users*
   SQLLogFile /var/log/proftpd.log
   SQLNamedQuery getcount SELECT "count, username from users where username='%u'"
   SQLNamedQuery updatecount UPDATE "count=count+1 WHERE username='%u'" users
   SQLShowInfo PASS "230" "You've logged on %{getcount} times, %u"
   SQLLog PASS updatecount
   SQLLog DELE,RETR,STOR, log_work
   SQLNamedQuery log_work FREEFORM "\
   INSERT INTO worklog (\
   user_name,\
   file_and_path,\
   bytes,\
   send_time,\
   client_ip,\
   client_name,\
   client_command) \
  VALUES('%u','%f','%b','%T','%a','%h','%m')"
</Global>
```

我们在设置中指定服务器将在主动模式下在端口 21 上工作，在被动模式下在 50000-60000 范围内工作.这些端口应该在防火墙中打开。 对于 PF，这是通过以下规则完成的：

```
pass in quick on $ext_if proto tcp from any to $ext_if port { 21, 50000:60000 }
```

### 创建用户

出于安全目的，我们将以非 root 用户身份运行 Proftpd。 因此，我们将创建此用户：

```
# adduser
Username: proftpd
Full name: FTP User
Uid (Leave empty for default):
Login group [proftpd]:
Login group is proftpd. Invite proftpd into other groups? []:
Login class [default]:
Shell (sh csh tcsh bash nologin) [sh]: nologin
Home directory [/home/proftpd]:
Home directory permissions (Leave empty for default):
Use password-based authentication? [yes]: no
Lock out the account after creation? [no]:
Username : proftpd
Password : <disabled>
Full Name : FTP User
Uid : 2000
Class :
Groups : proftpd
Home : /home/proftpd
Shell : /usr/sbin/nologin
Locked : no
OK? (yes/no): yes
adduser: INFO: Successfully added (proftpd) to the user database.
Add another user? (yes/no): no
Goodbye!
```

现在已经创建了自己的 proftpd 用户和组 ID。 因此，在添加 ftp 用户时，您将使用它。 您可以通过以 下方式确定 UID：

```
# cat /etc/passwd | grep proftpd
proftpd:*:2000:2000:FTP User:/home/proftpd:/usr/sbin/nologin
```

### 日志相关

创建一个目录来存储 FTP 服务器的日志：

```
# mkdir /var/log/proftpd
```

创建一个 MySQL 数据库和一个对创建的数据库具有完全访问权限的用户:

```
CREATE DATABASE `proftpd` CHARACTER SET utf8 COLLATE utf8_general_ci;
```

创建数据库用户和密码(授权 proftpd 数据库)：

```
grant select,insert,update,delete on proftpd.* to pftp@localhost identified by "123456";
FLUSH PRIVILEGES;  立即生效权限
```

或

```
grant select,insert,update,delete on *.* to pftp@"localhost" Identified by "123456";
```

创建数据量：

```
DROP TABLE IF EXISTS users;
CREATE TABLE `users` (
   `username` varchar(30) NOT NULL DEFAULT '',
   `descr` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
   `password` varchar(30) NOT NULL DEFAULT '',
   `uid` int(11) DEFAULT NULL,
   `gid` int(11) DEFAULT NULL,
   `homedir` varchar(255) DEFAULT NULL,
   `shell` varchar(255) DEFAULT NULL,
   `count` int(11) NOT NULL DEFAULT '0',
  UNIQUE KEY `username` (`username`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4;
DROP TABLE IF EXISTS worklog;
CREATE TABLE worklog (
   id bigint(20) UNSIGNED NOT NULL AUTO_INCREMENT,
   date timestamp(0) NULL DEFAULT CURRENT_TIMESTAMP(0),
   user_name varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
   file_and_path varchar(1024) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL,
   bytes bigint(20) NULL DEFAULT NULL,
   send_time varchar(9) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
   client_ip varchar(15) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
   client_name text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL,
   client_command varchar(5) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
   PRIMARY KEY (id) USING BTREE,
   UNIQUE INDEX id(id) USING BTREE
) ENGINE = MyISAM CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = DYNAMIC;
```

创建一个目录和一个测试 FTP 用户，将创建的目录指定为用户目录：

```
# mkdir -p /home/www/ftp
# chown -R proftpd:proftpd /home/www/ftp
# mysql -u proftpd -p
INSERT INTO `proftpd`.`users` (`username` , `descr` , `password` , `uid` , `gid` ,`homedir` , `shell` , `count` ) VALUES ('test', 'Test user', ENCRYPT('FTPpassword_here' ) , '2000', '2000', '/home/www/ftp', NULL , '0' );

Query OK, 1 row affected, 1 warning (0.02 sec)
```

### 服务操作

```
# sysrc  proftpd_enable="YES"

# service proftpd start #启动服务器

# service proftpd stop #停止服务

# service proftpd restart #重启服务
```

## **连接到 FTP 服务器**

简单示例：

```
# telnet localhost 21
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 FTP Server ready
quit
221 Goodbye.
```

使用 `ftp` 命令可以快速连接到 FTP 服务器。

用法:

```
ftp [选项] [URL]
```

选项：

`-4` 强制使用 IPv4 协议连接

`-6` 强制使用 IPv6 协议连接

`-a` 使用匿名登录

`-q` \[quittime] 在设定时间后连接失败则自动放弃连接

`-r` \[wait] 每隔 `wait` 秒发送一次连接请求

`-A` 强制使用主动模式

`-d` 开启调试模式

`-v` 开启啰嗦模式

`-V` 关闭啰嗦模式

#### 登录后的命令：

```
account [passwd] 提交补充密码

append [locol-file] [remote-file] 以 remote-file 为文件名向服务器上传本地文件 local-file

ascii 将FTP文件传送类型设置为 ASCII 模式

bell 在文件传送完后发出提示音

bye 结束与服务器的会话

cd 切换目录

cdup 退回父目录

delete 删除文件

dir 显示该目录下的文件及文件夹

features 显示该服务器支持的功能

get remote-fil 下载服务器上的 remote-file
```
