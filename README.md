##一建编译安装(bate)

```
sudo install git -y
git clone https://git.coding.net/xin/centos7_LNPM.git
cd centos7_LNPM
sudo ./install.sh
```


















----------------------------

## 原文教程

环境：

系统硬件：vmware vsphere (CPU：2*4核，内存2G，双网卡)
系统版本：CentOS-7-x86_64-Minimal-1503-01.iso


安装步骤：


1.准备

1.1 主机名设置

当前主机名查看
```shell
hostname
```
主机名设置
```shell
hostnamectl --static set-hostname tCentos
service network restart
hostname
```
> tCentos


1.2 设置静态IP、DNS地址（网络设备名称有可能不一样，这里是eno16780032，如使用DHCP获取动态IP，可忽略）
```shell
vi /etc/sysconfig/network-scripts/ifcfg-eno16780032
```

找到BOOTPROTO，并且修改（设为静态网址）

`BOOTPROTO="static"`

在最后添加三行内容（添加IP，子网掩码，网关）
```
IPADDR="192.168.1.117"
NETMASK="255.255.255.0"
GATEWAY="192.168.1.1"
```

：wq 保存退出

```shell
vi /etc/resolv.conf
```

添加以下几个DNS地址
```
nameserver 114.114.114.114
nameserver 192.168.1.1
nameserver 8.8.8.8
```

：wq 保存退出


1.3 显示IP地址
```shell
ip addr|grep inet
```
> ```
inet 127.0.0.1/8 scope host lo
inet6 ::1/128 scope host
inet 192.168.1.117/24 brd 192.168.1.255 scope global eno16780032
inet6 fe80::250:56ff:feb0:30f2/64 scope link
```
 

1.4 安装基本软件包
```shell
yum install vim wget lsof gcc gcc-c++ bzip2 -y
yum install net-tools bind-utils -y
yum install firewalld -y
```


1.5 更新系统，显示系统版本(使用阿里云源)
```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
yum update -y
```

更新完成后重启，查看内核版本
```shell
shutdown -r now
cat /etc/redhat-release
```
> ```
CentOS Linux release 7.1.1503 (Core)
```

```shell
uname -a
Linux centos 3.10.0-229.20.1.el7.x86_64 #1 SMP Tue Nov 3 19:10:07 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```

 
 
 

1.6 关闭selinux (如不关闭，有时会添加不了用户，或者重启后没法开机)
```shell
vim /etc/selinux/config
```
屏蔽以下两行
```
#SELINUX=enforcing
#SELINUXTYPE=targeted
```
添加以下一行
```
SELINUXTYPE=disabled
```
保存，退出


重启后，查询是否关闭（显示Disabled则表示关闭）
```shell
shutdown -r now
getenforce
```
> ```
Disabled
```

1.7 设置PUTTY远程登录时，不使用密码，使用密钥文件登录（如不需要，可忽略）

1.7.1 在客户机生成对称密钥
```shell
ssh-keygen -t rsa
```

1.7.2 把客户机上的公钥复制到服务器（公钥文件：id_rsa.pub）

创建目录
```shell
mkdir -p /root/.ssh
```
使用软件远程复制id_rsa.pub到服务器/root/.ssh中。


查看服务器上，公钥是否已经存在
```shell
ll /root/.ssh
-rw-r--r-- 1 root root 394 12月 5 09:33 id_rsa.pub
```



导入密钥到authorized_keys
```shell
cat id_rsa.pub >> authorized_keys
ll /root/.ssh
-rw-r--r-- 1 root root 394 12月 5 09:37 authorized_keys
-rw-r--r-- 1 root root 394 12月 5 09:33 id_rsa.pub
```

导入后，删除公钥文件
```shell
rm id_rsa.pub
```

1.7.3 设置sshd配置文件
```shell
vim /etc/ssh/sshd_config
```
找到GSSAPICleanupCredentials，并且修改为以下内容
```
GSSAPICleanupCredentials yes
```
:wq 保存退出


重启sshd服务，让其生效
```shell
systemctl restart sshd
```

1.7.4 客户端设置PUTTY，进行远程登录

打开软件 PuTTYgen

点击load 选择之前客户机生成私钥文件id_rsa, 点击save private key 生成 pKey.ppk文件

打开软件 PuTTY

点击Session，在HostName(or IP address)输入服务器地址

点击Connection下的DATA，在Auto-login username中输入登录账号（当前账号为root）

