# Elasticsearch

Elastic 是 Lucene 的封装，提供了 REST API 的操作接口，开箱即用。

官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

## 一、基本概念

### 1、Index（索引）

动词，相当于 MySQL 中的 insert； 名词，相当于 MySQL 中的 Database。

### 2、Type（类型）

在 Index（索引）中， 可以定义一个或多个类型。 类似于 MySQL 中的 Table；每一种类型的数据放在一起。

### 3、Document（文档）

保存在某个索引（Index）下，某种类型（Type）的一个数据（Document），文档是 JSON 格式的， Document 就像是 MySQL 中的某个 Table 里面的内容；

## 二、初步检索

### 1、_cat

查看所有节点：

```http
GET /_cat/nodes
```

查看 es 健康状况：

```http
GET /_cat/health
```

查看主节点：

```http
_GET /_cat/master
```

查看所有索引：

```http
GET /_cat/indices
```

### 2、索引一个文档（保存）

格式：
```http
POST 索引/类型
或
PUT 索引/类型/id
```

实例：

```http
POST qwz_test/one
{
  "name":"qwz"
}
或
PUT qwz_test/one/1
{
  "name":"qwz"
}
```

结果：

```json
{
  "_index" : "qwz_test",
  "_type" : "one",
  "_id" : "lfPimYABwSZhcibz4j0O",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

说明：

PUT 和 POST 都可以， POST 新增。如果不指定 id， 会自动生成 id。 指定 id 就会修改这个数据，并新增版本号

PUT 可以新增可以修改。 PUT 必须指定 id；由于 PUT 需要指定 id， 我们一般都用来做修改 操作，不指定 id 会报错。

### 3、查询文档

格式：

```http
GET 索引/类型/id
```

实例：

```http
GET qwz_test/one/1
```

结果：

```json
{
  "_index" : "qwz_test", //索引
  "_type" : "one", //类型
  "_id" : "1", //id
  "_version" : 3, //版本号
  "_seq_no" : 2, //并发控制字段，每次更新就会+1，用来做乐观锁
  "_primary_term" : 1, //同上，主分片重新分配，如重启，就会变化
  "found" : true,
  "_source" : { //内容
    "name" : "qwz"
  }
}
```

如果要做乐观锁更新数据时携带 `?if_seq_no=0&if_primary_term=1`

### 4、更新文档

格式：

```http
PUT 索引/类型/id
{
    "key":"value"
}
或
POST 索引/类型/id
{
    "key":"value"
}
或
POST 索引/类型/id/__update
{
  "doc":{
    "key":"value"
  }
}
```

实例：

```http
PUT qwz_test/one/1
{
  "name":"qwz_put"
}
```

说明：

POST 操作会对比源文档数据，带_update 对比元数据如果一样就不进行任何操作。PUT 操作总会将数据重新保存并增加 version 版本。 

对于大并发更新，不带 update；对于大并发查询偶尔更新，带 update；对比更新，重新计算分配规则。

### 5、删除文档&索引

格式：

```http
删除文档
DELETE 索引/类型/id
删除索引
DELETE 索引
```

实例：

```http
DELETE qwz_test/one/lfPimYABwSZhcibz4j0O
DELETE qwz_test
```

### 6、bulk 批量 API

格式：

```json
{ action: { metadata }}
{ request body }
{ action: { metadata }} 
{ request body }
```

实例：

```http
POST qwz_test/one/_bulk
{"index":{"_id":1}}
{"name":"qinwz"}
{"index":{"_id":"2"}}
{"name":"qinweizhao"}
```

复杂实例：

```http
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title": "My first blog post" } 
{ "index": { "_index": "website", "_type": "blog" }}
{ "title": "My second blog post" } 
{ "update": { "_index": "website", "_type": "blog", "_id": "123"} } 
{ "doc" : {"title" : "My updated blog post"} }
```

bulk API 以此按顺序执行所有的 action（动作）。 如果一个单个的动作因任何原因而失败，它将继续处理它后面剩余的动作。当 bulk API 返回时，它将提供每个动作的状态（与发送的顺序相同），所以您可以检查是否一个指定的动作是不是失败了。

## 四、进阶检索

### 1、SearchAPI

ES 支持两种基本方式检索：一种是通过使用 REST request URI 发送搜索参数（uri+检索参数），另一种是通过使用 REST request body 来发送它们（uri+请求体）。

检索信息

检索 bank 下所有信息，包括 type 和 docs：

```http
GET bank/_search
```

#### 1. 请求参数方式检索

```http
GET bank/_search?q=*&sort=account_number:asc
```

结果：

```json
{
  "took" : 1, //执行搜索的时间（毫秒）
  "timed_out" : false, //是否超时
  "_shards" : { //多少个分片被搜索了，以及统计了成功/失败的搜索分片
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : { //搜索结果
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : 1.0, //最高得分（全文检索用）
    "hits" : [{ //实际的搜索结果数组（默认为前 10 的文档）
        "_index" : "bank",
        "_type" : "account",
        "_id" : "1",
        "_score" : 1.0, //相关性得分
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        }
      }]
  }
}
```

#### 2. uri+请求体进行检索

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "account_number": {
        "order": "desc"
      }
    }
  ]
}
```

