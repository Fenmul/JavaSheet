### ES

基于 lucene 开发的分布式全文检索引擎

#### 安装

##### JVM 配置

- 修改 JVM -config/jvm.options
  - 7.1 默认设置是 1 GB
- 配置
  - Xmx 和 Xms 设置成一样的
  - Xmx 不要超过内存的 50%
  - 不要超过 30GB 

##### ES启动

```
// 打开 9200 端口查看
bin/elasticsearch
// 查看安装了哪些插件
bin/elasticsearch-plugin list
// 安装分词插件
bin/elasticsearch-plugin install analysis-icu
```

##### Kibana

```
// 启动 kibana
bin/kibana

// 查看插件
bin/kibana-plugin list
```

汉化 kibana.yml文件中，增加 i18n.locale: "zh-CN"，就支持中文显示。

####基本

##### 概念

对比关系型数据库理解 es

| Mysql    | ES       |
| -------- | -------- |
| Database | Index    |
| Table    | Type     |
| Row      | Document |
| Column   | Field    |

##### 命令

```
#查看索引相关信息
GET kibana_sample_data_ecommerce

#查看索引的文档总数
GET kibana_sample_data_ecommerce/_count

#查看前10条文档，了解文档格式
POST kibana_sample_data_ecommerce/_search
{
}

#_cat indices API
#查看indices
GET /_cat/indices/kibana*?v&s=index

#查看状态为绿的索引
GET /_cat/indices?v&health=green

#按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

#查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

#How much memory is used per index?
GET /_cat/indices?v&h=i,tm&s=tm:desc
```



