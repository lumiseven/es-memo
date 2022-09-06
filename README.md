# ES 备忘录 

# HTTP 操作

## 1.索引操作
对比关系型数据库,创建索引就等同于创建数据库

1. 创建索引

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/shopping\?pretty
{
  "acknowledged" : true, # 响应结果 true 操作成功
  "shards_acknowledged" : true,  # 分片结果 分片操作成功
  "index" : "shopping" # 索引名称
}
# 注意:创建索引库的分片数默认 1 片,在 7.0.0 之前的 Elasticsearch 版本中,默认 5 片
```

如果重复添加索引,会返回错误信息

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/shopping\?pretty
{
  "error" : {
    "root_cause" : [
      {
        "type" : "resource_already_exists_exception",
        "reason" : "index [shopping/0z5i49HJQjSn4VvBry5M4Q] already exists",
        "index_uuid" : "0z5i49HJQjSn4VvBry5M4Q",
        "index" : "shopping"
      }
    ],
    "type" : "resource_already_exists_exception",
    "reason" : "index [shopping/0z5i49HJQjSn4VvBry5M4Q] already exists",
    "index_uuid" : "0z5i49HJQjSn4VvBry5M4Q",
    "index" : "shopping"
  },
  "status" : 400
}
```

2. 查看所有索引
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/_cat/indices\?v 
health status index           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   my-index-000001 BlGRV5bsRiiNMD-c2bCWnw   1   1         34            0     44.7kb         44.7kb
yellow open   shopping        0z5i49HJQjSn4VvBry5M4Q   1   1          0            0       225b           225b
```

输出含义

| 表头 | 含义 |
| health | 当前服务器健康状态:
green(集群完整) yellow(单点正常、集群不完整) red(单点不正常) |
| status | 索引打开、关闭状态 |
| index | 索引名 |
| uuid | 索引统一编号 |
| pri | 主分片数量 |
| rep | 副本数量 |
| docs.count | 可用文档数量 |
| docs.deleted | 文档删除状态(逻辑删除) |
| store.size | 主分片和副分片整体占空间大小 |
| pri.store.size | 主分片占空间大小 |

3. 查看单个索引
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/shopping\?pretty
{
  "shopping" : { # 索引名
    "aliases" : { }, # 别名
    "mappings" : { }, # 映射
    "settings" : { # 设置
      "index" : { # 设置 - 索引
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1", # 设置 - 索引 - 主分片数量
        "provided_name" : "shopping", # 设置 - 索引 - 名称
        "creation_date" : "1662360347956", # 设置 - 索引 - 创建时间
        "number_of_replicas" : "1", # 设置 - 索引 - 副分片数量
        "uuid" : "0z5i49HJQjSn4VvBry5M4Q", # 设置 - 索引 - 唯一标识
        "version" : { # 设置 - 索引 - 版本
          "created" : "8010099"
        }
      }
    }
  }
}
```

4. 删除索引
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X DELETE https://localhost:9200/shopping\?pretty

