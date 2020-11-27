## 1. 安装

集群配置：三台哨兵，一主俩从

> ​     	服务器   							部署应用
>
> 192.168.85.151						哨兵：主节点
>
> 192.168.85.152						哨兵：从1
>
> 192.168.85.153						哨兵：从2

### 解压

```sh
tar -zxvf redis-6.0.6.tar.gz
cd redis-6.0.6

## 6.0.0以上版本需要升级gcc
yum install -y gcc
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

## 临时将此时的gcc版本改为9
scl enable devtoolset-9 bash

make
```

### 安装

修改从节点 152  153 的redis.conf 的配置

```sh
daemonize yes

replicaof 192.168.56.106 6379
masterauth 123456  #密码
bind 192.168.85.152  #云服务器搭建，使用内网ip，连接时使用公网ip

# 确保有1个从节点写入，且延时不超过10s，否则主节点会停止写入请求（防止数据丢失）
min-replicas-to-write 1
min-replicas-max-lag 10  #有1个redis的连接大于10秒，主连接不再接收写请求

requirepass 123456
appendonly yes

# 常用配置
database 16   			  # 16个实例 ，每个空实例占用1M大小，集群模式下只有一个db0
maxmemory   5gb		 	  # 最大使用的内存
logfile "logs/redis.log"  # redis启动日志路径
maxclients 10000  		  # 客户端最大连接数，默认10000		
aof-use-rdb-preamble yes  # 默认yes，恢复时同时使用aof和rdb
```

修改主节点  151  的  redis.conf 的配置

```sh
daemonize yes

masterauth 123456
bind 192.168.85.151

# 确保有1个从节点写入，且延时不超过10s，否则主节点会停止写入请求（防止数据丢失）
min-replicas-to-write 1
min-replicas-max-lag 10

requirepass 123456

appendonly yes
```

### 设置日志路径

```sh
# 先创建该目录
logfile "/home/soft/redis-5.0.7/logs/redis.log"
```

### 启动

启动并查看主节点状态

```sh
# 启动
src/redis-server redis.conf

# 客户端连接  查询状态
src/redis-cli -p 6379
info replication
```

### sentinel配置

修改 151  152  153 的sentinel.conf 的配置

```sh
daemonize yes

sentinel monitor mymaster 192.168.56.106 6379 2
# 2 : 当前有2个sentinel认为主节点挂掉就算挂掉

sentinel auth-pass mymaster 123456
sentinel down-after-milliseconds mymaster 30000  #默认30秒无响应，主观下线
sentinel failover-timeout mymaster 180000 		 #默认3分钟，选举超3分钟，就是异常
sentinel parallel-syncs mymaster 1 				 #默认为1，主备切换时，最多有多少个从copy主
```

### 启动哨兵

```sh
src/redis-sentinel sentinel.conf

src/redis-cli -p 26379
# 主节点状态
sentinel master mymaster
# 副本状态
SENTINEL replicas mymaster
# 哨兵状态
SENTINEL sentinels mymaster
```

## 2. 过期淘汰策略

* 过期策略

  惰性删除

  随机删除：默认每秒10次过期扫描，每次随机查询20个，然后删除

* 淘汰策略

  可以设置maxmemory（**默认不限制**）来限制最大使用内存，如果超出了还可以选择几种策略来应对：

  noeviction：不会继续服务写请求，读请求可以继续进行，这是默认的淘汰策略

  volatile-lru：淘汰已经过期的，最少使用的key   (推荐使用)

  volatile-ttl：淘汰已经过期的，剩余寿命最短的

  volatile-random：随机淘汰一些已经过期的

  allkeys-lru：淘汰那些最少使用的key

  allkeys-random：随机淘汰一些key

  volatile-lfu 对已经过期的key进行LFU (访问频率) 淘汰

  allkeys-lfu 对所有key进行LFU淘汰