点击Connection下的SSH下的Auth,点击Browse 选择之前生成 pKeyppk文件

点击Session，在Saved Sessions中，输入需要保存的Session名称，点击保存


1.7.5 设置完成后，即可以远程连接到服务器

打开软件 PuTTY

点击Session，在"Default Settings"下，找到之前已经保存的Session，双击打开连接

如果显示 Authenticating with public key "xxxxx-xxxx"时，即表未成功


1.8 下载源码包（或直接通过FTP上传到指定目录）
```shell
cd /usr/local/src
wget https://cmake.org/files/v3.4/cmake-3.4.1.tar.gz
wget http://ftp.gnu.org/gnu/bison/bison-3.0.4.tar.gz
wget https://github.com/jemalloc/jemalloc/releases/download/4.0.4/jemalloc-4.0.4.tar.bz2
wget https://downloads.mariadb.org/f/mariadb-10.1.9/source/mariadb-10.1.9.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
wget https://www.openssl.org/source/openssl-1.0.2e.tar.gz
wget http://zlib.net/zlib-1.2.8.tar.gz
wget http://nginx.org/download/nginx-1.9.9.tar.gz
wget http://am1.php.net/get/php-7.0.2.tar.gz
wget http://ftp.gnu.org/gnu/libiconv/libiconv-1.14.tar.gz
wget http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
wget http://sourceforge.net/projects/mcrypt/files/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
wget http://sourceforge.net/projects/mhash/files/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz
wget http://pecl.php.net/get/zendopcache-7.0.5.tgz
wget http://xdebug.org/files/xdebug-2.4.0rc3.tgz
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
wget https://sourceforge.net/projects/levent/files/libevent/libevent-2.0/libevent-2.0.22-stable.tar.gz
```

查看是否已经需要的包是否下载完成(如下载地址失效，需另外下载并上传到服务器)
```shell
ll /usr/local/src
```

2.安装mariadb

2.1 安装依赖
```shell
yum install ncurses-devel openssl* bzip2 m4 -y
```

2.2 安装cmake
```shell
cd /usr/local/src/
wget https://cmake.org/files/v3.4/cmake-3.4.1.tar.gz
tar zvxf cmake-3.4.1.tar.gz
cd cmake-3.4.1
./bootstrap && make && make install
```

2.3 安装bison(需要 m4 库)
```shell
cd /usr/local/src/
wget http://ftp.gnu.org/gnu/bison/bison-3.0.4.tar.gz
tar zvxf bison-3.0.4.tar.gz
cd bison-3.0.4
./configure && make && make install
```

2.4 安装jemalloc(需要 bzip2 库解压)
```shell
cd /usr/local/src/
wget https://github.com/jemalloc/jemalloc/releases/download/4.0.4/jemalloc-4.0.4.tar.bz2
tar xjf jemalloc-4.0.4.tar.bz2
cd jemalloc-4.0.4
./configure && make && make install
echo '/usr/local/lib' > /etc/ld.so.conf.d/local.conf
ldconfig
```
 

2.5 安装libevent
```shell
cd /usr/local/src
wget https://sourceforge.net/projects/levent/files/libevent/libevent-2.0/libevent-2.0.22-stable.tar.gz
tar zvxf libevent-2.0.22-stable.tar.gz
cd libevent-2.0.22-stable
./configure --prefix=/usr
make && make install
```
 