{
  "acknowledged" : true
}
```

## 文档操作

1. 创建文档
索引已经创建好了,接下来我们来创建文档,并添加数据。这里的文档可以类比为关系型数
据库中的表数据,添加的数据格式为 JSON 格式

```json
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/shopping/_doc\?pretty -H 'Content-Type: application/json' -d'
{
    "title":"小米手机",
    "category":"小米",                           
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
' 
{
  "_index" : "shopping", # 索引
  "_id" : "oTV7DIMB798oQui36rWl", # 唯一标识 
  "_version" : 1, # 版本
  "result" : "created", # 结果 这里的 create 表示创建成功
  "_shards" : { # 分片
    "total" : 2, # 分片 - 总数
    "successful" : 1, # 分片 - 成功
    "failed" : 0 # 分片 - 失败
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

上面的数据创建后,由于没有指定数据唯一性标识(ID),默认情况下,ES 服务器会随机
生成一个。

如果想要自定义唯一性标识,需要在创建时指定
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/shopping/_doc/1\?pretty -H 'Content-Type: application/json' -d'{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}'
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

注意 **如果增加数据时明确数据主键,那么请求方式也可以为 PUT**

2. 查看文档
查看文档时,需要指明文档的唯一性标识,类似于 MySQL 中数据的主键查询

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/shopping/_doc/1\?pretty                                          
{
  "_index" : "shopping", # 索引
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true, # 查询结果 true 表示查找到,false 表示未查找到
  "_source" : { # 文档源信息
    "title" : "小米手机",
    "category" : "小米",
    "images" : "http://www.gulixueyuan.com/xm.jpg",
    "price" : 3999.0
  }
}
```

3. 修改文档
和新增文档一样,输入相同的 URL 地址请求,如果请求体变化,会将原有的数据内容覆盖

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/shopping/_doc/1\?pretty -H 'Content-Type: application/json' -d'{
    "title":"华为手机",
    "category":"华为",      
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":4999.00
}'
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 2, # 版本
  "result" : "updated", # 结果 updated 表示数据被更新
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

4. 修改字段
修改数据时,也可以只修改某一给条数据的局部信息

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/shopping/_doc/1\?pretty -H 'Content-Type: application/json' -d'{
    "doc": {           
        "price":3000.00
    }
}'
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}

-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/shopping/_doc/1\?pretty                                          
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 3,
  "_seq_no" : 3,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "doc" : {
      "price" : 3000.0
    }
  }
}
```

5. 删除文档
删除一个文档不会立即从磁盘上移除,它只是被标记成已删除(逻辑删除)。
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X DELETE https://localhost:9200/shopping/_doc/1\?pretty
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 4, # 版本 对数据的操作,都会更新版本
  "result" : "deleted", # deleted 表示数据被标记为删除
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

如果删除后再查询当前文档信息
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/shopping/_doc/1\?pretty   
{
  "_index" : "shopping",
  "_id" : "1",
  "found" : false
}
```

如果删除一个并不存在的文档
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X DELETE https://localhost:9200/shopping/_doc/1\?pretty
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 1,
  "result" : "not_found", # 结果 not_found 表示未查找到
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 1
}
```

6. 条件删除文档
一般删除数据都是根据文档的唯一性标识进行删除,实际操作时,也可以根据条件对多条数
据进行删除
首先分别增加多条数据:
```json
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":4000.00
}

{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":4000.00
}
```

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/shopping/_doc/1\?pretty -H 'Content-Type: application/json' -d'{
    "title":"小米手机",
    "category":"小米",                           
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":4000.00
}'
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 6,
  "_primary_term" : 1
}

-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/shopping/_doc/1\?pretty -H 'Content-Type: application/json' -d'{
    "title":"华为手机",
    "category":"华为",                           
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":4000.00
}'
{
  "_index" : "shopping",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 7,
  "_primary_term" : 1
}


```

**_delete_by_query**

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/shopping/_delete_by_query\?pretty -H 'Content-Type: application/json' -d'{
    "query":{
        "match":{
            "price":4000.00
        }
    }
}'
{
  "took" : 995, # 耗时
  "timed_out" : false, # 是否超时
  "total" : 2, # 总数
  "deleted" : 2, # 删除数量
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

## 映射操作
有了索引库,等于有了数据库中的 database。
接下来就需要建索引库(index)中的映射了,类似于数据库(database)中的表结构(table)。
创建数据库表需要设置字段名称,类型,长度,约束等;索引库也一样,需要知道这个类型
下有哪些字段,每个字段有哪些约束信息,这就叫做映射(mapping)。

1. 创建映射
```json
{
    "properties": {
        "name":{
            "type": "text",
            "index": true
        },
        "sex":{
            "type": "text",
            "index": false
        },
        "age":{
            "type": "long",
            "index": false
        }
    }
}
```

```zsh
# 先创建一个新的 index 名称 student
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/student\?pretty 
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "student"
}

# 创建 mapping
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/student/_mapping\?pretty -H 'Content-Type: application/json' -d'{
    "properties": {
        "name":{
            "type": "text",
            "index": true
        },
        "sex":{
            "type": "text",
            "index": false
        },
        "age":{
            "type": "long",
            "index": false
        }
    }
}'
{
  "acknowledged" : true
}
```

映射数据说明:
- 字段名:任意填写,下面指定许多属性,例如:title、subtitle、images、price
- type:类型,Elasticsearch 中支持的数据类型非常丰富,说几个关键的:
  - String 类型,又分两种:
    - text:可分词
    - keyword:不可分词,数据会作为完整字段进行匹配
  - Numerical:数值类型,分两类
    - 基本数据类型:long、integer、short、byte、double、float、half_float
    - 浮点数的高精度类型:scaled_float
  - Date:日期类型
  - Array:数组类型
  - Object:对象
- index:是否索引,默认为true,也就是说你不进行任何配置,所有字段都会被索引。
  - true:字段会被索引,则可以用来进行搜索
  - false:字段不会被索引,不能用来搜索
- store:是否将数据进行独立存储,默认为 false
  - 原始的文本会存储在_source 里面,默认情况下其他提取出来的字段都不是独立存储的,是从_source 里面提取出来的。当然你也可以独立的存储某个字段,只要设置"store": true 即可,获取独立存储的字段要比从_source 中解析快得多,但是也会占用更多的空间,所以要根据实际业务需求来设置。
- analyzer:分词器,这里的 ik_max_word 即使用 ik 分词器

2. 查看映射
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_mapping\?pretty                                         
{
  "student" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long",
          "index" : false
        },
        "name" : {
          "type" : "text"
        },
        "sex" : {
          "type" : "text",
          "index" : false
        }
      }
    }
  }
}
```

