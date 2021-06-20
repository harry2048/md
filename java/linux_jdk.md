# jdk安装

## 1.安装前检查，是否已安装，若安装了，则先卸载

echo $JAVA_home

## 2.创建jdk安装路径

mkdir /usr/local/java

## 3.上传压缩包，进入文件目录解压

tar包

```sh
cd /usr/local/java
tar -xvf jdk-8u221-linux-x64.tar.gz
```

rpm

```sh
chmod +x jdk-8u211-linux-x64.rpm
rpm -ivh --prefix=/usr/local/java/ jdk-8u211-linux-x64.rpm
```

## 4.解压后对jdk文件进行授权

chmod +x /usr/local/java/jdk1.8.0_221/bin/java

chmod +x /usr/local/java/jdk1.8.0_221/bin/javac

chmod +x /usr/local/java/jdk1.8.0_221/jre/bin/java

## 5.配置java环境变量

**在文件末尾添加**

```sh
export JAVA_HOME=/usr/java/jdk1.8.0_221-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

设置配置生效

source /etc/profile

## 6. 检查是否安装成功

java -version

# linux 安装

安装centos7:  https://www.osyunwei.com/archives/7829.html

错误：ifconfig command is not found

执行命令：yum install -y net-tools

## 静态ip：

> su
>
> vi /etc/sysconfig/network-scripts/ifcfg-ens33
>
> 修改
>
> BOOTPROTO=static
>
> ONBOOT="yes"
>
> 增加
>
> IPADDR=192.168.85.152
>
> NETMASK=255.255.255.0
>
> GATEWAY=192.168.85.2
>
> DNS1=8.8.8.8  #设置主DNS
>
> DNS2=8.8.4.4  #设置备DNS
>
> 重启
>
> systemctl restart network
>

## hosts文件中指定ip

hostname  www  #设置主机名为www

vi /etc/hostname #编辑配置文件

www   #修改localhost.localdomain为www

:wq!  #保存退出

vi /etc/hosts #编辑配置文件

192.168.85.144   localhost  www   #修改localhost.localdomain为www  要指定ip

:wq!  #保存退出

shutdown -r now  #重启系统

# hadoop安装

### 解压

tar -xzf hadoop-2.6.5.tar.gz -C /opt

### 添加环境变量

```sh
export HADOOP_HOME=/opt/hadoop-2.6.5
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```



# 免密钥登录

ssh-keygen -t dsa -P '' -f ./ssh/id_dsa

cat ./ssh/id_dsa.pub > ./ssh/authorized_keys