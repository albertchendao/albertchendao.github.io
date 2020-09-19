---
layout: article
title: ElasticSearch RESTful 接口
tags: []
key: 5a8bed76-c005-487c-b580-29661ec9f48b
---

ElasticSearch 提供了 RESTful 接口进行操作,这里做下整理.

<!--more-->

## _cat

`_cat` 相关的可以用来查看 ES 的各种状态,可以用 `curl localhost:9200/_cat` 查看支持的命令:

```bash
$ curl localhost:9200/_cat
# 查看每个节点上的分片数(shards),以及每个节点剩余磁盘
/_cat/allocation
# 查看分片
/_cat/shards
/_cat/shards/{index}
# 查看集群主节点
/_cat/master
# 查看集群节点与剩余磁盘
/_cat/nodes
# 查看索引
/_cat/indices
/_cat/indices/{index}
# 查看索引的分段
/_cat/segments
/_cat/segments/{index}
# 查看文档数量
/_cat/count
/_cat/count/{index}
# 查看索引分片的恢复视图, 有分片迁移时会发生恢复事件
/_cat/recovery
/_cat/recovery/{index}
# 查看集群健康状态
/_cat/health
# 查看被挂起的任务
/_cat/pending_tasks
# 查看别名
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
# 查看每个节点自定义属性
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
# 查看索引模板
/_cat/templates
```

## _aliases

### 查询别名

```bash
GET index/_aliases
```

### 建立别名

```bash
POST _aliases
{
    "actions": [
        {"add": {"index": "test1", "alias": "alias1"}}
    ]
}
```

### 删除别名

```bash
POST _aliases
{
    "actions": [
        {"remove": {"index": "test1", "alias": "alias1"}}
    ]
}
```

## _analyze

用来查看分词结果

### 指定 analyzer

```bash
# GET _analyze?pretty&analyzer=ik_smart&text=腾讯科技(深圳)有限公司
POST _analyze
{
    "analyzer": "stardard", #指定的分词器
    "text": "hello world" #要进行分析的文本
}
```

### 索引中的 analyzer

```bash
POST index_name/_analyze
{
    "field":"firtname", #要进行分析的索引中的字段
    "text":"ni cai" #要进行分析的文本
}
```

### 自定义 analyzer

```bash
POST _analyze
{
    "tokenizer":"standard", #指定的分词器
    "filter":["lowercase"],  #对分析后的词最后进行小写转换
    "text":"hello world" #要进行分析的文本
}
```

## 文档

### 查询指定 id 文档

```bash
GET index/type/id
```

### 查询指定字段的分词结果

```bash
GET index/type/id/_termvectors?fields=fieldsName
```

## _search

`_search` 使用 json 进行操作,例如:

```bash
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} }
}'
```

返回内容也是个 json.

```json
{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "_score" : 1.0, "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }]
    }
}
```

* `took` : 查询花费的时间,毫秒单位
* `time_out` : 标识查询是否超时
* `_shards` : 描述了查询分片的信息,查询了多少个分片、成功的分片数量、失败的分片数量等
* `hits` : 搜索的结果,total是全部的满足的文档数目,hits是返回的实际数目（默认是10）
* `_score` : 是文档的分数信息,与排名相关度有关,参考各大搜索引擎的搜索结果,就容易理解.

常用的查询有如下:

### Term Query

直接使用关键字进行查询,不对关键字进行分词, 文档中必须包含整个搜索的词:

```json
{
  "from": 0,
  "size": 10,
  "query": {
    "term": {
      "pkg": "com.youku.phone"
    }
  },
  "sort": {
    "date": {
      "order": "desc"
    }
  }
}
```

### Terms Query

指定多个关键字进行查询, 匹配其一即可.

当 term 的个数少的时候,termsQuery  等效为多个 termQuery 使用 boolQuery 使用 or 操作符连接起来；
当 term 的个数多的时候,termsQuery 查询创建一个位集的方式进行查询,效率会比普通的 bool 方式好一些