4. 修改映射
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/student/_mapping\?pretty -H 'Content-Type: application/json' -d'
{
  "properties" : {
    "age" : {
      "type" : "long",
      "index" : false
    },
    "name" : {
      "type" : "text"
    },
    "nickname" : {
      "type" : "text",
      "fields" : {
        "keyword" : {
          "type" : "keyword",
          "ignore_above" : 256
        }
      }
    },
    "sex" : {
      "type" : "text",
      "index" : false
    },
    "uid" : {
      "type" : "text",
      "fields" : {
        "keyword" : {
          "type" : "keyword",
          "ignore_above" : 256
        }
      }
    }
  }
}
'
{
  "acknowledged" : true
}
```

## 高级查询
Elasticsearch 提供了基于 JSON 提供完整的查询 DSL 来定义查询
定义数据:
```json
# POST /student/_doc/1001
{
    "name":"zhangsan",
    "nickname":"zhangsan",
    "sex":"男",
    "age":30
}
# POST /student/_doc/1002
{
    "name":"lisi",
    "nickname":"lisi",
    "sex":"男",
    "age":20
}
# POST /student/_doc/1003
{
    "name":"wangwu",
    "nickname":"wangwu",
    "sex":"女",
    "age":40
}
# POST /student/_doc/1004
{
    "name":"zhangsan1",
    "nickname":"zhangsan1",
    "sex":"女",
    "age":50
}
# POST /student/_doc/1005
{
    "name":"zhangsan2",
    "nickname":"zhangsan2",
    "sex":"女",
    "age":30
}
```

1. 查询所有文档
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
    "query": {         
        "match_all": {}
    }
}'
{
  "took" : 92, # 查询花费时间,单位毫秒
  "timed_out" : false, # 是否超时
  "_shards" : { # 分片信息
    "total" : 1, # 总数
    "successful" : 1, # 成功
    "skipped" : 0, # 忽略
    "failed" : 0 # 失败
  },
  "hits" : { # 搜索命中结果
    "total" : { # 搜索条件匹配的文档总数
      "value" : 5, # 总命中计数的值
      "relation" : "eq" # 计数规则 eq 表示计数准确, gte 表示计数不准确
    },
    "max_score" : 1.0, # 匹配度分值
    "hits" : [ # 命中结果集合
      {
        "_index" : "student",
        "_id" : "1001",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      },
      {
        "_index" : "student",
        "_id" : "1002",
        "_score" : 1.0,
        "_source" : {
          "name" : "lisi",
          "nickname" : "lisi",
          "sex" : "男",
          "age" : 20
        }
      },
      {
        "_index" : "student",
        "_id" : "1003",
        "_score" : 1.0,
        "_source" : {
          "name" : "wangwu",
          "nickname" : "wangwu",
          "sex" : "女",
          "age" : 40
        }
      },
      {
        "_index" : "student",
        "_id" : "1004",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan1",
          "nickname" : "zhangsan1",
          "sex" : "女",
          "age" : 50
        }
      },
      {
        "_index" : "student",
        "_id" : "1005",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan2",
          "nickname" : "zhangsan2",
          "sex" : "女",
          "age" : 30
        }
      }
    ]
  }
}
# "query":这里的 query 代表一个查询对象,里面可以有不同的查询属性
# "match_all":查询类型,例如:match_all(代表查询所有), match,term , range 等等
# {查询条件}:查询条件会根据类型的不同,写法也有差异
```