HTTP 客户端工具（POSTMAN），Get 请求不能携带请求体，但是变为 Post 也是一样的，POST 一个 JSON 风格的查询请求体到 _search API。 一旦搜索的结果被返回， Elasticsearch 就完成了这次请求，并且不会维护任何服务端的资源或者结果的 cursor（游标）。

### 2、Query DSL

#### 1. 基本语法格式

Elasticsearch 提供了一个可以执行查询的 Json 风格的 DSL（domain-specific language 领域特 定语言）。这个被称为 Query DSL。 该查询语言非常全面，并且刚开始的时候感觉有点复杂， 真正学好它的方法是从一些基础的示例开始的。 

一个查询语句典型结构：

```json
{
	QUERY_NAME: { 
    ARGUMENT: VALUE, ARGUMENT: VALUE,... 
  }
}
```

如果是针对某个字段，那么它的结构如下：
```json
{
  QUERY_NAME: { 
    FIELD_NAME: { 
      ARGUMENT: VALUE, ARGUMENT: VALUE,...
    }
  }
}
```

实例：

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 20, 
  "sort": [
    {
      "account_number": {
        "order": "desc"
      }
    }
  ]
}
```

query 定义如何查询，match_all 查询类型【代表查询所有的所有】，ES 中可以在 query 中组合非常多的查询类型完成复杂查询除了 query 参数之外，我们也可以传递其它的参数以改变查询结果。如 sort,size,from+size 限定，完成分页功能 sort 排序，多字段排序，会在前序字段相等时后续字段内部排序，否则以前序为准。

#### 2. 返回部分字段

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 20, 
  "_source": ["account_number","lastname"], 
  "sort": [
    {
      "account_number": {
        "order": "desc"
      }
    }
  ]
}
```

#### 3. match【匹配查询】

基本类型（非字符串），精确匹配：


```http
GET bank/_search
{
 "query": {
   "match": {
     "account_number": 25
   }
 } 
}
```

> match 返回 account_number=20 的。

字符串，全文检索：

```http
GET bank/_search
{
 "query": {
   "match": {
     "address": "mill"
   }
 } 
}
```

>最终查询出 address 中包含 mill 单词的所有记录，match 当搜索字符串类型的时候，会进行全文检索，并且每条记录有相关性得分。

字符串，多个单词（分词+全文检索）：

```http
GET bank/_search
{
 "query": {
   "match": {
     "address": "mill road"
   }
 } 
}
```

> 最终查询出 address 中包含 mill 或者 road 或者 mill road 的所有记录，并给出相关性得分
>

#### 4. match_phrase【短语匹配】

将需要匹配的值当成一个整体单词（不分词）进行检索

```http
GET bank/_search { "query": { "match_phrase": { "address": "mill road" } } }
```