如果libevent的lib目录不在LD_LIBRARY_PATH里，可以使用以下命令加入
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr
```

2.6 创建mysql需要的目录、配置用户和用户组
```
groupadd mysql
useradd -g mysql mysql -s /sbin/nologin
mkdir -p /data/mysql
chown -R mysql:mysql /data/mysql
```

2.7 编译mariadb(需要 cmake ncurses-devel bison　库)
```
cd /usr/local/src/
wget https://downloads.mariadb.org/f/mariadb-10.1.9/source/mariadb-10.1.9.tar.gz
tar zvxf mariadb-10.1.9.tar.gz
cd mariadb-10.1.9
cmake \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=/opt/mysql \
-DINSTALL_DOCDIR=share/doc/mariadb-10.1.9 \
-DINSTALL_DOCREADMEDIR=share/doc/mariadb-10.1.9 \
-DINSTALL_MANDIR=share/man \
-DINSTALL_MYSQLSHAREDIR=share/mysql \
-DINSTALL_MYSQLTESTDIR=share/mysql/test \
-DINSTALL_PLUGINDIR=lib/mysql/plugin \
-DINSTALL_SBINDIR=sbin \
-DINSTALL_SCRIPTDIR=bin \
-DINSTALL_SQLBENCHDIR=share/mysql/bench \
-DINSTALL_SUPPORTFILESDIR=share/mysql \
-DMYSQL_DATADIR=/data/mysql \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DWITH_EXTRA_CHARSETS=complex \
-DWITH_EMBEDDED_SERVER=ON \
-DTOKUDB_OK=0
make && make install
```

2.8 创建软连接
```
ln -s /opt/mysql/lib/lib* /usr/lib/
ln -s /opt/mysql/bin/mysql /bin
```

2.9 修改配置文件
```
cp ./support-files/my-large.cnf /etc/my.cnf
vim /etc/my.cnf
```
在[client]下添加以下内容
```
default-character-set = utf8
```
在[mysqld]下添加以下内容
```
datadir = /data/mysql
character-set-server = utf8
```
:wq 保存退出


2.10 初始化数据库
```
cd /opt/mysql
./bin/mysql_install_db --basedir=/opt/mysql --datadir=/data/mysql --user=mysql
./bin/mysqld_safe --datadir=/data/mysql
```

确认运行后，按可以按CTRL+Z结束
```
ps -ef|grep mysqld
lsof -n | grep jemalloc
```


设置数据库ROOT密码，移除删除临时用户，删除测试数据库等
```
./bin/mysql_secure_installation
```

2.11 登录数据库，查看数据库状态
```
mysql -u root -p
```
```
status;
show engines;
exit;
```

2.12 设置mysql开机自动启动服务
```
vim /etc/systemd/system/mysqld.service
```
录入以下内容
```
[Unit]
Description=MySQL Community Server
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target
Alias=mysql.service

[Service]
User=mysql
Group=mysql

# Execute pre and post scripts as root
PermissionsStartOnly=true

# Needed to create system tables etc.

# Start main service
ExecStart=/opt/mysql/bin/mysqld_safe

# Don't signal startup success before a ping works

# Give up if ping don't get an answer
TimeoutSec=30

