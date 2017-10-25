docker常用命令
====

查看docker容器IP 
docker-machine ip default

运行应用并映射到主机
docker run -d -P xxx  (端口号随机)
docker run -d -p 5000:5000 xxx 使用-p绑定指定端口

查看正在运行中的容器
docker ps

查看容器端口映射情况
docker port xxx

查看web应用日志
docker logs -f xxx

查看容器进程
docker top xxx

查看docker的底层信息,容器配置和状态信息
docker inspect xxx

停止、重启、移除web应用容器(当移除容器是，容器必须是停止状态)
docker stop/start/rm xx

###############################################

镜像
====

列出本地主机上的镜像
docker images

使用某镜像来运行容器(如果未指定版本，默认会使用最新版latest)
docker run -t -i ubuntu:15.10 /bin/bash

获取新镜像
docker pull ubuntu:13.10

查找镜像(https://hub.docker.com)
docker search httpd

构建镜像
创建一个Dockerfile文件，例如
ROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
每一个指令都会在镜像上创建一个新的层，每一个指定的前缀必须是大写
FROM 指定使用哪个镜像源
RUN 指令docker在镜像内执行命令

然后docker build构建一个镜像(Dockerfile文件位于当前目录，也可以指定绝对路径)
docker build -t xxxx .

指定容器绑定的网络地址
docker run -d -p 127.0.0.1:5001:5002 training/webapp python app.py

######################################################

删除none镜像
docker images|grep none|awk '{print $3 }'|xargs docker rmi
docker rmi -f xxx


启动应用并挂载
docker run -d --name=tomcat_test -it -v /tomcat_self/test:/tomcat/webapps -p 50080:8080 tomcat:test1

--name=tomcat_test: 是给容器自定义一个名称，用来区分业务，为测试环境
-v:给测试的容器指定挂载的本地目录为/ylkj/test 以后测试的包就发布到Host主机的这个目录下面 /tomcat/webapps是tomcat的运行环境目录 冒号":"前面的目录是宿主机目录，后面的目录是容器内目录


docker run -d -i -t centos /bin/bash 这样centos一直停留在后台运行
docker attach <ContainerID>  进入centos
