### 静态IP

```sh
cd /etc/sysconfig/network-scripts
# 修改ifcfg-ens33
ONBOOT="yes"
BOOTPROTO="static"
IPADDR="192.168.85.152"
NETMASK="255.255.255.0"
GATEWAY="192.168.85.2"

service network restart
```

# ELK + filebeat

## Filebeat

> Filebeat由两个主要组件组成： prospectors 和 harvesters。 这些组件一起工作来尾随文件并将事件数据发送到您指定的输出。
>
> https://www.cnblogs.com/blogjun/articles/8108440.html
>
> 启动命令： nohup  ./filebeat -e -c filebeat.yml &
>
> 启动后  按 ctrl+C    输入exit命令退出

### harvesters

harvesters负责读取单个文件的内容。 harvesters逐行读取每个文件，并将内容发送到输出。 每个文件启动一台harvesters。 harvesters负责打开和关闭文件，这意味着在harvesters运行时文件描述符保持打开状态。 如果在收获文件时删除或重命名文件，Filebeat将继续读取文件。 这有副作用，在harvesters关闭之前，磁盘上的空间被保留。 默认情况下，Filebeat保持文件打开，直到达到close_inactive的设置（close_inactive默认为5分钟，即5分钟之内，没有最新的日志信息产生则关闭文件句柄）。
关闭harvester有以下情况：

- 如果在harvester还在读取文件时文件被删除，那么文件处理程序关闭，释放基础资源。
- 只有在scan_frequency过后，文件的采集才会重新开始。（scan_frequency参数默认为10秒，每隔10秒prospector检查目录中日志文件的变化情况）
- 如果在harvester关闭的情况下移动或移除文件，则不会继续收集文件。

### prospector

prospector负责管理harvesters并找到所有的读取源。如果输入类型是日志，则prospector会查找驱动器上与所定义的全局路径匹配的所有文件，并为每个文件启动一个harvesters。 每个prospector都在自己的Go例程中运行。

### Filebeat保持文件状态

Filebeat保持每个文件的状态，并经常刷新注册表文件中的磁盘状态。状态用于记住收割机正在读取的最后偏移量，并确保发送所有日志行。data目录中的registry文件

如果输出（如Elasticsearch或Logstash）无法访问，Filebeat将跟踪发送的最后一行，并在输出再次可用时继续读取文件。当Filebeat正在运行时，每个prospectors的状态信息也被保存在内存中。当Filebeat重新启动时，来自注册表文件的数据被用来重建状态，并且Filebeat继续在最后一个已知位置的每个harvester。
每个prospectors为每个找到的文件保留一个状态。由于文件可以被重命名或移动，文件名和路径不足以识别文件。对于每个文件，Filebeat存储唯一标识符以检测文件是否先前被收集。
如果您的使用案例涉及每天创建大量新文件，您可能会发现注册表文件会变得太大。（使用clean_inactive、clean_removed参数来调整）

### Filebeat确保至少一次交付

Filebeat保证事件至少被传递到配置的输出一次，没有数据丢失。 Filebeat能够实现此行为，因为它将每个事件的传递状态存储在注册表文件中。
在定义的输出被阻止并且没有确认所有事件的情况下，Filebeat会一直尝试发送事件，直到输出确认已经收到事件。
如果Filebeat在发送事件的过程中关闭，则不会等待输出在关闭之前确认所有事件。 任何发送到输出的事件，在Filebeat关闭之前没有被确认，在重新启动Filebeat时会再次发送。 这可确保每个事件至少发送一次，但最终可能会将重复事件发送到输出。 您可以通过设置shutdown_timeout选项来配置Filebeat以在关闭之前等待特定时间。（shutdown_timeout选项默认是关闭状态，可以设置时间，关闭时等待多长时间后再关闭）。
但是如果日志写入磁盘的速度超过了Filebeat读取日志的速度，当日志删除或者日志被覆盖时，则可能会丢失数据。
例如：
在Linux文件系统上，Filebeat使用inode和设备来识别文件。从磁盘中删除文件时，可将inode分配给新文件。在涉及文件旋转的使用情况下，如果旧文件被删除并且之后立即创建新文件，则新文件可能与删除的文件具有完全相同的inode。在这种情况下，Filebeat假定新文件与旧文件相同，并尝试在旧位置继续读取，这是不正确的。
默认状态不会从注册表文件中删除。要解决inode重用问题，我们建议您使用clean_ *选项（特别是clean_inactive）来删除非活动文件的状态。例如，如果您的文件每24小时轮换一次，并且轮换的文件不再更新，则可以将ignore_older设置为48小时，将clean_inactive设置为72小时。
您可以使用clean_removed从磁盘中删除的文件。请注意，clean_removed会在扫描期间无法找到文件时清除注册表中的文件状态。如果该文件稍后再次显示，则将从头开始重新发送。