2. 匹配查询
match 匹配类型查询,会把查询条件进行分词,然后进行查询,多个词条之间是 or 的关系
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
    "query": {         
        "match": {
            "name": "zhangsan"
        }
    }
}'
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.3862942,
    "hits" : [
      {
        "_index" : "student",
        "_id" : "1001",
        "_score" : 1.3862942,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      }
    ]
  }
}
```

3. 字段匹配查询
multi_match 与 match 类似,不同的是它可以在多个字段中查询。
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
    "query": {         
        "multi_match": {
            "query": "zhangsan",
            "fields": ["name","nickname"]
        }
    }
}'
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.3862942,
    "hits" : [
      {
        "_index" : "student",
        "_id" : "1001",
        "_score" : 1.3862942,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      }
    ]
  }
}
```

4. 关键字精确查询
term 查询,精确的关键词匹配查询,不对查询条件进行分词。
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
    "query": {         
        "term": {
            "name": {
                "value": "zhangsan"
            }
        }
    }
}'
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.3862942,
    "hits" : [
      {
        "_index" : "student",
        "_id" : "1001",
        "_score" : 1.3862942,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      }
    ]
  }
}
```

5. 多关键字精确查询
terms 查询和 term 查询一样,但它允许你指定多值进行匹配。
如果这个字段包含了指定值中的任何一个值,那么这个文档满足条件,类似于 mysql 的 in
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
    "query": {         
        "terms": {
            "name": ["zhangsan","lisi"]
        }
    }
}'
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "student",
        "_id" : "1001",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      },
      {
        "_index" : "student",
        "_id" : "1002",
        "_score" : 1.0,
        "_source" : {
          "name" : "lisi",
          "nickname" : "lisi",
          "sex" : "男",
          "age" : 20
        }
      }
    ]
  }
}
```

6. 指定查询返回字段
默认情况下,Elasticsearch 在搜索的结果中,会把文档中保存在_source 的所有字段都返回。
如果我们只想获取其中的部分字段,我们可以添加_source 的过滤
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'
{
    "_source": ["name","nickname"],
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
'
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
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
        "_id" : "1001",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan"
        }
      }
    ]
  }
}
```

7. 过滤字段
我们也可以通过:
- includes:来指定想要显示的字段
- excludes:来指定不想要显示的字段

**includes**
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'
{
    "_source": {
        "includes": ["name", "nickname"]
    },
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
'
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
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
        "_id" : "1001",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan"
        }
      }
    ]
  }
}
```

**excludes**
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'
{
    "_source": {
        "excludes": ["name", "nickname"]
    },
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
'
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
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
        "_id" : "1001",
        "_score" : 1.0,
        "_source" : {
          "sex" : "男",
          "age" : 30
        }
      }
    ]
  }
}
```

8. 组合查询
`bool`把各种其它查询通过`must`(必须)、`must_not`(必须不)、`should`(应该)的方式进行组合
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "zhangsan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "age": "40"
          }
        }
      ],
      "should": [
        {
          "match": {
            "sex": "男"
          }
        }
      ]
    }
  }
}
'
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 2.261763,
    "hits" : [
      {
        "_index" : "student_v2",
        "_id" : "1001",
        "_score" : 2.261763,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      }
    ]
  }
}
'
```

9. 范围查询
range 查询找出那些落在指定区间内的数字或者时间。range 查询允许以下字符

| 操作符 | 说明 |
| -- | -- |
| gt | 大于> |
| gte | 大于等于>= |
| lt | 小于< |
| lte | 小于等于<= |

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 35
      }
    }
  }
}
'
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "student_v2",
        "_id" : "1001",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      },
      {
        "_index" : "student_v2",
        "_id" : "1005",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan2",
          "nickname" : "zhangsan2",
          "sex" : "女",
          "age" : 30
        }
      }
    ]
  }
}
```

10. 模糊查询
返回包含与搜索字词相似的字词的文档。
编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数。这些更改可以包括:
- 更改字符(box → fox)
- 删除字符(black → lack)
- 插入字符(sic → sick)
- 转置两个相邻字符(act → cat)

为了找到相似的术语,fuzzy 查询会在指定的编辑距离内创建一组搜索词的所有可能的变体
或扩展。然后查询返回每个扩展的完全匹配。

通过 fuzziness 修改编辑距离。一般使用默认值 AUTO,根据术语的长度生成编辑距离。

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "zhan2sana"
      }
    }
  }
}
'
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.078229,
    "hits" : [
      {
        "_index" : "student_v2",
        "_id" : "1004",
        "_score" : 1.078229,
        "_source" : {
          "name" : "zhangsan1",
          "nickname" : "zhangsan1",
          "sex" : "女",
          "age" : 50
        }
      },
      {
        "_index" : "student_v2",
        "_id" : "1005",
        "_score" : 1.078229,
        "_source" : {
          "name" : "zhangsan2",
          "nickname" : "zhangsan2",
          "sex" : "女",
          "age" : 30
        }
      },
      {
        "_index" : "student_v2",
        "_id" : "1001",
        "_score" : 1.0397208,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      }
    ]
  }
}
```