```json
{
  "from": 0,
  "size": 10,
  "query": {
    "terms": {
      "tags": ["novel","book"]
    }
  }
}
```

### Match Query

匹配的时候,会将查询的关键字进行分词,合并每个分词后的查询结果:

```json
{
  "from": 0,
  "size": 10,
  "query": {
    "match":{
      "content" : {
        "query" : "我的宝马多少马力"
      }
    }
  }
}
```

### Match Phrase Query

在 Match Query 的基础上更进一步,筛选所有分词都匹配的结果:

```json
{
  "from": 0,
  "size": 10,
  "query": {
    "match_phrase": {
      "content" : {
        "query" : "我的宝马多少马力"
      }
    }
  }
}
```

### Multi Match Query

多个字段进行匹配:

```json
{
  "query": {
    "multi_match": {
        "query" : "我的宝马多少马力",
        "fields" : ["title", "content"]
    }
  }
}
```

### Wildcard Query

支持 * 和 ?:

```json
{
    "query": {
        "wildcard": {
             "title": "cr?me"
        }
    }
}
```

### Range Query

```json
{
    "query": {
        "range": {
             "year": {
                  "gte" :1890,
                  "lte":1900
              }
        }
    }
}
```

### Regex Query

```json
{
    "query": {
        "regexp": {
             "title": {
                  "value" :"cr.m[ae]",
                  "boost":10.0
              }
        }
    }
}
```

### Query String

将查询语句组装成字符串传递给 ES:

```json
{
  "query" : {
    "query_string" : {
      "query" : "\"com.tencent.mm\"",
      "fields" : [ "pkg" ]
    }
  }
}
```

### Exists Query

没有 Nested Object 类型下:

```json
 {
  "from" : 0,
  "size" : 10,
  "query" : {
    "exists":{
        "field":"age"
    }
  }
}
```

有 Nested Object 类型下:

```json
 {
  "from" : 0,
  "size" : 10,
  "query" : {
      "nested":{
          "path":"apps",
          "query":{
            "exists":{
                "field":"apps.scene_class"
            }
          }
      }
  }
}
```

### Bool Query

Bool Query 可以将很多查询条件组合起来, 组合条件支持 `must`, `must_not`, `should`, `filter`

```json
{
  "query" : {
    "bool" : {
      "must" : [ {
        "query_string" : {
          "query" : "\"com.tencent.mm\"",
          "fields" : [ "pkg" ]
        }
      }, {
        "query_string" : {
          "query" : "\"com.xiaomi.channel\"",
          "fields" : [ "pkg" ]
        }
      } ]
    }
  }
}
```

有参数 `minimum_should_match` 可以用来限制匹配的数量:

```json
{
  "query": {
    "bool": {
      "minimum_should_match": 2,
      "should": [
        {
          "term": {
            "title": "java"
           }
        },
        {
          "term": {
            "title": "编程"
          }
        },
        {
          "term": {
            "title":  "思想"
          }
        }
      ]
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```

### 父子查询

```javascript
//查询某文档,只有该文档有"父文档"且满足一定条件才算匹配
{"has_parent": {                //文档是否有 parent
      "type": "branch",         //其 parent 所在 type 必须是 branch
      "query": {                //其 parent 必须满足以下 query 条件
        "match": {
          "country": "UK"
        }
      }
    }                           //如果满足以上条件,hit 该文档
}
//查询某文档,只有该文档有"子文档"且满足一定条件才算匹配
{
"has_child": {                       //文档是否有 child
      "type":       "employee",      //其 child所在 type 必须是 employee
      "query": {                     //其 parent 必须满足以下 query 条件
        "match": {
          "name": "Alice Smith"
        }
      }
    }                                //如果满足以上条件,hit 该文档
}
```

### 聚合查询