### filebeat.yml

```yml
filebeat.inputs:
- type: log
  enabled: true
  # 收集路径
  paths:
    - /usr/local/etc/nginx/logs/access.log
    - /usr/local/etc/filebeat/test.log
  fields:
    tag: "afsp-ms"
  exclude_files: ['.gz$']
  # 以日期开头的数据为新的一行
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after
# 输出到logstash
output.logstash:
  hosts: ["192.168.85.143:5044", "192.168.85.148:5044"]
  loadbalance: true
logging.level: warning
# 默认设置 不用修改
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.template.settings:
  index.number_of_shards: 1
setup.kibana:
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
```

## ElasticSearch

> Elasticsearch 是一个分布式可扩展的实时搜索和分析引擎,一个建立在全文搜索引擎 Apache Lucene(TM) 基础上的搜索引擎.当然 Elasticsearch 并不仅仅是 Lucene 那么简单，它不仅包括了全文搜索功能，还可以进行以下工作:
>
> - 分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。
> - 实时分析的分布式搜索引擎。
> - 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。
> - https://www.cnblogs.com/dreamroute/p/8484457.html
> - https://blog.csdn.net/tianyeshiye/article/details/80863343
>
> 启动命令： nohup  ./elasticsearch &
>
> 启动后  按 ctrl+C    输入exit命令退出

### 倒排索引

也常被称为反向索引、置入档案或反向档案，是一种索引方法，被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射。它是文档检索系统中最常用的数据结构。通过倒排索引，可以根据单词快速获取包含这个单词的文档列表。

### 节点类型

​    Elasticsearch的一个实例是一个节点，一组节点形成一个集群。Elasticsearch集群中的节点可以通过三种不同的方式进行配置：

#### Master节点

- Master节点控制Elasticsearch集群，并负责在集群范围内创建/删除索引，跟踪哪些节点是集群的一部分，并将分片分配给这些节点。主节点一次处理一个集群状态，并将状态广播到所有其他节点，这些节点需要响应并确认主节点的信息。
- 在elasticsearch.yml中，将nodes.master属性设置为true（默认），可以将节点配置为有资格成为主节点的节点。
- 对于大型生产集群，建议拥有一个专用主节点来控制集群，并且不服务任何用户请求。

#### Data节点

- 数据节点用来保存数据和倒排索引。默认情况下，每个节点都配置为一个data节点，并且在elasticsearch.yml中将属性node.data设置为true。如果您想要一个专用的master节点，那么将node.data属性更改为false。

#### Client节点

​    如果将node.master和node.data设置为false，则将节点配置为客户端节点，并充当**负载平衡**器，将传入的请求路由到集群中的不同节点。 
​    若你连接的是作为客户端的节点，该节点称为协调节点(coordinating node)。协调节点将客户机请求路由到集群中对应分片所在的节点。对于读取请求，协调节点每次选择不同的分片来提供请求以平衡负载。 
​    在我们开始审查发送到协调节点的CRUD请求如何通过集群传播并由引擎执行之前，让我们看看Elasticsearch如何在内部存储数据，以低延迟为全文搜索提供结果。

