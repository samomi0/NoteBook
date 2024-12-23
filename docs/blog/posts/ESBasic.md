---
title: ElasticSearch基本理论
date: 2024-01-01
authors: ["Mireux"]
categories: ["ElasticSearch"]
---

## 什么是ElasticSearch

它被用作全文检索、结构化搜索、分析以及这三个功能的组合：

- Wikipedia 使用 Elasticsearch 提供带有高亮片段的全文搜索，还有 search-as-you-type 和 did-you-mean 的建议。
- 卫报 使用 Elasticsearch 将网络社交数据结合到访客日志中，为它的编辑们提供公众对于新文章的实时反馈。
- Stack Overflow 将地理位置查询融入全文检索中去，并且使用 more-like-this 接口去查找相关的问题和回答。
- GitHub 使用 Elasticsearch 对1300亿行代码进行查询。
- ...

<!-- more -->

除了搜索，结合Kibana、Logstash、Beats开源产品，Elastic Stack（简称ELK）还被广泛运用在大数据近实时分析领域，包括：日志分析、指标监控、信息安全等。它可以帮助你探索海量结构化、非结构化数据，按需创建可视化报表，对监控数据设置报警阈值，通过使用机器学习，自动识别异常状况。ElasticSearch是基于Restful WebApi，使用Java语言开发的搜索引擎库类，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。其客户端在Java、C#、PHP、Python等许多语言中都是可用的。
其中Lucene是Es的底层搜索引擎库。

### 使用场景
主要功能：

- 1）海量数据的分布式存储以及集群管理，达到了服务与数据的高可用以及水平扩展；
- 2）近实时搜索，性能卓越。对结构化、全文、地理位置等类型数据的处理；
- 3）海量数据的近实时分析（聚合功能）

应用场景：

- 1）网站搜索、垂直搜索、代码搜索；
- 2）日志管理与分析、安全指标监控、应用性能监控、Web抓取舆情分析；

### 基础概念
我们还需对比结构化数据库，看看ES的基础概念，为我们后面学习作铺垫。

- Near Realtime（NRT） 近实时。数据提交索引后，立马就可以搜索到。
- Cluster 集群，一个集群由一个唯一的名字标识，默认为“elasticsearch”。集群名称非常重要，具有相同集群名的节点才会组成一个集群。集群名称可以在配置文件中指定。
- Node 节点：存储集群的数据，参与集群的索引和搜索功能。像集群有名字，节点也有自己的名称，默认在启动时会以一个随机的UUID的前七个字符作为节点的名字，你可以为其指定任意的名字。通过集群名在网络中发现同伴组成集群。一个节点也可是集群。
- Index 索引: 一个索引是一个文档的集合（等同于solr中的集合）。每个索引有唯一的名字，通过这个名字来操作它。一个集群中可以有任意多个索引。
- Mapping 映射：指在一个索引中，存放的数据的类型，类似于mysql中的表结构。
- Document 文档：被索引的一条数据，索引的基本信息单元，以JSON格式来表示。Shard 分片：在创建一个索引时可以指定分成多少个分片来存储。每个分片本身也是一个功能完善且独立的“索引”，可以被放置在集群的任意节点上。
- Replication 备份: 一个分片可以有多个备份（副本）


### 核心概念
### 索引<Index> 

**`一个索引就是一个拥有几分相似特征的文档的集合`**。比如说，你可以有一个商品数据的索引，一个订单数据的索引，还有一个用户数据的索引。**`一个索引由一个名字来标识`**`(必须全部是小写字母的)`**，**并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。

### 映射<Mapping>

**`映射是定义一个文档和它所包含的字段如何被存储和索引的过程`**。在默认配置下，ES可以根据插入的数据`自动地创建mapping，也可以手动创建mapping`。 mapping中主要包括字段名、字段类型等 

### 文档<Document>

**`文档是索引中存储的一条条数据。一条文档是一个可被索引的最小单元`**。ES中的文档采用了轻量级的JSON格式数据来表示。 

## 基本操作

### 索引

#### 查询索引

```markdown
# 查询索引
- GET /products/_cat/indices?v
```



 #### 创建索引

```markdown
# 1.创建索引
- PUT /索引名 ====> PUT /products
- 注意: 
		1.ES中索引健康转态  red(索引不可用) 、yellwo(索引可用,存在风险)、green(健康)
		2.默认ES在创建索引时回为索引创建1个备份索引和一个primary索引
```