fuzziness 指定编辑距离:
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "zhan2san",
        "fuzziness": 1
      }
    }
  }
}
'
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.2130076,
    "hits" : [
      {
        "_index" : "student_v2",
        "_id" : "1001",
        "_score" : 1.2130076,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        }
      }
    ]
  }
}
```

11. 单字段排序
sort 可以让我们按照不同的字段进行排序,并且通过 order 指定排序的方式。desc 降序,asc升序。

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "query": {         
    "match_all": {}
  },
  "sort": [{
    "age": {
      "order": "desc"
    }
  }]
}'
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "student_v2",
        "_id" : "1004",
        "_score" : null,
        "_source" : {
          "name" : "zhangsan1",
          "nickname" : "zhangsan1",
          "sex" : "女",
          "age" : 50
        },
        "sort" : [
          50
        ]
      },
      {
        "_index" : "student_v2",
        "_id" : "1003",
        "_score" : null,
        "_source" : {
          "name" : "wangwu",
          "nickname" : "wangwu",
          "sex" : "女",
          "age" : 40
        },
        "sort" : [
          40
        ]
      },
      {
        "_index" : "student_v2",
        "_id" : "1001",
        "_score" : null,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        },
        "sort" : [
          30
        ]
      },
      {
        "_index" : "student_v2",
        "_id" : "1005",
        "_score" : null,
        "_source" : {
          "name" : "zhangsan2",
          "nickname" : "zhangsan2",
          "sex" : "女",
          "age" : 30
        },
        "sort" : [
          30
        ]
      },
      {
        "_index" : "student_v2",
        "_id" : "1002",
        "_score" : null,
        "_source" : {
          "name" : "lisi",
          "nickname" : "lisi",
          "sex" : "男",
          "age" : 20
        },
        "sort" : [
          20
        ]
      }
    ]
  }
}
```

12. 多字段排序
假定我们想要结合使用 age 和 _score 进行查询,并且匹配的结果首先按照年龄排序,然后按照相关性得分排序

```zsh
curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "query": {         
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },
    {
      "_score": {
        "order": "desc"
      }
    }
  ]
}'
```

13. 高亮查询
在进行关键字搜索时,搜索出的内容中的关键字会显示不同的颜色,称之为高亮。
Elasticsearch 可以对查询内容中的关键字部分,进行标签和样式(高亮)的设置。
在使用 match 查询的同时,加上一个 highlight 属性:
- pre_tags:前置标签
- post_tags:后置标签
- fields:需要高亮的字段
- title:这里声明 title 字段需要高亮,后面可以为这个字段设置特有配置,也可以空

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "query": {         
    "match": {
      "name": "zhangsan"
    }
  },
  "highlight": {
    "pre_tags": "<font color=red>",
    "post_tags": "</font>",
    "fields": {
      "name": {}
    }
  }
}'
{
  "took" : 21,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.3862942,
    "hits" : [
      {
        "_index" : "student_v2",
        "_id" : "1001",
        "_score" : 1.3862942,
        "_source" : {
          "name" : "zhangsan",
          "nickname" : "zhangsan",
          "sex" : "男",
          "age" : 30
        },
        "highlight" : {
          "name" : [
            "<font color=red>zhangsan</font>"
          ]
        }
      }
    ]
  }
}
```

14. 分页查询
from:当前页的起始索引,默认从 0 开始。 from = (pageNum - 1) * size
size:每页显示多少条

```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "query": {         
    "match_all": {}
  },
  "sort": [
    {
    "age": {
      "order": "desc"
      }
    }
  ],
  "from": 4,
  "size": 2
}'
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "student_v2",
        "_id" : "1002",
        "_score" : null,
        "_source" : {
          "name" : "lisi",
          "nickname" : "lisi",
          "sex" : "男",
          "age" : 20
        },
        "sort" : [
          20
        ]
      }
    ]
  }
}
```

15. 聚合查询
聚合允许使用者对 es 文档进行统计分析,类似与关系型数据库中的 group by,当然还有很多其他的聚合,例如取最大值、平均值等等。
- 对某个字段取最大值 max
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "max_age":{
      "max":{"field":"age"}
    }
  },
  "size":0
}'
{
  "took" : 12,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "max_age" : {
      "value" : 50.0
    }
  }
}
```