Restart=always
PrivateTmp=false
```

:wq 保存
```
systemctl enable mysqld.service
systemctl list-unit-files|grep enabled|grep mysql
systemctl daemon-reload
```

2.13 重启，确认是否已自动启动服务
```
shutdown -r now
systemctl start mysqld.service
systemctl status mysqld.service -l
ps -ef|grep mysqld
lsof -n | grep jemalloc
```

2.14 增加远程访问用户，并且打开防火墙3306端口（不远程连接数据，可忽略）
```
mysql -u root -p
```
```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit;
```
（root是用户名，%是主机名或IP地址，这里的%代表任意主机或IP地址，也可指定唯一的IP地址；密码是MyPassword ）


2.15 防火墙添加3306端口（不远程连接数据，可忽略）
```
iptables -L|grep ACCEPT
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
iptables -L|grep ACCEPT
```
 

3.编译安装Nginx

3.1安装依赖
```
yum install zlib-devel openssl-devel -y
```

3.2 安装Pcre
```
cd /usr/local/src/
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
tar zvxf pcre-8.38.tar.gz
cd pcre-8.38
./configure && make && make install
```

3.3 安装zlib
```
cd /usr/local/src/
wget http://zlib.net/zlib-1.2.8.tar.gz
tar zvxf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure && make && make install
```

3.4 安装openssl (依赖zlib库)
```
openssl version
cd /usr/local/src/
wget https://www.openssl.org/source/openssl-1.0.2e.tar.gz
tar zvxf openssl-1.0.2e.tar.gz
cd openssl-1.0.2e
./config shared zlib --prefix=/usr
make && make install
```

#更新软连接 (如果编译时没有指定-prefix=/usr 需要添加软连接，指定/usr目录时即可忽略以下步骤 )
```
mv /usr/bin/openssl /usr/bin/openssl_bak
mv /usr/include/openssl/ /usr/include/openssl_bak
ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/ssl/include/openssl/ /usr/include/openssl
echo "/usr/local/ssl/lib" >> /etc/ld.so.conf
```

#查看最新版本
```
ldconfig -v | grep ssl
openssl version
```
 

3.6 创建www用户和组，创建www虚拟主机使用的目录，以及Nginx使用的日志目录，并且赋予他们适当的权限
```
groupadd www
useradd -g www www -s /sbin/nologin
mkdir -p /data/www/web
chmod +w /data/www/web
chown -R www:www /data/www/web
```
 

***如果没法创建用户，需要检查SELinux状态是否关闭

3.7 安装nginx
```
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.9.9.tar.gz
tar zvxf nginx-1.9.9.tar.gz
cd nginx-1.9.9
./configure --prefix=/opt/nginx \
--user=www \
--group=www \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-openssl=/usr/local/src/openssl-1.0.2e \
--with-zlib=/usr/local/src/zlib-1.2.8 \
--with-pcre=/usr/local/src/pcre-8.38 \
--with-ld-opt="-ljemalloc" \
--with-http_v2_module 
make && make install
```

3.8 配置nginx,以支持静态网页访问

修改 nginx.conf
```
vim /opt/nginx/conf/nginx.conf
```

修改前面几行为：
```
user www;
worker_processes auto;
error_log logs/error.log crit;
pid logs/nginx.pid;
events{
　　use epoll;
　　worker_connections 65535;
}
```
找到，并修改 root 行的内容
```
location / {
root /data/www/web;
index index.html index.htm;
}
```
:wq 保存退出

 

* 如果需要支持http2，参考以下设置（需要https证书，并且OpenSSL 1.0.2+）
```
server {
　　listen 443 ssl http2;
　　ssl_certificate server.crt;
　　ssl_certificate_key server.key;
　　...
}
```
* 完成后，可以在浏览器中，打开此网站，查看是否已经支持http2

chrome://net-internals/#http2

 

3.9 建立测试首页
```
vim /data/www/web/index.html
```
```
<html>
<head><title>nginx index.html</title></head>
<body>
<h1>index.html</h1>
</body>
</html>
```
保存，退出


3.10 测试和运行
```
cd /opt/nginx
ldconfig
./sbin/nginx -c /opt/nginx/conf/nginx.conf -t
```
如果显示下面信息，即表示配置没问题

nginx: the configuration file /opt/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /opt/nginx/conf/nginx.conf test is successful


查看jemalloc是否生效，需要先启动nginx
```
./sbin/nginx -c /opt/nginx/conf/nginx.conf
lsof -n | grep jemalloc
```
> ```
ginx 2346 root mem REG 253,1 1824470 51571788 /usr/local/lib/libjemalloc.so.1
nginx 2347 www mem REG 253,1 1824470 51571788 /usr/local/lib/libjemalloc.so.1
nginx 2348 www mem REG 253,1 1824470 51571788 /usr/local/lib/libjemalloc.so.1
nginx 2349 www mem REG 253,1 1824470 51571788 /usr/local/lib/libjemalloc.so.1
nginx 2350 www mem REG 253,1 1824470 51571788 /usr/local/lib/libjemalloc.so.1
```

3.11 防火墙添加80端口
```
iptables -L|grep ACCEPT
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
iptables -L|grep ACCEPT
```
 

3.12 浏览器打开

http://192.168.1.117

显示出欢迎内容，则表示成功

 

3.13 作为服务，开机后启动
```
vim /etc/systemd/system/nginx.service
```
增加以下内容
```
[Unit]
Description=The nginx HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/opt/nginx/logs/nginx.pid
ExecStartPre=/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf -t 
ExecStart=/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
:wq 保存退出
```
systemctl enable nginx.service
systemctl list-unit-files|grep enabled|grep nginx
```

3.14 启动服务
```
./sbin/nginx -s stop
systemctl daemon-reload
systemctl start nginx.service
systemctl status nginx.service -l
ps -ef|grep nginx
lsof -n | grep jemalloc
```
 

4 安装PHP

4.1 更新依赖
```
yum install autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers libXpm* gcc gcc-c++ -y
```

4.2 安装libiconv (加强系统对支持字符编码转换的功能)
```
cd /usr/local/src
wget http://ftp.gnu.org/gnu/libiconv/libiconv-1.14.tar.gz
tar zvxf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure --prefix=/usr/local
cd srclib
sed -i -e '/gets is a security/d' ./stdio.in.h
cd ..
make && make install
ln -sf /usr/local/lib/libiconv.so.2 /usr/lib64/
ldconfig
```
 

