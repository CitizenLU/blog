

Linux开放端口号

sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload



[centos7 安装mysql8.0 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1759772)

```shell
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update
yum install mysql-server
https://dev.mysql.com/downloads/file/?id=508899


sudo rpm -Uvh platform-and-version-specific-package-name.rpm

sudo rpm -Uvh mysql80-community-release-el6-n.noarch.rpm
yum repolist all | grep mysql

# 1、添加MySQL Yum Repository
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
rpm -ivh mysql80-community-release-el7-3.noarch.rpm
yum install mysql-community-server -y

# 2、开启防火墙3306
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload

# 3、设置开机启动
systemctl enable mysqld
systemctl daemon-reload

# 启动mysql
service mysqld start

# 关闭mysql
service mysqld stop

# 查看启动状态
service mysqld status

# 重启mysql
service mysqld restart

# 用户名、密码修改，开启本地、远程访问，创建数据库
# 进入mysql
mysql -u root -p
# 输入密码
mysql> use mysql;

# 查看默认密码
sudo grep 'temporary password' /var/log/mysqld.log

# 修改密码
mysql> UPDATE user SET password=password("密码") WHERE user='root';   # mysql 7.0
mysql> alter user 'root'@'localhost' identified with mysql_native_password by '密码'; # mysql 8.0
mysql> flush privileges;

# 开启本地访问
mysql> grant all privileges on *.* to root@"localhost" identified by "密码";

# 开启远程访问
mysql> grant all privileges on *.* to root@"%" identified by "密码"; # mysql 7.0
mysql> UPDATE user SET host = '%' WHERE user ='root'; # mysql 8.0
mysql> flush privileges; # 刷新MySQL的系统权限相关表

# 退出
mysql> exit


# 允许输入控制
mysql> SET GLOBAL sql_mode = '';

# 创建数据库表
mysql> CREATE DATABASE IF NOT EXISTS doctor DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
mysql> USE doctor;
```



使用Navicat连接到mysql数据库



![image-20211226024633070](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211226024633070.png)

新建一个数据库

![image-20211226025029879](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211226025029879.png)











```
docker run -d -v /srv/jellyfin/config:/config -v /srv/jellyfin/cache:/cache -v /data_disk/media:/media --net=host jellyfin/jellyfin:latest
```