#### 存储模型

​    Elasticsearch使用Apache Lucene，它是由Java编写的全文搜索库，由Doug Cutting（Apache Hadoop的创建者）内部开发，它使用称为倒排索引的数据结构，用于提供低延迟搜索结果。 
​    文档（document）是Elasticsearch中的数据单位，并通过对文档中的术语进行标记来创建倒排索引，创建所有唯一术语的排序列表，并将文档列表与可以找到该词的位置相关联。

### 动态mapping

> https://blog.csdn.net/qq_21383435/article/details/109387538
> dynamic用来配置处理新出现字段的行为，有true，false，strict 三种
>
> true 是默认值，会自动在 mapping 中添加字段，并在文档中保存新字段的值。
> false 代表不会在 mapping 中添加字段，但会将数据存起来。
> strict 代表既不会新增字段，也不会保存新字段的内容，遇到新字段直接报错。 

#### demo

```sh
# 设置动态mapping
PUT student
{
  "mappings": {
    "dynamic": "true",
    "properties": {
      "name": {
        "type": "keyword",
        "doc_values": false,
       #"analyzer":"ik_max_word" 指定ik分词
      }
    }
  },
  "settings": {
    "number_of_shards": "1",
    "number_of_replicas": "1"
  }
}

# 插入数据
POST _bulk
{ "index" : { "_index" : "student", "_id" : "1" } }
{ "name" : "张三", "age": 12 }

# 查询结果
GET student/_search
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 8,
    "successful" : 8,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "张三",
          "age" : 12
        }
      }
    ]
  }
}

# 查询mapping
GET student/_mapping
{
  "student" : {
    "mappings" : {
      "dynamic" : "true",
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "name" : {
          "type" : "keyword",
          "doc_values" : false
        }
      }
    }
  }
}
```

# 

### elasticsearch.yml