4.4 安装libmcrypt，libltdl(加密算法库，PHP扩展mcrypt功能对此库有依耐关系，要使用mcrypt必须先安装此库)
```
cd /usr/local/src
wget http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
tar zvxf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure && make && make install
cd libltdl/
./configure --enable-ltdl-install
make && make install
ln -sf /usr/local/lib/libmcrypt.* /usr/lib64/
ln -sf /usr/local/bin/libmcrypt-config /usr/lib64/
ldconfig
```

4.5 安装mhash (hash加密算法库)
```
cd /usr/local/src/
wget http://sourceforge.net/projects/mhash/files/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz
tar zvxf mhash-0.9.9.9.tar.gz
cd mhash-0.9.9.9
./configure && make && make install
ln -sf /usr/local/lib/libmhash.* /usr/lib64/
ldconfig
```

4.6 安装mcrypt
```
cd /usr/local/src/
wget http://sourceforge.net/projects/mcrypt/files/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
tar zvxf mcrypt-2.6.8.tar.gz
cd mcrypt-2.6.8
./configure && make && make install
```

4.6 安装re2c
```
cd /usr/local/src/
wget http://sourceforge.net/projects/re2c/files/0.15.3/re2c-0.15.3.tar.gz
tar zvxf re2c-0.15.3.tar.gz
cd re2c-0.15.3
./configure && make && make install
```


4.7 安装php (已经安装mariadb,nginx,ldap)

4.7.1 创建mysql软连接、ldap软连接
```
mkdir -p /opt/mysql/include/mysql
ln -s /opt/mysql/include/* /opt/mysql/include/mysql/
ln -s /usr/lib64/libldap* /usr/lib
ln -s /usr/lib64/liblber* /usr/lib
```

4.7.2 安装
```
cd /usr/local/src/
wget http://am1.php.net/distributions/php-7.0.2.tar.gz
tar zvxf php-7.0.2.tar.gz
cd php-7.0.2
./configure \
--prefix=/opt/php \
--with-config-file-path=/opt/php/etc \
--with-openssl \
--with-mysqli=shared,mysqlnd \
--with-pdo-mysql=shared,mysqlnd \
--with-iconv-dir=/usr/local \
--with-libxml-dir=/usr \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib \
--with-curl \
--with-mhash \
--with-ldap \
--with-ldap-sasl \
--with-mcrypt \
--with-gd \
--with-xmlrpc \
--with-libdir=/lib/ \
--with-kerberos \
--with-pcre-regex \
--with-zlib-dir \
--with-bz2 \
--with-gettext \
--disable-rpath \
--enable-pdo \
--enable-xml \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--enable-mbregex \
--enable-fpm \
--enable-mbstring \
--enable-gd-native-ttf \
--enable-pcntl \
--enable-sockets \
--enable-zip \
--enable-soap \
--enable-opcache \
--enable-calendar \
--enable-ctype \
--enable-exif \
--enable-session \
--enable-ftp \
make ZEND_EXTRA_LIBS='-liconv'
make install
```

4.7.3 复制配置文件
```
cp php.ini-production /opt/php/etc/php.ini
```

4.7.4 开启系统HugePages
```
sysctl vm.nr_hugepages=512
cat /proc/meminfo | grep Huge

```
> ```
AnonHugePages: 106496 kB
HugePages_Total: 512
HugePages_Free: 504
HugePages_Rsvd: 27
HugePages_Surp: 0
Hugepagesize: 2048 kB
```
 

4.7.5 修改php配置文件，支持ZendOpcache
```
ll /opt/php/lib/php/extensions/no-debug-non-zts-20151012
vim /opt/php/etc/php.ini
```

在文件中搜索; extension_dir = "./" ,并在下面添加以下内容(如果extension_dir已存在，只添加后面的内容)
```
extension_dir = "/opt/php/lib/php/extensions/no-debug-non-zts-20151012/"

zend_extension="opcache.so"
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable=1
opcache.enable_cli=1
opcache.huge_code_pages=1
opcache.file_cache=/tmp
```
:wq 保存退出

 

4.7.6 修改php配置文件，支持pdo_mysql，mysqli
```
vim /opt/php/etc/php.ini
```

在文件中搜索; extension_dir = "./" ,并在下面添加以下内容(如果extension_dir已存在，只添加后面的一行)
```
extension_dir = "/opt/php/lib/php/extensions/no-debug-non-zts-20151012/"

extension = "pdo_mysql.so"
extension = "mysqli.so"
```
:wq 保存退出