> 如果不手动设置 那么索引为黄色 存在风险



```markdown
		
# 2.创建索引 进行索引分片配置
- PUT /products
{
  "settings": {
    "number_of_shards": 1, #指定主分片的数量
    "number_of_replicas": 0 #指定副本分片的数量
  }
}
```



#### 删除索引

最简单

```markdown
# 删除索引
- DELETE /products
```





### 映射<Mapping>

#### 创建

**映射的类型：**

字符串类型: keyword 关键字 关键词 、text 一段文本

数字类型：integer long  

小数类型：float double

布尔类型：boolean 

日期类型：date

```markdown
# 创建索引&映射
```

索引和映射可以同时创建

我们创建一个商品映射作为后面的例子

其中包括 title 、 price 、 created_at 、 description

```json
# 创建索引的同时创建映射
PUT /products
{
  "settings": {
    "number_of_replicas": 0, 
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "keyword"
      },
      "price": {
        "type": "float"
      },
      "created_at": {
        "type": "date"
      },
      "description": {
        "type": "text"
      }
    }
  }
}
```




#### 查询

```markdown
# 查询某个索引的映射
- GET /索引名/_mapping =====> GET /products/_mapping
```



### 文档<document>

#### 添加文档

```http
# 添加文档
  POST /products/_doc/1 #指定id生成文档
  {
    "title": "Iphone XR",
    "created_at": "2018-09-01",
    "price": "4999",
    "description": "这是一部Iphone XR 手机"
  }

```

> 指定id添加





> 不指定id 添加



#### 查询文档

```http
# 查询文档
GET /products/_doc/1
```





#### 删除文档

```json
# 删除文档
DELETE /products/_doc/bLw1qX0BViY44oHwqzJq
```

```json
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "bLw1qX0BViY44oHwqzJq",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}

```

#### 更新文档

更新文档有两种方式：

- 一种是PUT /索引名/_doc/id 这种方式是先删除文档内容，再插入更新内容
- 一种是POST /索引名/_doc/id/ _update 这种是在原先的文档内容上更新





> 我们先插入一条数据

```json
POST /products/_doc
{
  "title":"Iphone 4",
  "created_at": "2014-09-01",
  "price": "499",
  "description": "这是一部过时了的Iphone 4手机"
}



{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "bbw6qX0BViY44oHwSzKn",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}

```



> 第一种方式更新：

```json
# 第一种方式更新 不保存原文档的内容
PUT /products/_doc/bbw6qX0BViY44oHwSzKn
{
  "title": "Iphone15"
}


{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "bbw6qX0BViY44oHwSzKn",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

查询该文档之后：




> 第二种方式

前面是一样的插入一条数据：

```json
POST /products/_doc
{
  "title":"Iphone 4",
  "created_at": "2014-09-01",
  "price": "499",
  "description": "这是一部过时了的Iphone 4手机"
}



{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "bbw6qX0BViY44oHwSzKn",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}

```

```json
# 第二种方式更新 在原来的文档上进行更新
POST /products/_doc/brw9qX0BViY44oHwbDLo/_update 
{
 "doc": {
  "title": "Iphone15",
  "price": "4000"
 }
}
```

数据更新完成，原来的数据依然存在。

#### 批量操作

#### 批量操作

```http
POST /products/_doc/_bulk #批量索引两条文档
 	{"index":{"_id":"1"}}
  		{"title":"iphone14","price":8999.99,"created_at":"2021-09-15","description":"iPhone 13屏幕采用6.8英寸OLED屏幕"}
	{"index":{"_id":"2"}}
  		{"title":"iphone15","price":8999.99,"created_at":"2021-09-15","description":"iPhone 15屏幕采用10.8英寸OLED屏幕"}
```


```http
POST /products/_doc/_bulk #更新文档同时删除文档
	{"update":{"_id":"1"}}
		{"doc":{"title":"iphone17"}}
	{"delete":{"_id":2}}
	{"index":{}}
		{"title":"iphone19","price":8999.99,"created_at":"2021-09-15","description":"iPhone 19屏幕采用61.8英寸OLED屏幕"}
```

> 说明:批量时不会因为一个失败而全部失败,而是继续执行后续操作,在返回时按照执行的状态返回!

