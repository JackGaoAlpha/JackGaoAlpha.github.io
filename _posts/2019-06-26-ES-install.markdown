---
layout: post
title: Elasticsearch 实战（01）综述
date: 2019-06-26 19:46:01.000000000 +09:00
---
## Elasticsearch 安装和简单配置

ES 安装的地址请 [点击这儿](https://www.elastic.co/cn/downloads/elasticsearch)

ES 在 7.0 开始内置了 JAVA 环境，不需要像之前版本一样设置 $JAVA_HOME。

### 目录结构

| 目录        | 配置   |  描述  |
| :----:    | :----:   | :----:  |
| bin     |  |   脚本文件，包括启动 ES、安装插件、运行统计数据等     |
| config        |   elasticsearch.yml   |   集群配置文件，user 、role based 相关配置   |
| JDK     |    path.data    |  JAVA 运行环境  |
| data     |        |  数据文件  |
| lib     |        |  JAVA 类库  |
| log     |    path.log    |  日志文件  |
| modules     |        |  包含所有 ES 模块 |
| pliugin     |        |  包含素有已安装插件  |

### JVM 配置

* 修改 JVM  —— config/jvm.options

* 配置建议：Xms 和 Xmx 配置成一样，不要超过机器内存的 50%，不要超过 30 GB

### 测试

```bash
//启动单节点
bin/elasticsearch

//安装插件
bin/elasticsearch-plugin install analysis-icu
//查看插件
bin/elasticsearch-plugin list
//查看安装的插件
GET http://localhost:9200/_cat/plugins?v

// 本机启动多节点集群 （-d 表示后台运行）
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -d
bin/elasticsearch -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -d
bin/elasticsearch -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -d
bin/elasticsearch -E node.name=node3 -E cluster.name=geektime -E path.data=node3_data -d

//查看集群
GET http://localhost:9200

//查看nodes
GET _cat/nodes
GET _cluster/health
```


## Kibana 安装

Kibana 下载地址请 [点击这儿](https://www.elastic.co/cn/downloads/kibana)

```bash
// 启动 kibana
bin/kibana

// 插件
bin/kibana-plugin list
bin/kibana-plugin install plugin_location
bin/kibana remove

//查看
GET http://localhost:5601

```

## 在 Docker 中运行 ES、Kibana 和 Cerebro

切换到你想要的文件夹，新建一个文件 `docker-compose.yaml` ，内容如下：

```bash
version: '2.2'
services:
  cerebro:
    image: lmenezes/cerebro:0.8.3
    container_name: cerebro
    ports:
      - "9000:9000"
    command:
      - -Dhosts.0.host=http://elasticsearch:9200
    networks:
      - es7net
  kibana:
    image: docker.elastic.co/kibana/kibana:7.1.0
    container_name: kibana7
    environment:
      - I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - es7net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_01
    environment:
      - cluster.name=geektime
      - node.name=es7_01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - es7net
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_02
    environment:
      - cluster.name=geektime
      - node.name=es7_02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data2:/usr/share/elasticsearch/data
    networks:
      - es7net


volumes:
  es7data1:
    driver: local
  es7data2:
    driver: local

networks:
  es7net:
    driver: bridge
```

用 docker-compose 启动：

```bash
//启动
docker-compose up

//停止容器
docker-compose down

//停止容器并且移除数据
docker-compose down -v

```

## Logstach 的安装和数据导入

Logstash 下载地址请 [点击这儿](https://www.elastic.co/cn/downloads/logstash)

```bash
//下载与ES相同版本号的logstash，并解压到相应目录
//修改movielens目录下的logstash.conf文件
//path修改为实际的movies.csv路径
input {
  file {
    path => "YOUR_FULL_PATH_OF_movies.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

//启动Elasticsearch实例，然后启动 logstash，并制定配置文件导入数据
bin/logstash -f /YOUR_PATH_of_logstash.conf
```

## 参考资料和相关阅读

* Elasticsearch核心技术与实战 （极客时间 阮一鸣） [https://time.geekbang.org/course/intro/197](https://time.geekbang.org/course/intro/197)

* ES 安装 [https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)
* ES 配置 [https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)
* ES 重要配置 [https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)

* 初识 Kibana [https://www.elastic.co/guide/en/kibana/current/setup.html](https://www.elastic.co/guide/en/kibana/current/setup.html)

* Kibana 插件 [https://www.elastic.co/guide/en/kibana/current/known-plugins.html](https://www.elastic.co/guide/en/kibana/current/known-plugins.html)


* Elasticsearch on Kuvernetes [https://www.elastic.co/cn/blog/introducing-elastic-cloud-on-kubernetes-the-elasticsearch-operator-and-beyond](https://www.elastic.co/cn/blog/introducing-elastic-cloud-on-kubernetes-the-elasticsearch-operator-and-beyond)
* CAT Plugins API [https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-plugins.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cat-plugins.html)


* 安装 Docker [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

* 安装 docker-compose [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

* 如何创建自己的 Docker Image - [https://www.elastic.co/cn/blog/how-to-make-a-dockerfile-for-elasticsearch](https://www.elastic.co/cn/blog/how-to-make-a-dockerfile-for-elasticsearch)

* 如何在为 Docker Image 安装 Elasticsearch 插件 - [https://www.elastic.co/cn/blog/elasticsearch-docker-plugin-management](https://www.elastic.co/cn/blog/elasticsearch-docker-plugin-management)

* 如何设置 Docker 网络 - [https://www.elastic.co/cn/blog/docker-networking](https://www.elastic.co/cn/blog/docker-networking)

* Cerebro 源码 [https://github.com/lmenezes/cerebro](https://github.com/lmenezes/cerebro)