4.7.7 安装xdebug扩展（调试PHP用，不需要时可忽略）
```
cd /usr/local/src/
wget http://xdebug.org/files/xdebug-2.4.0rc3.tgz
tar zvxf xdebug-2.4.0rc3.tgz
cd xdebug-2.4.0RC3
/opt/php/bin/phpize
./configure --enable-xdebug --with-php-config=/opt/php/bin/php-config
make && make install
```

修改php配置文件，支持xdebug
```
vim /opt/php/etc/php.ini
```

在文件中搜索; extension_dir = "./" ,并在下面添加以下内容(如果extension_dir已存在，只添加后面的一行)
```
extension_dir = "/opt/php/lib/php/extensions/no-debug-non-zts-20151012/"

[xdebug]
zend_extension = "xdebug.so"
xdebug.remote_enable=1
xdebug.remote_connect_back=on
xdebug.remote_port=8080
xdebug.idekey=PHPSTORM
xdebug.remote_autostart=1
```
:wq 保存退出

 

4.7.8 安装memcahced扩展 (需要 libmemcached 库)
```
cd /usr/local/src
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
tar zvxf libmemcached-1.0.18.tar.gz
cd libmemcached-1.0.18
./configure --with-memcached --prefix=/opt/libmemcached
make && make install
```
```
cd /usr/local/src
yum install git
git clone https://github.com/rlerdorf/php-memcached.git
cd php-memcached
git checkout php7
/opt/php/bin/phpize
./configure --with-php-config=/opt/php/bin/php-config --with-libmemcached-dir=/opt/libmemcached
make && make install
ll /opt/php/lib/php/extensions/no-debug-non-zts-20151012/
```

修改php配置文件，支持memcache
```
vim /opt/php/etc/php.ini
```
在文件中搜索; extension_dir = "./" ,并在下面添加以下内容(如果extension_dir已存在，只添加后面的一行)
```
extension_dir = "/opt/php/lib/php/extensions/no-debug-non-zts-20151012/"

extension = "memcached.so"
```
 

4.7.9 安装redis扩展 
```
cd /usr/local/src
yum install git 
git clone https://github.com/phpredis/phpredis/
cd phpredis
git checkout php7
/opt/php/bin/phpize
./configure --with-php-config=/opt/php/bin/php-config
make && make install
ll /opt/php/lib/php/extensions/no-debug-non-zts-20151012/
```

修改php配置文件，支持redis
```
vim /opt/php/etc/php.ini
```
在文件中搜索; extension_dir = "./" ,并在下面添加以下内容(如果extension_dir已存在，只添加后面的一行)
```
extension_dir = "/opt/php/lib/php/extensions/no-debug-non-zts-20151012/"

extension = "redis.so"
```
 

4.7.10 安装php-fpm
```
cp /opt/php/etc/php-fpm.conf.default /opt/php/etc/php-fpm.conf
cp /opt/php/etc/php-fpm.d/www.conf.default /opt/php/etc/php-fpm.d/web.conf
vim /opt/php/etc/php-fpm.conf
```

修改内容，并且让其它生效
```
[global]
pid = run/php-fpm.pid

error_log = log/php-fpm.log

emergency_restart_threshold = 10

emergency_restart_interval = 1m

process_control_timeout = 5s
```
:wq 保存退出
```
vim /opt/php/etc/php-fpm.d/web.conf
```
修改内容，并且让其它生效
```
user = www

group = www

pm.max_children = 35

pm.start_servers = 20

pm.min_spare_servers = 5

pm.max_spare_servers = 35
```
:wq 保存退出


