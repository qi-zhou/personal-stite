---
title: Wazuh HIDS系统使用
date: 2016-10-21 11:06:06
tags: 安全
---
.......
<!--more-->
## Wazuh HIDS manager安装
```
sudo apt-get install mailutils
sudo apt-get install gcc make git

If you want to use Auth, also install:
sudo apt-get install libssl-dev
```

clone our Github repository and compile the source code, to install OSSEC:
```
cd ~
mkdir ossec_tmp && cd ossec_tmp
git clone -b stable -depth 1 https://github.com/wazuh/ossec-wazuh.git
cd ossec-wazuh
sudo ./install.sh
```
[gent ID reusage](http://wazuh-documentation.readthedocs.io/en/latest/manual_reuse_id.html)

start your OSSEC manager running:
```
sudo /var/ossec/bin/ossec-control start
```

Here are some useful commands to check that everything is working as expected
```
ps aux | grep ossec
lsof /var/ossec/logs/alerts/alerts.json
cat /var/ossec/logs/alerts/alerts.json
```

install agent
```
echo -e "deb http://ossec.wazuh.com/repos/apt/ubuntu trusty main" >> /etc/apt/sources.list.d/ossec.list
apt-get update
apt-get install ossec-hids-agent
```

Add a new agent
On your OSSEC manager, run /var/ossec/bin/manage_agents:
```
/var/ossec/bin/manage_agents
```

Agent configuration on Linux
/var/ossec/etc/ossec.conf,and set the server-ip to the right value:
```
<ossec_config>
  <client>
    <server-ip>XXX.XXX.XXX.XXX</server-ip>
  </client>
```
/var/ossec/bin/manage_agents
/var/ossec/bin/ossec-control restart

integration with ELK
```
sudo cp ~/ossec_tmp/ossec-wazuh/extensions/logstash/01-ossec.conf  /etc/logstash/conf.d/
```

And now download and install GeoLiteCity from the Maxmind website. This will add geolocation support for public IP addresses:
```
sudo curl -O "http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz"
sudo gzip -d GeoLiteCity.dat.gz && sudo mv GeoLiteCity.dat /etc/logstash/
```

[Kibana Dashboard](http://wazuh-documentation.readthedocs.io/en/latest/ossec_elk_kibana.html)

sudo usermod -a -G ossec logstash
sudo apt-get install sendmail
