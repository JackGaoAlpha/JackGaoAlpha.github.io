---
layout: post
title: Elasticsearch 实战（03）索引、文档、节点、分片
date: 2019-06-27 10:52:01.000000000 +09:00
---

![基本概念](http://ww1.sinaimg.cn/large/006tNc79ly1g4g145kf5dj30qk0dwgmo.jpg)

如上图所示，文档和分片、索引和节点，其实从开发和运维的两个不同视角下的产物。

和 RDBMS 的相关概念做一个不太准确的类比：

| RDBMS        | Elasticsearch   |
|  :----:   | :----:  |
| table     | index ( type ) |
| row       |   document   |
| column        |    field    |
| schema        |    mapping    |
| SQL        |    DSL    |

## 文档

文档是 ES 中所有可搜索数据的 **最小单位**，比如：日志文件中的日志项、一张唱片的详情等。

它会被序列化成 **JSON** 格式 （ 无需预定格式、类型可以指定也可以自动推算、可嵌套，支持数组 ）。

每个文档都有一个 **Unique ID**，可以自己指定，也可以由 ES 自动生成。

文档的元数据标注了文档相关信息：

	_index : 文档所属索引名
	_type ： 文档所以类型名
	_id ：文档唯一 id
	_score ：相关性分数
	_source : 文档原始 JSON

示例如下：

![文档](http://ww3.sinaimg.cn/large/006tNc79ly1g4g14tahrqj30nw0g20vc.jpg)

## 索引

索引是一类文档的集合，体现了逻辑空间的概念，有自己的 Mapping 定义，包含文档的字段名和字段类型。（分片体现的是物理空间的概念，索引中的数据分散在分片上）

其中，索引的Mapping 定义文档字段的类型。

```javascript
{
  "mapping": {
    "properties": {
      "@timestamp": {
        "type": "alias",
        "path": "timestamp"
      },
      "agent": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "bytes": {
        "type": "long"
      },
      ...
    }
  }
}
```

Setting 定义了不同的数据分布。

```javascript
  "settings": {
    "index": {
      "number_of_shards": "1",
      "auto_expand_replicas": "0-1",
      "provided_name": "kibana_sample_data_logs",
      "creation_date": "1561535052961",
      "number_of_replicas": "1",
      "uuid": "ZgZxux9QTfW4e-oVoP3ogA",
      "version": {
        "created": "7010099"
      }
    }
  },
```

在 ES 7.0 之前，一个索引可以设置锁哥 Type，但是从 7.0 开始一个索引只能设置一个 Type —— 也就是 `_doc` 。

7.0 弃用了接受类型的 API，引入了新的无类型 API，并移除了对 `_default_` 映射的支持。

8.0 将移除接受类型的 API。

## 索引相关操作

Kibana 中管理索引的界面：

![Kibana操作](http://ww4.sinaimg.cn/large/006tNc79ly1g4g15epyy1j31mg0u0k10.jpg)

在 Kibana 的 dev tools 下操作索引的相关 API ：

```bash
//查看索引相关信息
GET kibana_sample_data_logs

//查看索引的文档总数
GET kibana_sample_data_logs/_count

//查看前10条文档，了解文档格式
POST kibana_sample_data_logs/_search
{
}

//_cat indices API
//查看indices，使用通配符
GET /_cat/indices/kibana*?v&s=index

//查看状态为绿的索引
GET /_cat/indices?v&health=green

//按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

//查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

//How much memory is used per index?
GET /_cat/indices?v&h=i,tm&s=tm:desc
```

## 集群

ES 中不同集群通过不同名字区分，默认是 “elasticsearch”。通过配置文件或者命令行指定 `-E cluster.name=xxx` 来指定。一个集群可以有一个或多个节点。

集群的健康状态：

	Green ：主分片和副本都分配正常
	Yellow ：主分片正常，副本分片未能正常分配
	Red ： 有主分片未能正常分配，例如服务器磁盘占用超过 85% 时试图创建索引

## 节点

节点时 ES 的一个实例，本质上是一个 JAVA 进程。一台机器可以运行多个 ES 进程，但工业环境一般一天机器只运行一个 ES 实例。

每个节点都有一个名字，通过配置文件或者命令行  `-E node.name=xxx` 指定。

每个节点启动后会分配一个 ID，保存在 data 目录下。

### Master-eligible node 和 Master node

每个节点启动后都会成为 Master-eligible node，可以参与选举成为 Master node，可以设置 `node.master=false` 来禁止。

只有 Master node 能修改集群信息。每个节点都保存了集群信息，包含：

	所有节点信息
	所有索引和相关 Mapping 和 Setting
	分片的路由信息

### Data node 和 Coordinating node

Data node 保存分片的数据。

Coordinating node 接受客户端请求，将请求分发得到结果后汇总。每个节点默认都有 Coordinating node 的职责。

### 其他类型的 node

	 Hot & Warm Node
	 Machine Learning Node
	 Tribe Node

### 节点配置

在生产环境中，每个节点都应该配置成 **单一** 的节点类型。

| 节点类型        | 配置参数   |  默认值  |
| :----:   | :----:  | :----:  |
| master eligible     | node.master |   true     |
| data     | node.data |   true     |
| ingest     | node.ingest |   true     |
| coordinating only     | 无 |   每个节点默认都是 coordinating，设置其他类型为 false     |
| machine learning     | node.nl |   true ( 需 enable x-pack )    |

## 分片

主分片 （ Primary Shard ）解决水平扩展，可以将数据分布到集群内所有节点。一个分片是一个 Lucene 实例。主分片数在在索引创建时指定，不允许修改（除非 reindex）。

副本 （ Replica Shard ） 解决数据高可用，可以动态调整。

以下是一个三节点集群的分片情况：

![分片示例](http://ww3.sinaimg.cn/large/006tNc79ly1g4g15wryeyj314a0eugnu.jpg)

## 集群、节点相关操作

```bash
get _cat/nodes?v
GET /_nodes/es7_01,es7_02
GET /_cat/nodes?v
GET /_cat/nodes?v&h=id,ip,port,v,m


GET _cluster/health
GET _cluster/health?level=shards
GET /_cluster/health/kibana_sample_data_logs
GET /_cluster/health/kibana_sample_data_logs?level=shards

#### cluster state
#### The cluster state API allows access to metadata representing the state of the whole cluster. This includes information such as
GET /_cluster/state

//cluster get settings
GET /_cluster/settings
GET /_cluster/settings?include_defaults=true

GET _cat/shards
GET _cat/shards?h=index,shard,prirep,state,unassigned.reason
```

## 参考资料和相关阅读

* Elasticsearch核心技术与实战 （极客时间 阮一鸣） [https://time.geekbang.org/course/intro/197](https://time.geekbang.org/course/intro/197)

* 为什么不再支持单个Index下，多个Tyeps [https://www.elastic.co/cn/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0](https://www.elastic.co/cn/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0)

* CAT Index API [https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-indices.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-indices.html)

* CAT Nodes API [https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-nodes.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-nodes.html)

* Cluster API [https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cluster.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cluster.html)

* CAT Shards API [https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-shards.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-shards.html)




