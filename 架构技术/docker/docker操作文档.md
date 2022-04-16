# docker安装

**1.第一步如果系统内有老版本的docker，我们需要先删除之前的docker以及相关依赖。**

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

**2.安装社区版docker**

*2.1 安装所需要的包。*

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

*2.2 设置稳定的存储库*

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

**3.安装最新的社区版docker**

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

**4.启动docker服务**

```shell
sudo systemctl start docker
```

**5.运行一个 `hello-world`镜像，来检测社区版的docker是否安装成功**

```shell
sudo docker run hello-world
```

出现了`Hello from Docker!`表示安装成功。



# docker基础命令

**基本操作：**

docker —help — 万能的help

docker info — 查看docker容器信息

docker version — 查看docker容器版本



**下载镜像**

$ docker pull nginx  — 下载镜像

$ docker pull nginx.latest  — 下载最新镜像

docker pull -a redis — 下载仓库所有Redis镜像



**搜索镜像**

docker search mysql — 搜索仓库镜像

--filter=stars=600：只显示 starts>=600 的镜像

--no-trunc 显示镜像完整 DESCRIPTION 描述

--automated ：只列出 AUTOMATED=OK 的镜像

```
docker search --filter=stars=600 mysql
docker search --no-trunc mysql
docker search --automated mysql
```



**查看镜像**

$ docker images  — 查看镜像

docker images -a — 含中间映像层

docker images -q — 只显示镜像ID

docker images -qa — 含中间映像层

docker images —digests — 显示镜像摘要信息(DIGEST列)

docker images --no-trunc — 显示镜像完整信息



**历史记录**

docker history -H redis — 显示指定镜像的历史创建

参数：-H 镜像大小和日期，默认为true；--no-trunc  显示完整的提交记录；-q  仅列出提交记录ID



**运行镜像**

$ docker run -d -p 80:80 nginx — 启动容器

docker run -itd --privileged -p 60022:22 -p 81:80 ansible/centos7-ansible:latest — docker启动centos

$ docker ps  — 查看容器



**查看容器ip**

`docker inspect $(docker ps -aq)|grep -i ipaddr|tail -1`

`docker inspect 容器id|grep -i ipaddr|tail -1`



**修改容器属性：**

$ docker exec -it b2 bash  — 进入容器内部

root@b2df2f8c68ca:/# cd /usr/share/nginx/html/

root@b2df2f8c68ca:/usr/share/nginx/html# ls

root@b2df2f8c68ca:/usr/share/nginx/html# cat index.html 

root@b2df2f8c68ca:/usr/share/nginx/html# echo hello > index.html 

root@b2df2f8c68ca:/usr/share/nginx/html# cat index.html 

root@b2df2f8c68ca:/usr/share/nginx/html# exit  — 退出容器内部

$ docker rm -f b2  — 删除容器

$ docker commit d1 m1  —  提交容器保存为镜像



**dockerfile脚本：**

$ vim dockerfile — 编写脚本

FROM ngnix
ADD ./ /usr/share/nginx/html/

$ vim index.html

$ docker build -t m2 . 

`docker build -f /docker/dockerfile/mycentos -t mycentos:1.1` — 构建docker镜像



**镜像打包：**

$ docker save m2 > 1.tar  — 打包镜像

$ docker rm -f 32

$ docker rmi m2  — 删除镜像

docker rmi -f redis — 强制删除(针对基于镜像有运行的容器进程)

docker rmi -f redis tomcat nginx — 不同镜像间以空格间隔

docker rmi -f $(docker images -q) — 删除本地全部镜像

$ docker load < 1.tar  — 解压镜像



新建并启动容器：

参数：-i  以交互模式运行容器；-t  为容器重新分配一个伪输入终端；--name  为容器指定一个名称

```
docker run -i -t --name mycentos
```





**docker启动命令**

docker start tomcat，语法为docker start 容器名称

**docker停止命令**

docker stop tomcat，语法为docker stop 容器名称

**docker重启命令**

docker restart tomcat ，语法为docker restart 容器名称



```
docker run --name nginx-test -p 8080:80 -d nginx

docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql

docker run -itd --name redis-test -p 6379:6379 redis
```



```
docker ps -a # 查看已经创建的容器
docker start 容器id # 开启容器
docker rm 容器id # 删除容器
```

