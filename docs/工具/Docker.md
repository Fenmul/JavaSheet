### Docker

#### 1. docker 安装
centos 7 中 docker 的安装步骤

1. 更新 yum 包
```
sudo yum update
```
2. 安装软件包
```
sudo yum install -y yum -utils device-mapper-persistent-data lvm2
```
3. 设置 yum 源为 阿里云
```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
4. 安装 docker
```
sudo yum install docker-ce
```
5. 检查 docker
```
docker -v
```
6. 设置 ustc 镜像
```
vi /etc/docker/daemon.json
{
    "registry-mirrors": ["https://jpabnqm3.mirror.aliyuncs.com"]
}
```
#### 2. 常用命令 
```
# 启动
systemctl start docker
# 查看状态
systemctl status docker
# 停止
systemctl stop docker
# 重启
systemctl restart docker
# 开机自启动
systemctl enable docker
```

查看概要信息

```
docker info
```

查看帮助文档

```
docker --help
```

##### 2.1 镜像相关

查看镜像

```
docker images
```
查询镜像

```
docker search 镜像名称
```

拉取镜像

```
docker pull 镜像名称
```

删除镜像

```
docker rmi 镜像ID
```

##### 2.2 容器相关

查看运行中的容器

```
docker ps
```

查看所有的容器

```
docker ps -a
```

查看最后一次运行的容器

```
docker ps -l
```

查看停止的容器

```
docker ps -f status=exited
```

**创建容器**

```
docker run
```

-i：表示运行容器

-t：表示容器启动后进入命令行，分配一个伪终端

--name：为容器命名

-v：表示目录映射关系，挂载目录使宿主机和容器目录保持一致

-d：在 run 后面加上参数，会创建一个守护式容器进程

-p：表示端口映射



- 交互式

```
docker run -it --name=mycentos centos:7 /bin/bash
```

启动 centos7 并且命名为 mycentos 进入目录 /bin/bash

exit 退出，退出后就会关闭

- 守护式

```
docker run -di--name=mycentos2 centos:7 /bin/bash

# 进入centos
docker exec -it mycentos2 /bin/bash
```

exit 退出后仍然开启



**停止与创建**

```
docker stop 容器名称（ID）
docker start 容器名称（ID）
```

查看容器 ip

```
docker inspect 容器名
docker inspect --format='{{.Networksetting.IPAddress}}' mycentos
```

删除容器

```
docker -rm 容器名
```

#### 3. 部署 Mysql
```
# 拉取镜像
docker pull centos/mysql-57-centos7
# 创建容器,操作宿主机的 33306 去操作容器 3306 的端口
docker run -di --name=fox_mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
```

通过宿主机的端口去映射到 mysql 中去
#### 4. 部署 Redis 
```
docker pull redis
docker run -di --name=fox_redis -p 6379:6379 redis
```
#### 5. 迁移与备份 

```
# 保存为镜像
docker commit fox_redis fox_redis_i
# 镜像备份
docker save -o fox_redis.tar fox_redis_i
# 镜像加载
docker load -i fox_redis.tar
```

#### Dockerfile

dockerfile 将一系列的命令应用于基础镜像并创建一个新的镜像 vi Dockerfile

![image-20200428002738749](/Docker.assets/image-20200428002738749.png)
```
FROM centos:7
MAINTAINER fox
WORKDIR /usr
RUN mkdir /usr/local/java
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
ENV JAVA_HOME=/usr/local/java/jdk1.8.0_171
ENV JRE_HOME= $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/bin/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
```

buid -t='jdk1.8' .

镜像名称是 jdk1.8 dockerfile 的文件位置在当前文件夹下所以是 . 注意空格和点不能少

