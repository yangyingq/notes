**mysqldump 导出指定库**

mysqldump 导数据经常使用，指定数据库，指定表，指定条件，可以这么执行

```
mysqldump -h192.168.11.10 -uroot -pcentos  --databases clue --tables  –progress-report -t  --where='id=1'  >/tmp/clue_outcall_tasks.sql

#参数解释：

--databases 指定数据库
-–progress-report 进度报告显示
--tables 指定表
--where='' 是筛选条件
-t只导数据，不导结构
-d只导结构，不导数据
```



vmware centos 磁盘扩容

```
https://blog.csdn.net/qq_45562973/article/details/124728786
```



# 环境篇

## 一、linux 安装nginx

### 1.安装依赖包

```
//一键安装上面四个依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```



### 2.下载并解压安装包

```
//创建一个文件夹
cd /usr/local
mkdir nginx
cd nginx
//下载tar包
wget http://nginx.org/download/nginx-1.13.7.tar.gz
tar -xvf nginx-1.13.7.tar.gz
```

### 3.安装nginx

```
//进入nginx目录
cd /usr/local/nginx
//进入目录
cd nginx-1.13.7
//执行命令 考虑到后续安装ssl证书 添加两个模块
./configure --with-http_stub_status_module --with-http_ssl_module
//执行make命令
make
//执行make install命令
make install
```

### 4.启动nginx服务

```
 ​​​​​​​/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```



#### 5.重启nginx

```
/usr/local/nginx/sbin/nginx -s reload
```



### 6.查看端口是否开放

```
firewall-cmd --zone=public --list-ports
```



### 7.设置服务器启动

```
vim /etc/init.d/nginx
```

将下列代码填入

```
#!/bin/bash
# chkconfig: - 85 15
#nginx安装位置
PATH=/usr/local/nginx
DESC="nginx daemon"
NAME=nginx
DAEMON=$PATH/sbin/$NAME
CONFIGFILE=$PATH/conf/$NAME.conf
PIDFILE=$PATH/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
set -e
[ -x "$DAEMON" ] || exit 0
do_start() {
$DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}
do_stop() {
$DAEMON -s stop || echo -n "nginx not running"
}
do_reload() {
$DAEMON -s reload || echo -n "nginx can't reload"
}
case "$1" in
start)
echo -n "Starting $DESC: $NAME"
do_start
echo "."
;;
stop)
echo -n "Stopping $DESC: $NAME"
do_stop
echo "."
;;
reload|graceful)
echo -n "Reloading $DESC configuration..."
do_reload
echo "."
;;
restart)
echo -n "Restarting $DESC: $NAME"
do_stop
do_start
echo "."
;;
*)
echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
exit 3
;;
esac
exit 0
```

```
#设置权限 chmod 755 /etc/init.d/nginx
#设置开机自启 chkconfig nginx on
#nginx not found command 解决方案
vim /etc/profile
在文件末尾添加：export PATH="nginx 命令地址":$PATH
配置生效：source /etc/profile
```



二、linux 安装PHP

1.下载php压缩包并上传linux : php7.2.4.tar.gz

2.创建路径 并将压缩包移动到该路径下,解压。

```
mkdir  /usr/local/php
cp /php7.2.4.tar.gz /usr/local/php
tar -xvf php7.2.4.tar.gz
```

 3.下载安装依赖

```
yum -y install gcc openssl openssl-devel curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel pcre pcre-devel libxslt libxslt-devel bzip2 bzip2-devel
```

4.编译安装php

```
cd php7.2.4

./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-pdo-mysql --with-mysqli --with-gd --with-zlib --with-mcrypt --enable-fpm

 
make && make install

安装完成后，别忘记 make test

有如下报错 configure: error: mcrypt.h not found. Please reinstall libmcrypt.  

解决

yum install -y epel-release

yum install -y libmcrypt-devel

重新编译
```

5.复制配置文件

```
1）配置php.ini，这是php的配置文件：

    cp **/etc/php/php-7.2.4/php.ini-development** /usr/local/php/lib/php.ini  #**** 内的内容是解压的php文件地址

2）配置php-fpm.conf，这是php-fpm配置文件：

     cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf

3）配置www.conf，配置用户的文件：

    cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf

4）将php-fpm启动文件复制到init.d文件夹中一份方便启动php：

     cp /usr/local/php/sbin/php-fpm /etc/init.d/php-fpm
```

6.启动php 确认php正在运行

```
执行命令/etc/init.d/php-fpm即可

查看是否启动：ps -ef |grep php
```



