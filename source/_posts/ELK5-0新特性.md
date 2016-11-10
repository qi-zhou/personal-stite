---
title: ELK5.0新特性
date: 2016-11-07 11:25:08
tags: ELK
---
............
<!--more-->
同时由于现在的版本比较混乱，每个产品的版本号都不一样，Elasticsearch和Logstash目前是2.3.4；Kibana是4.5.3；Beats是1.2.3；

------------------------
## Elasticsearch
#### 性能提升
- Elasticsearch5.0集成了Lucene6版本，其中最重要的特性就是 Dimensional Point Fields，多维浮点字段，ES里面相关的字段如date, numeric，ip 和 Geospatial 都将大大提升性能。
- 磁盘空间少一半,索引时间少一半,查询性能提升25%,IPV6也支持了。

#### 新增项
- 新增了一个Profile API

玩过SQL的人都知道，数据库服务的执行计划（execution plan）非常有用，可以看到那些查询走没走索引和执行时间，用来调优，elasticsearch现在提供了Profile API来进行查询的优化，只需要在查询的时候开启profile：true就可以了，一个查询执行过程中的每个组件的性能消耗都能收集到。

- 新增了一个Shrink API
- 新增了一个Rollover API
- 新增：Ingest Node
- 新增Painless Scripting
- 新增: half_float 类型
- 引入新的字段类型Text/Keyword 来替换 String

#### 其他
- 使用UUID来作为索引的物理的路径名，有很多好处，避免命名的冲突
- 默认使用BM25评分算法，效果更佳，之前是TF/IDF
- 移除 site plugins，就是说head、bigdesk都不能直接装es里面了，不过可以部署独立站点
- 支持分号（；）来分割url参数，与符号（&）一样 

-----------
## Logstash

--------------
## Kibana
#### 新增Timelion

以前以re{Search} 项目介绍过，现在Timelion 作为Kibana原生的核心组件可直接可用。Timelion 提供一个查询表达式和可视化类型让你探索基于时间的数据。

------------
#### Metricbeat
在Elastic 我们热爱扩展。太多我们构建的东西我们给他们起了非常有趣的名字，如：Shield、Marvel和Watcher，作为提供给我们客户的额外的插件，独立闭源但没限制开源部分的能力的特性，随着后面又增加了Graph 和Reporting，安装流程也变得困难和困惑。

Metricbeat 替换 Topbeat 成为Elastic Stack里主要的收集度量指标的工具。和Topbeat一样，Metricbeat 收集和“top” 类似的诸如机器及进程的资源(CPU, memory, disk, network)统计信息。和Topbeat不同的是，Metricbeat 同时也收集其它系统的指标信息，如：Apache、HAProxy、MongoDB、MySQL、Nginx、PostgreSQL、 Redis和 Zookeeper，并且在不久的将来还会支持更多应用和系统。

----------------
## x-pack
一个包含了security、alerting、monitoring & management、reporting和graph 能力的Elastic Stack的插件。我们对5.0的工程不仅限于Elastic Stack，同时也包括给X-Pack 添加如下：
- Kibana里的管理和监控的UI界面
- Kibana里创建用户和角色的UI界面
- 非常简化的安装流程

X-Pack 可以试用，同时提供商业和免费版。详细请见 [Subscriptions](https://www.elastic.co/subscriptions ) 页。

-------------------------
Elastic Stack安装体验
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/5.x stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
sudo apt-get update && sudo apt-get install elasticsearch lohstash kibana
```

-------------------
相关官方文档:
[Elastic Stack文档](https://www.elastic.co/guide/index.html)
[Elastic Stack 5.0.0 Released](https://www.elastic.co/blog/elastic-stack-5-0-0-released)
[elasticsearch Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/breaking-changes-5.0.html) 
[elasticsearch升级](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/setup-upgrade.html)
[kibana概览](https://www.elastic.co/products/kibana) 
[logstash Breaking changes](https://www.elastic.co/guide/en/logstash/5.0/breaking-changes.html)
[x-pack安装](https://www.elastic.co/downloads/x-pack)
[x-packe文档](https://www.elastic.co/guide/en/x-pack/5.0/index.html)
[metricbeat概览](https://www.elastic.co/guide/en/beats/metricbeat/5.0/metricbeat-overview.html)
[The Beats Family](https://www.elastic.co/products/beats)

参考文章:
[1]. http://www.infoq.com/cn/news/2016/08/Elasticsearch-5-0-Elastic
[2]. https://www.iteblog.com/archives/1865
[3]. http://elasticsearch.cn/article/106
[4]. http://kibana.logstash.es/content/logstash/plugins/output/elasticsearch.html
