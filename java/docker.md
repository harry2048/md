#  1 安装docker

### 1.下载关于Docker的依赖环境

yum -y install yum-utils device-mapper-persistent-data lvm2

### 2. 设置一下下载Docker的镜像源

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

```
curl: (6) Could not resolve host: mirrors.163.com; Unknown error 服务器上解析不了域名，换成ip可以
原因是DNS域名解析问题：
添加nameserver即可解决
echo nameserver 8.8.8.8 > /etc/resolv.conf

或
echo nameserver 8.8.4.4 > /etc/resolv.conf

阿里云镜像不好使，换163  http://mirrors.163.com/.help/centos.html
```

> ```
> wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
> yum clean all
> yum makecache
> ```

### 3. install Docker:

yum makecache fast

yum -y install docker-ce

### 4. start :

systemctl start docker

设置开机启动:

systemctl enable docker

test:

docker run hello-world



#  2 Docker的中央仓库

1. 中央:

   https://hub.docker.com

2. 国内镜像

   https://c.163yun.com/hub#/home

   http://hub.daocloud.io/     推荐

3. 私服，需要以下配置：

   修改  /etc/docker/daemon.json

   {

   ​	"registry-mirrors":["https://registry.docker-cn.com"],

   ​	"insecure-registries":["私服ip:私服port"]

   }

4. 重启俩个服务

   systemctl daemon-reload

   systemctl restart docker

   

##  镜像的操作

1. 拉取镜像到本地

   docker pull 镜像名称[:tag]

   > 栗子：docker pull daocloud.io/library/tomcat:8.5.15-jre8

2. 查看所有镜像

   docker images

3. 删除本地镜像

   docker rmi 镜像的标识

   强制删除 docker rmi mirrorId -f

4. 镜像的导入导出，手动

   * 将本地的镜像导出

   docker save -o 导出的路径  mirrorId

   * 加载本地的镜像文件

   docker load -i  镜像文件

   * 修改镜像名称

   docker tag mirrorId tomcat:8.5

## 容器的操作

### 1. 运行容器

docker run 镜像标识|镜像名称

* 常用命令

  docker run -d -p 宿主机端口:容器端口  --name  容器名称  镜像标识|镜像名称

  -d:  后台运行

  -p:  宿主机端口:容器端口： 为了映射当前Linux的端口号和容器的端口号

  --name 容器的名称

   镜像标识|镜像名称 ： 镜像名称或者标识，都可以

### 2. 查看正在运行的容器

​       docker ps -a

​		-a: 所有的

​	    -q: 只查看正在运行的容器标识

### 3. 删除本地容器

docker stop 容器id

停止所有容器：   docker stop $(docker ps -qa)

docker rm 容器id

删除所有容器：   docker rm $(docker ps -qa)

### 4. 进入到容器内部

docker exec -it  容器id  bash

### 5. 查看容器的日志

docker logs -f  容器id

### 6. 测试方式启动容器

启动docker 时，如果只是为了调试。那么可以加上 --rm  如：

docker run --name nginx --rm nginx:stable-alpine

### 7. 启动已经停止的容器

docker start 容器id



## Docker应用

### 1. 运行mysql 容器

```sh
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root daocloud.io/library/mysql:5.7.4
```

### 2. 将war包放入tomcat中，放入webapps中

```sh
docker cp 文件名称 容器id:容器内部路径
栗子：
docker cp ssm.war fe:/usr/local/tomcat/webapps/
```

### 3. 数据卷

> 为了部署ssm项目，需要使用cp命令将ssm.war复制到容器内部。
>
> 数据卷：将宿主机的一个目录映射到容器的一个目录中
>
> 可以在宿主机中操作目录中的内容，容器内部映射的文件，也会一起改变

```sh
1. 创建数据卷
docker volume create 数据卷名称
创建数据卷之后，默认会存放在一个目录下 /var/lib/docker/volumes/数据卷名称/_data
```

```
2. 查看数据卷的详细信息
docker volume inspect 数据卷名称
```

```
3. 查看全部数据卷
docker volume ls
```

```
4. 删除数据卷
docker volume rm 数据卷名称
```

```
5. 应用数据卷
# 当你映射数据卷时，如果数据卷不存在，Docker会帮你自动创建
docker run -v 数据卷名称:容器内部的路径  镜像id

# 直接指定一个路径作为数据卷的存放位置  推荐使用
docker run -v 路径:容器内部的路径  镜像id
```



#  3 Docker_Compose

> 之前运行一个镜像，需要添加大量的参数
>
> 可以通过Docker-Compose编写这些参数
>
> Docker-Compose可以帮助我们批量的管理容器
>
> 只需要通过一个Docker-Compose.yml 文件去维护即可

### 1. 下载Docker-Compose

1.去github官网搜索docker-compose

```
https://github.com/docker/compose/releases/download/1.21.1/docker-compose-Linux-x86_64
```

2.将下载好的文件，拖拽到linux系统中

3.需要将DockerCompose文件的名称修改简单一点，基于DockerCompose文件一个可执行的权限

```
chmod 777 docker-compose
```

4.将docker-compose放到/usr/local/bin，并设置环境变量

```
vi /etc/profile
export PATH=$PATH:$JAVA_HOME:/usr/local/bin
source /etc/profile
```

5.测试，在任意目录，输入 docker-compose version

###  2.Docker-Compose管理MySQL和Tomcat容器

> yml文件以 key:value 方式来指定配置信息
>
> 多个配置信息以换行+缩进的方式来区分

```yml
version: '3.1'
services:
  mysql:			 # 自定义服务的名称
    restart: always  # 代表只要docker启动，那么这个容器也跟着启动
    image: daocloud.io/library/mysql:5.7.4  #指定镜像路径
    container_name: mysql #指定容器的名称
    ports:
      - 3306:3306    #指定端口号的映射
    environment:
      MYSQL_ROOT_PASSWORD: root   # 指定MySQL的ROOT用户登录密码
      TZ: Asia/Shanghai           # 指定时区
    volumes:
      - /opt/docer_mysql_tomcat/data:/var/lib/mysql  # 映射数据卷
  tomcat:
    restart: always
    image: daocloud.io/library/tomcat:8.5.15-jre8
    container_name: tomcat
    ports:
      - 8080:8080
    environment:
      TZ: Asia/Shanghai
    volumes:
      - /opt
```