输入php -v显示not command 时：

```
export PATH=$PATH:/usr/local/php/bin
```



设置nginx 配置支持解析php文件

```
vim /usr/local/nginx/conf/nginx.conf

将转发php 处理打开（将/script*** 改为$document_root）
 location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

```

7 .linux 安装自带php扩展（这里以curl扩展为例）

```
#进入之前下载解压的php-7.2.4文件夹下
cd /data/php-7.2.4/ext/curl

/usr/local/php/bin/phpize

#执行配置映射
./configure --with-php-config=/usr/local/php/bin/php-config

#安装之清除安装配置
make clean

#重新安装
make && make install
```



linux 环境安装mysql5.7

1.查看是否安装mysql 

```
yum list installed | grep mysql
```

2.卸载mysql 及其依赖

```
yum -y remove mysql-libs.x86_64
```

3.下载wget命令

```
yum install wget -y
```

4.给centos添加rpm源，并且选择比较新的源

```
wget dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
```

5.安装下好的rpm文件

```
yum install mysql-community-release-el6-5.noarch.rpm -y
```

6.安装成功后，会在/etc/yum.repo.d/下面新增两个文件

```
mysql-community.repo
mysql-community-source.repo
```

 7、修改mysql-community.repo文件

```
vi mysql-community.repo

enabled = 1  => enabled = 0
baseurl****el/6/$base*** => baseurl****el/7/$base***
enabled = 0  => enabled = 1 表示安装5.7版本
gpgcheck = 1  => 0  
```

8、使用yum安装mysql

```
yum install mysql-community-server -y
```



9、启动mysql服务

```
systemctl mysqld start
```

10、设置mysql开机启动

```
chkconfig mysqld on
```

11、从mysqld.log文件中，查看mysql临时密码

```
grep "password" /var/log/mysqld.log
```

12、进入mysql修改密码验证策略(不更改，可能修改的密码通不过)，然后更改root用户密码

```
set global validate_password_policy=0;
set global validate_password_length=4;
alter user 'root'@'localhost' identified by '123456';
```

13、设置数据库用户在所有ip下都可以访问，以下用root用户示例：

```
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
```

14、刷新mysql的系统权限相关表

```
flush privileges;
```

15、查询3306端口是否开启：

```
firewall-cmd --query-port=3306/tcp
```

16、在防火墙上，添加需要开放的3306端口(远程连接)：

```
firewall-cmd --add-port=3306/tcp --permanent
```

17、 根据端口号查看进程状态

```
netstat -anp 端口号
```

安装 mysql 工具 percona-toolkit

```
 
 wget https://www.percona.com/downloads/percona-toolkit/3.1.0/binary/redhat/7/x86_64/percona-toolkit-3.1.0-2.el7.x86_64.rpm
 
 # 安装依赖
 yum install perl-DBI perl-DBD-MySQL perl-Digest-MD5 perl-IO-Socket-SSL perl-TermReadKey -y
 
 #安装
 rpm -ivh percona-toolkit-3.0.13-1.el6.x86_64.rpm
 
 #验证是否安装成功
 pt-query-digest --help
 
```

pt-archiver 的简单使用

```
//归档删除
pt-archiver --source h=192.168.182.136,P=3306,u=root,p=hckj@\).2017,D=platform_system_jsyd_v1,t=user_watches --dest  h=192.168.182.136,P=3306,u=root,p=hckj@\).2017,D=platform_system_jsyd_v1,t=user_watches_2022_04 --charset=UTF8 --where 'created_at BETWEEN "2022-04-01 00:00:00" AND "2022-05-01 00:00:00"' --progress 10000 --limit=1000 --txn-size 1000 --bulk-insert --statistics --bulk-delete --no-check-charset 


//归档不删除
pt-archiver --source h=127.0.0.1,P=3306,u=root,p='123456',D=platform_system_jsyd_v1,t=user_watches --dest h=127.0.0.1,P=3306,u=root,p=123456,D=platform_system_jsyd_v1,t=user_watches_2022 --charset=UTF8 --where 'created_at BETWEEN "2021-06-01 00:00:00" AND "2022-01-01 00:00:00"' --progress 10000 --limit=1000 --txn-size 1000 --bulk-insert --statistics --no-delete --no-check-charset 

--progress 每多少行输出一次信息

--limit 每次取1000行数据进行处理

--txn-size  多少行数据提交一次事务

--bulk-delete 删除旧表数据

--bulk-insert 插入数据不删除
```



linux (centos) 安装docker 

1.卸载旧的版本

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2.安装yum包

