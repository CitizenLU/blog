

# docker的一些概念



**镜像（image）：**

面向对象中的类

**容器（container）：**

面向对象中的对象

启动，停止，删除。简易的Linux系统

**仓库（repository）：**

存放镜像的地方。分为公有仓库和私有仓库。



# 安装

```shell
# 1、卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 2、需要的安装包
yum install -y yum-utils

# 3、设置镜像仓库，默认是国外的
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# 使用阿里云镜像安装
yum-config-manager \
    --add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新yum软件包索引
yum makecache fast

# 4、安装docker引擎 docker-ce社区版 ee企业版
yum install docker-ce docker-ce-cli containerd.io

# 5、启动docker
systemctl start docker

# 6、docker version看是否安装成功

# 7、hello world
docker run hello-world

# 8、查看下载的镜像
docker images

[root@192 /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
mysql         latest    ecac195d15af   12 days ago   516MB
hello-world   latest    feb5d9fea6a5   5 weeks ago   13.3kB
centos        latest    5d0da3dc9764   6 weeks ago   231MB

```





# 命令



```shell
# 查看所有docker容器
docker ps -a
```

