> 查出 address 中包含 mill road 的所有记录，并给出相关性得分

#### 5. multi_match【多字段匹配】

> state 或者 address 包含 mill

#### 6. bool【复合查询】

复合语句可以合并**任何**其它查询语句，包括复合语句，这一点是很重要的。这就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

**must**：必须达到 must 列举的所有条件；**should**：应该达到 should 列举的条件，如果达到会增加相关文档的评分，并不会改变查询的结果。如果 query 中只有 should 且只有一种匹配规则，那么 should 的条件就会被作为默认匹配条件而去改变查询结果；**must_not** ：必须不是指定的情况。

```http
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        },
        {
          "match": {
            "gender": "M"
          }
        }
      ],
      "should": [
        {
          "match": {
            "address": "lane"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "email": "baluba.com"
          }
        }
      ]
    }
  }
}
```

> address 包含 mill， 并且 gender 是 M， 如果 address 里面有 lane 最好不过，但是 email 必 须不包含 baluba.com
> 

![2022-05-07_005959](https://img.qinweizhao.com/2022/05/2022-05-07_005959.png)

#### 7. filter【结果过滤】

并不是所有的查询都需要产生分数，特别是那些仅用于 “filtering”（过滤）的文档。为了不 计算分数 Elasticsearch 会自动检查场景并且优化查询的执行。

```http
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "filter": {
        "range": {
          "balance": {
            "gte": 10000,
            "lte": 20000
          }
        }
      }
    }
  }
}
```

#### 8. term

和 match 一样。匹配某个属性的值。全文检索字段用 match，其他非 text 字段匹配用 term。

```http
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "age": {
              "value": "28"
            }
          }
        },
        {
          "match": {
            "address": "990 Mill Road"
          }
        }
      ]
    }
  }
}
```

#### 9. aggregations（执行聚合）

聚合提供了从数据中分组和提取数据的能力。 最简单的聚合方法大致等于 SQL GROUP BY 和 SQL 聚合函数。在 Elasticsearch 中，您有执行搜索返回 hits（命中结果），并且同时返 回聚合结果，把一个响应中的所有 hits（命中结果）分隔开的能力。这是非常强大且有效的， 您可以执行查询和多个聚合，并且在一次使用中得到各自的（任何一个的）返回结果，使用 一次简洁和简化的 API 来避免网络往返。

搜索 address 中包含 mill 的所有人的年龄分布以及平均年龄，但不显示这些人的详情：

```http
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "size": 0, 
  "aggs": {
    "group_by_age": {
      "terms": {
        "field": "age"
      }
    },
    "avg_age":{
      "avg": {
        "field": "age"
      }
    }
  }
}
```

按照年龄聚合，并且请求这些年龄段的这些人的平均薪资：

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "age_avg": {
      "terms": {
        "field": "age",
        "size": 1000
      },
      "aggs": {
        "banlances_avg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 1000
}
```

查出所有年龄分布，并且这些年龄段中 M 的平均薪资和 F 的平均薪资以及这个年龄 段的总体平均薪资：

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "age_agg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "gender_agg": {
          "terms": {
            "field": "gender.keyword",
            "size": 100
          },
          "aggs": {
            "balance_avg": {
              "avg": {
                "field": "balance"
              }
            }
          }
        },
        "balance_avg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 1000
}
```

### 3、Mapping

#### 1. 字段类型

![2022-05-07_014925](https://img.qinweizhao.com/2022/05/2022-05-07_014925.png)

[Field data types | Elasticsearch Guide [7.17\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/mapping-types.html)

#### 2. 映射

**Mapping 是用来定义一个文档（document）**，以及它所包含的属性（field）是如何存储和索引的。比如，使用 mapping 来定义：

- 哪些字符串属性应该被看做全文本属性（full text fields）。 
- 哪些属性包含数字，日期或者地理位置。 
- 文档中的所有属性是否都能被索引（_all 配置）。 
- 日期的格式。 
- 自定义映射规则来执行动态添加属性。

查看映射信息：

```
GET bank/_mapping
```

创建映射：

```http
PUT /myindex
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "age":{
        "type": "long"
      },
      "email":{
        "type": "keyword"
      }
    }
  }
}
```

添加新的字段映射：

```http
PUT /myindex/_mapping
{
 "properties": {
      "gender":{
        "type": "keyword"
      }
    }
}
```

更新映射：

对于已经存在的映射字段，我们不能更新。更新必须创建新的索引进行数据迁移。

#### 3. 新版本改变

关系型数据库中两个数据表示是独立的，即使他们里面有相同名称的列也不影响使用， 但 ES 中不是这样的。 elasticsearch 是基于 Lucene 开发的搜索引擎， 而 ES 中不同 type 下名称相同的 filed 最终在 Lucene 中的处理方式是一样的。

- 两个不同 type 下的两个 user_name， 在 ES 同一个索引下其实被认为是同一个 filed， 你必须在两个不同的 type 中定义相同的 filed 映射。否则，不同 type 中的相同字段名称就会在处理中出现冲突的情况，导致 Lucene 处理效率下降。
- 去掉 type 就是为了提高 ES 处理数据的效率。

Elasticsearch 7.x 版本 URL 中的 type 参数为可选。比如，索引一个文档不再要求提供文档类型。

Elasticsearch 8.x 版本不再支持 URL 中的 type 参数。

解决：

将索引从多类型迁移到单类型， 每种类型文档一个独立索引。将已存在的索引下的类型数据， 全部迁移到指定位置即可。

补充：

数据迁移

先创建出 new_twitter 的正确映射。然后使用如下方式进行数据迁移：

```http
POST _reindex
{
  "source": {
    "index": "twitter",
    "type": "tweet"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

其中，type 可省略。

### 4、分词

一个 tokenizer（分词器）接收一个字符流，将之分割为独立的 tokens（词元，通常是独立 的单词），然后输出 tokens 流。 例如，whitespace tokenizer 遇到空白字符时分割文本。它会将文本 "Quick brown fox!" 分割 为 [Quick, brown, fox!]。 该 tokenizer（分词器）还负责记录各个 term（词条）的顺序或 position 位置（用于 phrase 短 语和 word proximity 词近邻查询），以及 term（词条）所代表的原始 word（单词）的 start （起始）和 end（结束）的 character offsets（字符偏移量）（用于高亮显示搜索的内容）。 Elasticsearch 提供了很多内置的分词器，可以用来构建 custom analyzers（自定义分词器）。

#### 1. 安装 ik 分词器

GitHub 地址：[Releases · medcl/elasticsearch-analysis-ik (github.com)](https://github.com/medcl/elasticsearch-analysis-ik/releases)

选择与 ES 对应的版本，解压到容器内部的 plugin 目录下，并且更名为 ik。

可以确认是否安装好了分词器 ：

```sh
 cd ../bin
```

列出系统的分词器

```sh
elasticsearch plugin list
```

#### 2. 测试分词器

使用默认：

```http
POST _analyze 
{ 
  "text": " 我是中国人" 
}
```

使用分词器：

```http
POST _analyze 
{ 
  "analyzer": "ik_smart",
  "text": "我是中国人"
}
```

另外一个分词器 ik_max_word：

```http
POST _analyze 
{ 
  "analyzer": "ik_max_word",
  "text": "我是中国人"
}
```

能够看出不同的分词器，分词有明显的区别，所以以后定义一个索引不能再使用默认的 mapping 了，要手工建立 mapping，因为要选择分词器。

#### 3. 自定义词库

修改**/usr/share/elasticsearch/plugins/ik/config/**中的 **IKAnalyzer.cfg.xml**。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict"></entry>
	<!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
  <!-- 更改此处 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

注意：

更新完成后， es 只会对新增的数据用新词分词。历史数据是不会重新分词的。如果想要历 史数据重新分词。需要执行：

```http
POST 索引/_update_by_query?conflicts=proceed
```