> php-fpm.conf 重要参数详解
>
> pid = run/php-fpm.pid
> pid设置，默认在安装目录中的var/run/php-fpm.pid，建议开启
>
> error_log = log/php-fpm.log
> 错误日志，默认在安装目录中的var/log/php-fpm.log
>
> log_level = notice
> 错误级别. 可用级别为: alert（必须立即处理）, error（错误情况）, warning（警告情况）, notice（一般重要信息）, debug（调试信息）. 默认: notice.
>
> emergency_restart_threshold = 60
> emergency_restart_interval = 60s
> 表示在emergency_restart_interval所设值内出现SIGSEGV或者SIGBUS错误的php-cgi进程数如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个选项一般保持默认值。
>
> process_control_timeout = 0
> 设置子进程接受主进程复用信号的超时时间. 可用单位: s(秒), m(分), h(小时), 或者 d(天) 默认单位: s(秒). 默认值: 0.
> daemonize = yes
> 后台执行fpm,默认值为yes，如果为了调试可以改为no。在FPM中，可以使用不同的设置来运行多个进程池。 这些设置可以针对每个进程池单独设置。
>
> listen = 127.0.0.1:9000
> fpm监听端口，即nginx中php处理的地址，一般默认值即可。可用格式为: 'ip:port', 'port', '/path/to/unix/socket'. 每个进程池都需要设置.
>
> listen.backlog = -1
> backlog数，-1表示无限制，由操作系统决定，此行注释掉就行。backlog含义参考：http://www.3gyou.cc/?p=41
>
> listen.allowed_clients = 127.0.0.1
> 允许访问FastCGI进程的IP，设置any为不限制IP，如果要设置其他主机的nginx也能访问这台FPM进程，listen处要设置成本地可被访问的IP。默认值是any。每个地址是用逗号分隔. 如果没有设置或者为空，则允许任何服务器请求连接
>
> listen.owner = www
> listen.group = www
> listen.mode = 0666
> unix socket设置选项，如果使用tcp方式访问，这里注释即可。
>
> user = www
> group = www
> 启动进程的帐户和组
>
> pm = dynamic #对于专用服务器，pm可以设置为static。
> 如何控制子进程，选项有static和dynamic。如果选择static，则由pm.max_children指定固定的子进程数。如果选择dynamic，则由下开参数决定：
> pm.max_children #，子进程最大数
> pm.start_servers #，启动时的进程数
> pm.min_spare_servers #，保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程
> pm.max_spare_servers #，保证空闲进程数最大值，如果空闲进程大于此值，此进行清理
>
> pm.max_requests = 1000
> 设置每个子进程重生之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 '0' 则一直接受请求. 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 默认值: 0.
>
> pm.status_path = /status
> FPM状态页面的网址. 如果没有设置, 则无法访问状态页面. 默认值: none. munin监控会使用到
>
> ping.path = /ping
> FPM监控页面的ping网址. 如果没有设置, 则无法访问ping页面. 该页面用于外部检测FPM是否存活并且可以响应请求. 请注意必须以斜线开头 (/)。
>
> ping.response = pong
> 用于定义ping请求的返回相应. 返回为 HTTP 200 的 text/plain 格式文本. 默认值: pong.
>
> request_terminate_timeout = 0
> 设置单个请求的超时中止时间. 该选项可能会对php.ini设置中的'max_execution_time'因为某些特殊原因没有中止运行的脚本有用. 设置为 '0' 表示 'Off'.当经常出现502错误时可以尝试更改此选项。
>
> request_slowlog_timeout = 10s
> 当一个请求该设置的超时时间后，就会将对应的PHP调用堆栈信息完整写入到慢日志中. 设置为 '0' 表示 'Off'
>
> slowlog = log/$pool.log.slow
> 慢请求的记录日志,配合request_slowlog_timeout使用
>
> rlimit_files = 1024
> 设置文件打开描述符的rlimit限制. 默认值: 系统定义值默认可打开句柄是1024，可使用 ulimit -n查看，ulimit -n 2048修改。
>
> rlimit_core = 0
> 设置核心rlimit最大限制值. 可用值: 'unlimited' 、0或者正整数. 默认值: 系统定义值.
>
> chroot =
> 启动时的Chroot目录. 所定义的目录需要是绝对路径. 如果没有设置, 则chroot不被使用.
>
> chdir =
> 设置启动目录，启动时会自动Chdir到该目录. 所定义的目录需要是绝对路径. 默认值: 当前目录，或者/目录（chroot时）
>
> catch_workers_output = yes
> 重定向运行过程中的stdout和stderr到主要的错误日志文件中. 如果没有设置, stdout 和 stderr 将会根据FastCGI的规则被重定向到 /dev/null . 默认值: 空.


4.7.11 配置nginx，支持php
```
mkdir -p /opt/nginx/conf/vhosts
vim /opt/nginx/conf/nginx.conf
```
删除原来内容，替换成以下内容：

```
user www;

worker_processes auto;

error_log logs/error.log crit;

pid logs/nginx.pid;

events {
use epoll;
worker_connections 1024;
}

http {
include mime.types;
default_type application/octet-stream;

# log_format main '$remote_addr - $remote_user [$time_local] "$request" '
# '$status $body_bytes_sent "$http_referer" '
# '"$http_user_agent" "$http_x_forwarded_for"';

# access_log logs/access.log main;

sendfile on;
# tcp_nopush on;

# keepalive_timeout 0;
keepalive_timeout 65;

# gzip on;

include /opt/nginx/conf/vhosts/*.conf;
}
```
:wq 保存退出