聚合查询分为 metric 聚合和 Bucketing 聚合, 两者又可以嵌套起来使用, 查询格式如下:

```javascript
"aggregations" : {                  // 表示聚合操作,可以使用aggs替代
    "<aggregation_name>" : {        // 聚合名,可以是任意的字符串.用做响应的key,便于快速取得正确的响应数据.
        "<aggregation_type>" : {    // 聚合类别,就是各种类型的聚合,如min等
            "<aggregation_body>":{}// 聚合体,不同的聚合有不同的body
        }
        [,"aggregations" : {  }]    // 嵌套的子聚合,可以有0或多个
    }
    [,"<aggregation_name_2>" : {  } ] // 另外的聚合,可以有0或多个
}
```

返回结果格式为:

```javascript
{
"aggregations": {                    // 聚合结果
    "<aggregation_name>": {          // 前面输入的聚合名
       "<aggregation_body>":{}             // 聚合后的数据 不同的聚合格式可能不同
    }
  }
}
```

聚合也可以嵌套使用 `filter`, 如下中内部嵌套的聚合是经过 `filter` 过滤后的文档集合中进行聚合:

```json
{
    "aggregations" : {
        "<aggregation_name>" : {
            "filter" : "<query_body>",
            "aggregations" : {
                "<aggregation_name>" : "<aggregation_body>"
            }
        }
    }
}
```

也可以单独只使用 `filter`:

```json
 {
  "from" : 0,
  "size" : 0,
  "query" : {
    "bool" : { }
  },
  "aggregations" : {
        "生活消费" : {
            "filter" : {
                "nested" : {
                    "path" : "apps",
                    "query" : {
                            "term":{
                                "apps.scene_class":"生活消费"
                            }
                        }
                    }
                }
            }
      }
}
```

#### metric 聚合

metric 聚合用于对数字类型的数据进行计算.

##### Min Aggregation

获取最小值

```json
{
  "query": {},
  "aggs": {
    "min_age": {
      "min": {
        "field": "age"
      }
    }
  }
}
```

##### Max Aggregation

获取最大值

```json
{
  "query": {},
  "aggs": {
    "max_age": {
      "max": {
        "field": "age"
      }
    }
  }
}
```

##### Sum Aggregation

求和

```json
{
  "query": {},
  "aggs": {
    "sum_age": {
      "sum": {
        "field": "age"
      }
    }
  }
}
```

##### Avg Aggregation

计算平均值

```json
{
  "query": {},
  "aggs": {
    "avg_age": {
      "avg": {
        "field": "age"
      }
    }
  }
}
```

NOTE: 计算的时候字段不存在的文档将被忽略, 也就是分子不算其值, 分母不 +1, 除非使用 `"avg":{"field":"age","missing":10}` 设置其默认值

#### Bucketing 聚合

普通分组

```json
{
  "query": {},
  "aggs": {
    "brand": {
      "terms": {
        "field": "brand"
      }
    },
    "size":50
  }
}
```

#### Nested 聚合

```json
{
 "size": 0,
 "aggs": {
  "apps": {
   "nested": {
    "path": "apps"
   },
   "aggs": {
    "master": {
     "terms": {
      "field": "apps.pkg"
     },
     "aggs": {
      "child": {
       "terms": {
        "field": "apps.times"
       }
      }
     }
    }
   }
  }
 }
}
```

```json
{
 "from": 0,
 "size": 0,
 "aggregations": {
  "apps": {
   "nested": {
    "path": "apps"
   },
   "aggregations": {
    "master": {
     "terms": {
      "field": "apps.pkg",
                       "size": 10,
                       "collect_mode" : "breadth_first"
     },
     "aggregations": {
      "reverse": {
       "reverse_nested": {},
       "aggregations": {
        "child": {
         "terms": {
          "field": "gender"
         }
        }
       }
      }
     }
    }
   }
  }
 }
}
```