- 对某个字段取最小值 min
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "min_age":{
      "min":{"field":"age"}
    }
  },
  "size":0
}'
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "min_age" : {
      "value" : 20.0
    }
  }
}
```

- 对某个字段求和 sum
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "sum_age":{
      "sum":{"field":"age"}
    }
  },
  "size":0
}'
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "sum_age" : {
      "value" : 170.0
    }
  }
}
```

- 对某个字段取平均值 avg
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "avg_age":{
      "avg":{"field":"age"}
    }
  },
  "size":0
}'
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_age" : {
      "value" : 34.0
    }
  }
}
```

- 对某个字段的值进行去重之后再取总数(distinct count)
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "distinct_age":{
      "cardinality":{"field":"age"}
    }
  },
  "size":0
}'
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "distinct_age" : {
      "value" : 4
    }
  }
}
```

- State 聚合
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "stats_age":{
      "stats":{"field":"age"}
    }
  },
  "size":0
}'
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "stats_age" : {
      "count" : 5,
      "min" : 20.0,
      "max" : 50.0,
      "avg" : 34.0,
      "sum" : 170.0
    }
  }
}
```

16. 桶聚合查询
桶聚和相当于 sql 中的 group by 语句
- terms 聚合,分组统计
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "age_groupby":{
      "terms":{"field":"age"}
    }
  },
  "size":0
}'
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "age_groupby" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 30,
          "doc_count" : 2
        },
        {
          "key" : 20,
          "doc_count" : 1
        },
        {
          "key" : 40,
          "doc_count" : 1
        },
        {
          "key" : 50,
          "doc_count" : 1
        }
      ]
    }
  }
}
```

- 在 terms 分组下再进行聚合
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
  "aggs":{
    "age_groupby":{
      "terms":{"field":"age"},
      "aggs":{
        "sum_age":{
          "sum":{"field":"age"}
        }
      }
    }
  },
  "size":0
}'
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "age_groupby" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 30,
          "doc_count" : 2,
          "sum_age" : {
            "value" : 60.0
          }
        },
        {
          "key" : 20,
          "doc_count" : 1,
          "sum_age" : {
            "value" : 20.0
          }
        },
        {
          "key" : 40,
          "doc_count" : 1,
          "sum_age" : {
            "value" : 40.0
          }
        },
        {
          "key" : 50,
          "doc_count" : 1,
          "sum_age" : {
            "value" : 50.0
          }
        }
      ]
    }
  }
}
```

## 重建索引
重建 `student` 索引:
1. 创建 `student_v2` 索引
```zsh
curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/student_v2
```

2. 添加新的 `mapping` 到 `student_v2`:
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/student_v2/_mapping\?pretty -H 'Content-Type: application/json' -d'
{
  "properties" : {
    "age" : {
      "type" : "long"
    },
    "name" : {
      "type" : "text"
    },
    "nickname" : {
      "type" : "text",
      "fields" : {
        "keyword" : {
          "type" : "keyword",
          "ignore_above" : 256
        }
      }
    },
    "sex" : {
      "type" : "text"
    }
  }
}
'
{
  "acknowledged" : true
}
```

3. reindex
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X POST https://localhost:9200/_reindex\?pretty -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "student"
  },
  "dest": {
    "index": "student_v2"
  }
}
'
{
  "took" : 9,
  "timed_out" : false,
  "total" : 5,
  "updated" : 0,
  "created" : 5,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

4. 删除原有 `index`
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X DELETE https://localhost:9200/student\?pretty 
{
  "acknowledged" : true
}
```

5. 设置 `alias`
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X PUT https://localhost:9200/student_v2/_alias/student\?pretty
{
  "acknowledged" : true
}
```

6. search
```zsh
-> % curl --cacert ~/record/t6/http_ca.crt -u elastic:ncscn2iA2B0138HMwL0A -X GET https://localhost:9200/student/_search\?pretty -H 'Content-Type: application/json' -d'{
    "query": {
        "match_all": {}
    }
}'
```
