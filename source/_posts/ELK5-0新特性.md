---
title: ELK5.0新特性
date: 2016-11-07 11:25:08
tags: ELK
---

[Elastic Stack 5.0.0 Released](https://www.elastic.co/blog/elastic-stack-5-0-0-released)

[elasticsearch Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/breaking-changes-5.0.html) 

[升级]sticsearch(https://www.elastic.co/guide/en/elasticsearch/reference/5.0/setup-upgrade.html)

[logstash Breaking changes](https://www.elastic.co/guide/en/logstash/5.0/breaking-changes.html)

[kibana概览](https://www.elastic.co/products/kibana) 

[kibana Breaking changes](https://www.elastic.co/guide/en/kibana/5.0/breaking-changes-5.0.html)

[x-pack安装](https://www.elastic.co/downloads/x-pack)

[x-packe文档](https://www.elastic.co/guide/en/x-pack/5.0/index.html)

[metricbeat概览](https://www.elastic.co/guide/en/beats/metricbeat/5.0/metricbeat-overview.html)

[The Beats Family](https://www.elastic.co/products/beats)

安装体验
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/5.x stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
sudo apt-get update && sudo apt-get install elasticsearch lohstash kibana
```
