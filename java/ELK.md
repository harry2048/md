# Elasticsearch

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