创建网站配置文件（网站thinkphp,网站目录/data/www/web）
```
vim /opt/nginx/conf/vhosts/web.conf
```
添加以下内容
```
server {
listen 80;
server_name 192.168.1.117;

root /data/www/web;

location / {
index index.html index.php;

# for bowers thinkphp without /index.php path
if (!-e $request_filename) {
rewrite ^/(.*)$ /index.php/$1 last;
break;
}
}
location ~ \.php {
root /data/www/web;
fastcgi_pass 127.0.0.1:9000;
fastcgi_index index.php;
fastcgi_param SCRIPT_FILENAME /data/www/web$fastcgi_script_name;
include fastcgi_params;

# for thinkphp pathinfo mode
set $path_info "";
set $real_script_name $fastcgi_script_name;
if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
set $real_script_name $1;
set $path_info $2;
}
fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
fastcgi_param SCRIPT_NAME $real_script_name;
fastcgi_param PATH_INFO $path_info;
}
}
```
:wq 保存退出

4.7.12 将php-fpm服务加到开机启动服务
```
cp /usr/local/src/php-7.0.2/sapi/fpm/php-fpm.service /etc/systemd/system/
vim /etc/systemd/system/php-fpm.service
```
删除原来内容，替换成以下内容
```
[Unit]
Description=The PHP FastCGI Process Manager
After=syslog.target network.target

[Service]
Type=simple
PIDFile=/opt/php/var/run/php-fpm.pid
ExecStart=/opt/php/sbin/php-fpm --nodaemonize --fpm-config /opt/php/etc/php-fpm.conf
ExecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target
```
保存,退出
```
systemctl enable php-fpm.service
systemctl list-unit-files|grep enabled|grep php-fpm
systemctl daemon-reload
systemctl start php-fpm.service
systemctl status php-fpm.service -l
```

4.7.13 编写测试页面
```
mkdir -p /data/www/web
vim /data/www/web/index.php
```
输入以下内容
```
<html>
<head><title>hello php</title></head>
<body>
<?php phpinfo();?>
</body>
</html>
```
:wq 保存退出
```
systemctl restart nginx
systemctl restart php-fpm
```
浏览器访问：http://192.168.1.117/index.php

 

5 安装memcached服务

5.1 安装memcached (需要libevent库)
```
cd /usr/local/src
wget http://www.memcached.org/files/memcached-1.4.25.tar.gz
tar zvxf memcached-1.4.25.tar.gz
cd memcached-1.4.25
./configure --prefix=/opt/memcached --with-libevent=/usr
make && make install
```

5.2 设置用户、运行参数，以及开机启动
```
groupadd memcached
useradd -g memcached memcached -s /sbin/nologin
vim /etc/sysconfig/memcached
```
添加以下内容
```
PORT="11211"
IP="127.0.0.1"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS=""
```
:wq 保存退出

```
vim /etc/systemd/system/memcached.service
```
添加以下内容
```
[Unit]
Description=Memcached
Before=httpd.service
After=network.target

[Service]
Type=simple
EnvironmentFile=-/etc/sysconfig/memcached
ExecStart=/opt/memcached/bin/memcached -l $IP -u $USER -p $PORT -m $CACHESIZE -c $MAXCONN $OPTIONS

[Install]
WantedBy=multi-user.target
```

:wq 保存退出
```
systemctl enable memcached.service
systemctl list-unit-files|grep enabled|grep memcached
systemctl daemon-reload
systemctl start memcached.service
systemctl status memcached.service -l
```

5.3 防火墙添加12121端口(如果不需要外部访问，可忽略)
```
iptables -L|grep ACCEPT
firewall-cmd --zone=public --add-port=11211/tcp --permanent
firewall-cmd --reload
iptables -L|grep ACCEPT
```

5.4 测试PHP支持（PHP需要安装memcached扩展）
```
vim /data/www/web/m.php
```
添加以下内容
```
<?php
echo "set and get memcached:";

$mem = new Memcached;
$mem->addServer('127.0.0.1',11211);
$mem->set('test',"hello memcached");

$val=$mem->get('test');

echo $val;
?>
```
:wq 保存退出

浏览器访问：http://192.168.1.117/m.php