```
sudo yum install -y yum-utils
```

3.设置阿里镜像仓库

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#设置阿里云镜像加速
{
	"reigstry-mirrors": ["阿里镜像加速地址"] #进入阿里云，打开菜单，找到镜像服务
}
```

4.更新yum软件包索引

```
yum makecache fast
```

5.安装docker-ce 社区版

```
sudo yum install docker-ce docker-ce-cli containerd.io
```

6.启动docker

```
systemctl start docker
```

7.docker 拉取镜像

```
docker pull nginx 
```

8.容器操作

移除

```
停止容器：docker stop b628a3549580
移除容器：docker rm b628a3549580
```

查看容器挂载

```
docker inspect container_id | grep Mounts -A 20
```

查看容器ip

```
docker inspect nginx | grep IPAddress
```

保存镜像

```
docker save -o 要保存的文件名  要保存的镜像
```

启动容器

```
docker run --name nginx -p 80:80 -p 8090:8090 -p 8091:8091 -v /data/server/nginx/nginx.conf:/etc/nginx/nginx.conf -v /data/wwwroot/:/etc/nginx/www/html/ -v /data/server/nginx/conf.d/:/etc/nginx/conf.d/ --link php7.2.4 --privileged=true -d nginx

docker run -d -p 9000:9000 --name php7.2.4 -v /data/wwwroot:/var/www/html -v /data/server/php/php.ini:/usr/local/etc/php/php.ini --privileged=true   php:7.2.4-fpm

docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
修改mysql 乱码

查看mysql 编码：show variables like '%char%';

修改mysql 容器中 /etc/mysql/mysql.cnf ,添加以下内容
show varia

[mysql]
default-character-set=utf8
 
[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
 
[client]
default-character-set=utf8


grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option; #进入mysql 后设置远程访问

-v 挂载目录 
--privileged=true 使用该参数，container内的root拥有真正的root权限。
否则，container内的root只是外部的一个普通用户权限。
进入容器后执行 nginx -s reload 可重新启动nginx

启动完之后修改nginx.conf中php转发配置
   location ~ \.php$ {
        fastcgi_pass   php7.2.4:9000; #转发至php容器9000端口，如果不通需要 将nginx容器 link php容器
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /var/www/html/$fastcgi_script_name;
        include        fastcgi_params;
    }
```

查看容器日志

```
docker inspect --format '{{.LogPath}}' mysql || docker logs 容器id
```

复制主机文件进入容器

```
docker cp '源文件' 容器id:容器内文件夹
```



linux 部署git

```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker #安装git环境

https://github.com/git/git/releases #下载git包

#解压后进入文件安装
make prefix=/home/soft/git all 
make prefix=/home/soft/git install

#查看git 
whereis git 
git --version
```



linux docker 安装composer

```
docker exec -it cb6c1fe83bff(php容器ID) bash #进入php容器

#安装composer

curl -sS https://getcomposer.org/installer | php
 
mv composer.phar /usr/local/bin/composer
 
#换阿里云源(可选)
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer
 
exit

在容器内执行composer update
```



docker 安装php-gd库扩展

```
apt update #更新软件源
apt install -y libwebp-dev libjpeg-dev libpng-dev libfreetype6-dev #安装各种库
docker-php-source extract #解压源码
cd /usr/src/php/ext/gd #gd源码文件夹
docker-php-ext-configure gd --with-webp-dir=/usr/include/webp --with-jpeg-dir=/usr/include --with-png-dir=/usr/include --with-freetype-dir=/usr/include/freetype2 #准备编译
docker-php-ext-install gd #编译安装
php -m | grep gd

```

mysql 8.0 修改密码

```
#由于mysql 8.0以上的密码验证方式不同，转变为传统的数据库密码验证方式
alter user 'root'@'%' identified with mysql_native_password by 'your password';
```

mysql.cnf 配置

```
[mysqld]
#设置文件编码
character-set-server=utf8mb4
#binlog日志的基本文件名，需要注意的是启动mysql的用户需要对这个目录(/usr/local/var/mysql/binlog)有写入的权限
log_bin=/var/lib/mysql/mysql-bin
## 配置binlog日志的格式
binlog_format = ROW
## 配置 MySQL replaction 需要定义，不能和 canal 的 slaveId 重复
server-id=123
## 设置中继日志的路径
relay_log=/usr/local/var/mysql/relaylog/mysql-relay

[client]
#客户端连接编码
default-character-set=utf8mb4 

[mysql]
default-character-set=utf8mb4
```

ubantu 安装libxml2

apt install lib