```yml
# 集群名称
cluster.name: poin-es-app
# 当前节点名称
node.name: data-1
# 数据位置
path.data: /usr/local/etc/es/data
# 日志位置
path.logs: /usr/local/etc/es/logs
# 内存超过系统内存时报错
bootstrap.memory_lock: true
# 当前节点ip
network.host: 192.168.85.144
# 当前节点port
http.port: 9200
# 通讯节点port
transport.port: 9300
# 主节点群ip
discovery.seed_hosts: ["192.168.85.150:9300"]
# 初始主节点群，可以是多个
cluster.initial_master_nodes: ["master-1"]
# 非主节点
node.master: false
# 数据节点
node.data: true
# 跨域
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### jvm.options

```options
# 根据情况修改
-Xms1g
-Xmx1g
```

### ES线程池配置

> https://blog.csdn.net/weixin_34150830/article/details/90102607
>
> 内部维护多个线程池
>
> index：此线程池用于索引和删除操作。它的类型默认为fixed，size默认为可用处理器的数量，队列的size默认为300。
> search：此线程池用于搜索和计数请求。它的类型默认为fixed，size默认为可用处理器的数量乘以3，队列的size默认为1000。
> suggest：此线程池用于建议器请求。它的类型默认为fixed，size默认为可用处理器的数量，队列的size默认为1000。
> get：此线程池用于实时的GET请求。它的类型默认为fixed，size默认为可用处理器的数量，队列的size默认为1000。
> bulk：此线程池用于批量操作。它的类型默认为fixed，size默认为可用处理器的数量，队列的size默认为50。
> percolate：此线程池用于预匹配器操作。它的类型默认为fixed，size默认为可用处理器的数量，队列的size默认为1000。

```yml
threadpool.index.type: fixed
threadpool.index.size: 100
threadpool.index.queue_size: 500
```



## Logstash

> Logstash事件处理有三个阶段：inputs → filters → outputs。是一个接收，处理，转发日志的工具。支持系统日志，错误日志，应用日志，总之包括所有可以抛出来的日志类型。
>
> https://www.cnblogs.com/momenglin/p/10744767.html
>
> 启动命令：nohup ./bin/logstash -f config/logstash-test.conf &
>
> 启动后  按 ctrl+C    输入exit命令退出

### input 接收数据

一些常用的输入为：

file：从文件系统的文件中读取，类似于tial -f命令

syslog：在514端口上监听系统日志消息，并根据RFC3164标准进行解析

redis：从redis service中读取

beats：从filebeat中读取

### filter 过滤数据

一些常用的过滤器为：

grok：解析任意文本数据，Grok 是 Logstash 最重要的插件。它的主要作用就是将文本格式的字符串，转换成为具体的结构化的数据，配合正则表达式使用。内置120多个解析语法。

官方提供的grok表达式：https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns
grok在线调试：https://grokdebug.herokuapp.com/

mutate：对字段进行转换。例如对字段进行删除、替换、修改、重命名等。

drop：丢弃一部分events不进行处理。

clone：拷贝 event，这个过程中也可以添加或移除字段。

geoip：添加地理信息(为前台kibana图形化展示使用)

### output 输出数据

outputs是logstash处理管道的最末端组件。一个event可以在处理过程中经过多重输出，但是一旦所有的outputs都执行结束，这个event也就完成生命周期。

一些常见的outputs为：

elasticsearch：可以高效的保存数据，并且能够方便和简单的进行查询。

file：将event数据保存到文件中。

graphite：将event数据发送到图形化组件中，一个很流行的开源存储图形化展示的组件。

### logstash.conf

```conf
# 监听beats发来的5044端口号
input {
  beats {
    port => 5044
  }
}
# 过滤数据
filter {
  grok{
    match => {
       "message" => "%{TIMESTAMP_ISO8601:logdate} %{USERNAME:serviceName} %{DATA:area} \[%{WORD:thread}\] %{LOGLEVEL:loglevel} %{WORD:fileName} (?<shortMsg>.*?) (?<data>.*)"
    }
  }
  # 获取日志中的日期字段设置为时间戳
  date{
    match => ["logdate", "yyyy-MM-dd HH:mm:ss.SSS"]
    target => "@timestamp"
    timezone => "Asia/Shanghai"
  }
  # 删掉没用的字段
  mutate{
    remove_field => ["logdate", "host", "agent", "ecs", "tags", "input"]
  }
}
# 输出到es master主节点群
output {
   elasticsearch {
     hosts => ["http://192.168.85.150:9200"]
     # 索引以filebeat中设置的tag变量开头，也可以不以这个开头
     index => "%{[fields][tag]}-%{+YYYY.MM.dd}"
   }
   # 测试时打印到console
   #stdout { codec => rubydebug}
}
```

### logstash.yml

```yml
# 当前节点名称
node.name: logstash
# 当前管道id
pipeline.id: logstash
# 当前节点ip
http.host: "192.168.85.143"
# 开启监控
xpack.monitoring.enabled: true
# es的master节点群
xpack.monitoring.elasticsearch.hosts: ["http://192.168.85.150:9200"]
```

## Kibana

> Kibana 在整个 Elastic Stack 家族中起到数据可视化的作用，也就是通过图、 表、统计等方式将复杂的数据以更直观的形式展示出来。由于 Kibana 运行于 Elasticsearch 基础之上，所以可以将 Kibana 视为 Elasticsearch 的用户图形界面 ( Graphic User Interface, GUI) 。
>
> https://blog.csdn.net/nandao158/article/details/109024478
>
> 启动命令：nohup ./kibana &
>
> 启动后  按 ctrl+C    输入exit命令退出

### kibana.yml

```yml
# 当前port
server.port: 5601
# 当前ip
server.host: "192.168.85.151"
# es主节点群ip:port
elasticsearch.hosts: ["http://192.168.85.150:9200"]
```

