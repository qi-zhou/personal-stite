---
title: 30分学会SaltStack
date: 2016-10-15 21:06:57
tags: DevOps
---
用salt集中配置管理服务器……
<!--more-->
##  SaltStack简介
salt是一个异构平台基础设置管理工具(虽然我们通常只用在Linux上)，使用轻量级的通讯器ZMQ,用Python写成的批量管理工具，完全开源，遵守Apache2协议，与Puppet，Chef功能类似，有一个强大的远程执行命令引擎，也有一个强大的配置管理系统，通常叫做Salt State System。

SaltStack除了传统的C/S架构外,其实还有Masterless架构,如果采用Masterless架构,我们就不需要单独安装一台SaltStack Master机器,只需要在每台机器上安装Minion,然后采用本机只负责对本机的配置管理工作机制服务模式。

## 基本原理
SaltStack 采用 C/S模式，server端就是salt的master，client端就是minion，minion与master之间通过ZeroMQ消息队列通信.

minion上线后先与master端联系，把自己的pub key发过去，这时master端通过salt-key -L命令就会看到minion的key，接受该minion-key后，也就是master与minion已经互信.

master可以发送任何指令让minion执行了，salt有很多可执行模块，比如说cmd模块.

这些模块是python写成的文件，里面会有好多函数，如cmd.run，当我们执行`salt '*' cmd.run 'uptime'`的时候，master下发任务匹配到的minion上去，minion执行模块函数，并返回结果。master监听4505和4506端口，4505对应的是ZMQ的PUB system，用来发送消息，4506对应的是REP system是来接受消息的。

### 具体步骤如下
1. Salt stack的Master与Minion之间通过ZeroMq进行消息传递，使用了ZeroMq的发布-订阅模式，连接方式包括tcp，ipc
1. salt命令，将cmd.run ls命令从`salt.client.LocalClient.cmd_cli`发布到master，获取一个Jodid，根据jobid获取命令执行结果。
1. master接收到命令后，将要执行的命令发送给客户端minion。
1. minion从消息总线上接收到要处理的命令，交给`minion._handle_aes`处理
1. `minion._handle_aes`发起一个本地线程调用cmdmod执行ls命令。线程执行完ls后，调用`minion._return_pub`方法，将执行结果通过消息总线返回给master
1. master接收到客户端返回的结果，调用`master._handle_aes`方法，将结果写的文件中
1. `salt.client.LocalClient.cmd_cli`通过轮询获取Job执行结果，将结果输出到终端。

### 启动服务
```
service salt-master start
service salt-minion start
```

## 测试
```
salt '*' test.ping   ##查看在线minion
```

### salt-key 密钥管理，通常在master端执行
```
salt-key [options]
salt-key -L              ##查看所有minion-key
salt-key -a <key-name>   ##接受某个minion-key
salt-key -d <key-name>   ##删除某个minion-key
salt-key -A              ##接受所有的minion-key
salt-key -D              ##删除所有的minion-key
```

### salt-call 该命令通常在minion上执行，minion自己执行可执行模块，不是通过master下发job,一般在使用时加上`--local`参数，这种方式相当于`Masterless`模式
```
salt-call [options] <function> [arguments]
salt-call test.ping           ##自己执行test.ping命令
salt-call cmd.run 'ifconfig'  ##自己执行cmd.run函数
```

### salt-cp 分发文件到minion上,不支持目录分发，通常在master运行
```
salt-cp [options] '<target>' SOURCE DEST
salt-cp '*' testfile.html /tmp
salt-cp 'test*' index.html /tmp/a.html
```

### salt-ssh 有点类似ansible,好处就是你不需要在客户端安装minion，也不需要安装master 
从0.17.0开始salt加入salt ssh，salt ssh不需要在客户端安装salt-minion包了，是通过ssh协议来完成运城命令执行，状态管理等任务的。它是作为master-minion形式的补充出现的，原理是有一个花名册的文件，里面记录了各个minion的信息，ip,账号，密码，等，需要远程执行命令时，直接通过ssh来执行，速度与master-minion形式慢很多。

使用： 配置/etc/salt/roster格式如下
```
test1:
  host: 192.168.1.133
  user: salt
  passwd: redhat
  sudo: True
  port: 22
  timeout: 5

test2:
  host: 192.168.1.134
  user: root
  passwd: redhat
test3:
  host: 192.168.1.135
  user: sa
  sudo: True
```

说明: test1我们定义了所有常见的选项，test2我们用了超级用户，使用账号密码，test3我们使用普通用户，没有写密码，就是通过key来认证了，并且以sudo方式执行的，需要注意的是1.key认证用的是/etc/salt/pki/master/ssh/目录下的密钥。2.如果普通用户的话，需要有sudo权限，因为一些功能，包括test.ping都需要sudo权限。

测试`salt-ssh`:
```
 salt-ssh '*' test.ping
 salt-ssh '*' -r 'ls /'   ##执行shell命令
 salt-ssh '*' cmd.run 'ls /' ##用模块来执行也是可以的
 salt-ssh '*' state.apply   ##执行状态
```

#### Target参数表
![](http://blog.cunss.com/wp-content/uploads/2014/02/20140210173734_68950.png)

## 帮助命令
- 查看Minion支持的所有module列表的命令如下:
```
salt 'Minion' sys.list_modules
```
- 查看cmd module的所有function的命令如下:
```
salt 'Minion' sys.list_functions cmd
```
- 查看cmd module的详细用法与例子的命令如下:
```
salt 'Minion' sys.doc cmd
```
- 要查看Minion支持的所有states列表,命令如下所示:
```
salt 'Minion' sys.list_state_modules
```
- 查看file.states的所有function,命令如下所示:
```
salt 'Minion' sys.list_state_function
```
- 要查看file.states的详细用法与例子,命令如下所示:
```
salt 'Minion' sys.state_doc file
```
- 要查看file.managed states的详细用法与例子,命令如下所示:
```
salt 'Minion' sys.state_doc file.manage
```

## 关于`saltstack grains`
下面我们先介绍下比较简单的Grains自定义方法,就是通过Minion配置文件定义。前面已经讲到Minions的Grains信息是在Minions服务启动的时候汇报给Matser的,所以我们需要修改好Minion配置文件后重启Minion服务。在Minion的/etc/salt/minion配置文件中默认有一些注释行。这里就是在Minion上的minion配置文件中如何定义Grains信息例子。下面只需根据自动的需求按照以下格式去填写相应的键值对就行,大家注意格式就行,SaltStack的配置文件的默认格式都是YAML格式:
```
grains:
  roles:
    - webserver
    - memcache
```

Master上查看定义的Grains信息是否生效:
```
salt 'Minion' grains.item roles

Minion:
----------
roles:
- webserver
- memcache
```

下面我们通过Grains模块来设置并定义Grains信息:
```
salt 'Minion' grains.append roles test
```

更多用法查看：
```
salt'Minion'sys.doc grains
```

## 关于`saltstack pillar`
Pillar在salt中是非常重要的组成部分，利用它可以完成很强大的功能，它可以指定一些信息到指定的minion上，不像grains一样是分发到所有Minion上的，它保存的数据可以是动态的,Pillar以sls来写的，格式是键值对

### 适用情景：

1. 比较敏感的数据，比如密码，key等

2. 特殊数据到特定Minion上

3. 动态的内容

4. 其他数据类型

查看Minion的Pillar信息
```
salt '*' pillar.items
```

查看某个Pillar值
```
salt '*' pillar.item <key>      #只能看到顶级的
salt '*' pillar.get <key>:<key> #可以取到更小粒度的
```

编写pillar数据

1. 指定pillar_roots，默认是/srv/pillar(可通过修改master配置文件修改),建立目录
```
mkdir /srv/pillar
cd /srv/pillar
```

2. 编辑一个pillar数据文件
```
vim test1.sls
    name: 'salt'
    users:   
        hadoop: 1000
        redhat: 2000
        ubuntu: 2001
```

3. 建立top file指定minion到pillar数据文件
```
 vim top.sls
 base:  
      '*':    
          - test1
```

4. 刷新Pillar数据
```
salt '*' saltutil.refresh_pillar
```

5. 测试
```
salt '*' pillar.get name
salt '*' pillar.item name
```

在state中通过jinja使用pillar数据

```
vim /srv/salt/user.sls
{% for user, uid in pillar.get('users', {}).items() %}  ##pillar.get('users',{})可用pillar['users']代替，前者在没有得到值的情况下，赋默认值

{{ user }}:   
   user.present:     
        - uid: {{ uid }}
{% endfor %}
```

当然也可以不使用jinja模板
```
vim /srv/salt/user2.sls
{{ pillar.get('name','') }}:
     user.present:   
        - uid: 2002
```

通过jinja模板配合grains指定pillar数据
```
/srv/pillar/pkg.sls

pkgs:
 {% if grains['os_family'] == 'RedHat' %}
 apache: httpd
 vim: vim-enhanced
 {% elif grains['os_family'] == 'Debian' %}
 apache: apache2
 vim: vim
 {% elif grains['os'] == 'Arch' %}
 apache: apache
 vim: vim
 {% endif %}
```

通过以下命令查看关于Pillar相关的一些模块用法:
```
salt 'Minion' sys.list_functions pillar
```

详细用法与例子可以通过命令
```
salt 'Minion' sys.doc pillar
```

## 关于`saltstack mine`
Mine是SaltStack收集Minion数据存储到Master的一个组件,它的功能与Grains有些类似,Mine可以指定任何Minion模块去采集数据。但是Master只能存储Minion收集上来的最近一段的数据,Mine的主要应用场景是配合前端负载均衡动态获取Mine汇报信息,来动态生成配置文件。例如官网通过mine.get指定业务设备的网卡地址动态生成haprxoy.cfg文件。Mine还支持get docker容器的地址,可简单实现动态添加业务。Mine配置目前支持两种方式,第一种是通过在Minion配置文件中定义,另一种是通过模块的方式去下发Mine采集任务。关于Mine模块的更多用法可以使用sys.doc mine命令查看。下面我们通过模块的方式下发一个采集docker0网卡地址的任务:
```
salt 'Minion' mine.send network.ip_addrs
salt 'Minion' mine.get 'Minion' network.ip_addrs
```

## 关于渲染器render system
我们上面也提过salt默认的渲染器是yaml_jinja,salt处理我们的sls文件时，会先把文件用jinja2处理，然后传给ymal处理器在处理，然后生成的是salt需要的python数据类型。除了yaml_jinja还有yaml_mako,yaml_wempy,py,pydsl。下面来看个样例吧，

apache/init.sls文件内容
```
apache:
     pkg: installed:   
       {% if grains['os'] == 'RedHat' %}
       - name: httpd
       {% endif %}
     service.running:   
       {% if grains['os'] == 'Redhat' %}
       - name: httpd
       {% endif %}
       - watch:
         - pkg: apache
```

这个样例很简单，就是加了个判断，如果Minion的grains的os是RedHat那么apache的包名是httpd，默认是apache,我们知道在别的Linux发行版上，如ubuntu，suse他们的apache的包名就是叫apache，而在redhat系上则叫httpd,所以才有了这个判断写法，下面的service也是如此。我们着重说语法，jinja中判断，循环等标签是放在{% raw %}{% %}{% endraw %}中的，通常也会有结束标签{% raw %}{% end** %}{% endraw %},而变量是放在 {% raw %}{{ }}{% endraw %}中的，salt,grains,pilla是salt中jinja里面的三个特殊字典，salt是包含所有salt函数对象的字典，grains是包含minion上grains的字典，pillar是包含minion上pillar的字典。

示例：for user/init.sls文件内容
```
{% set users = ['jerry','tom','gaga'] %}
{% for user in users %}
{{ user }}:
     user.present:  
           - shell: /bin/bash
           - home: /home/{{ user }}
{% endfor %}
```

示例；salt字典 user/init.sls文件内容
```
{% if salt['cmd.run']('uname -i') == 'x86_64' %}
hadoop:
    user.present:   
        - shell: /bin/bash        - home: /home/hadoop
{% elif salt['cmd.run']('uname -i') == 'i386' %}
openstack:
    user.present:   
        - shell: /bin/bash
        - home: /home/openstack
{% else %}
django:
     user.present:   
        - shell: /sbin/nologin
{% endif %}
```

## 依赖关系系统requisite system
前面我们已经用过了依赖关系系统，就是定义状态与状态之间的依赖关系的，经常遇到的依赖系统的函数有'require'和'watch'和它们的变种'require_in','watch_in',require和watch有什么区别吗？

1.不是所有的state都支持watch,比较常用的是service

2.watch定义的依赖条件发生变化时会执行一些动作，如当配置文件改变时，service会重启

示例: apache/init.sls文件内容
```
/etc/httpd/httpd.conf:
 file:   
    - managed
    - source: salt://httpd/httpd.conf

httpd:
 pkg:   
    - installed
 service:   
   - running
   - require:     
     - pkg: httpd
   - watch:     
     - file: /etc/httpd/httpd.conf            ##当httpd.conf改变时，重启httpd服务
```
require与require_in, watch与watch_in

require,watch是指依赖，require_in,watch_in是指被依赖

a reuire b 那么就是b require_in a

a watch b 那么就是b watch_in a

示例: apache/init.sls文件内容
```
/etc/httpd/httpd.conf:
 file:   
    - managed
    - source: salt://httpd/httpd.conf
    - watch_in:
        - service: httpd
 httpd:
 pkg:   
    - installed
    - require_in:
    - service: httpd
 service:   
    - running
```

## YAML
什么是YAML？

- 维基上第一句话说的是“是一个可读性高，用来表达资料序列的格式。” ,YAML是”YAML Ain't a Markup Language”（YAML不是一种标记语言）的递回缩写。

编写YAML

初期对于新手来说，这个最要命。不注意以下操作就要出错，本人以前是经常犯错。


- 缩进要用2个空格，不能用tab，更不能tab和空格混用。以前写Python tab顺手了，就常犯错。
- YAML是用空格来表示分层结构的，如第一层，（前面2个空格） 第二层，(前面4个空格)
- YAML的冒号是用来表示一个K/V的字典的。

例： `file: roddyname` 就表示 `{'file':'roddyname'}`

还可以写成
```
file:
  roddyname
```

- YAML使用短横线表示列表。

例：
```
– user1
– user2
– user3
```
对应的表示就是['user1′,'user2′,'user3']

如果看到这种表达方式
```
username:
  – user1
  – user2
  – user3
```
对应的表示就是`{'username':['user1′,'user2′,'user3']}`

我靠，好清晰，对了撒。这下子搞懂了，唉，以前不晓得杂个写。呵呵

## jinja
- 什么是jinja？

尼玛，我就觉得这个名字有点日怪，以前读的时候扭捏了半天。这又是个什么鬼？
jinja是一种模板引擎，是基于Python的，相当于php 的 smarty。哦，基本上了解了。可以基于这个引擎提前做一些操作嘛

salt默认是用的`yaml_jinja`的渲染器。

工作流程：

1. 通过jinjia2模板引擎处理SLS
2. 调用YAML解析器

如何使用:

在salt中 ,files和templates都使用了file这个states模块。但是是通过template这个标识来指明是模板还是普通文件。
OK。看个他的样例

jinja的使用方法：
1. 在file模块sls文件中使用`template ：- template: jinja`
2. 在模板文件里面使用变量{{变量名}}
3. 指定变量列表：

```
  defaults: server01
  SERVER: "192.168.1.2"
  PORT: "22"
```

```
使用grains： {{grains['hostname']}}

使用模块: {{ salt['network.hw_addr']('eth0') }}

使用pillar: {{ pillar['apache']['port'] }} 两边有空格
```

## states入门
首先介绍使用states的流程:
- 编写top.sls文件(非必须)。
- 编写states.sls文件。
```
SaltStack@Master: cat /srv/salt/one/test.conf
SaltStack@Master: cat /srv/salt/one/init.sls
/tmp/foo.conf:  #id
  file.managed:  #file states 的 managed function
    - source: salt://one/foo.conf  # 文件来源
    - user: root    # 文件属主
    - group: root   # 文件属组
    - mode: 644     # 文件权限
    - backup: minion  # 备份原文件
```

```
salt 'Minion' state.apply one
```

下面我们来介绍使用top.sls入口文件同时对多台机器进行一个简单的配置管理。首先在states的工作目录下新建top.sls文件:
```
SaltStack@Master: cat /src/salt/top.sls

base:  #base 环境
  '*':  #Target( 代表所有 Traget)
    - one  # 引用 one.sls 或者 one/init.sls states 文件
  'Minion':  #Target( 代表匹配 Minion)
    - two  # 引用 two.sls 或者 tow/init.sls states 文件
  'Minion1':  #Target( 代表批量 Minion1)
    - three  # 引用 three.sls 或者 three/init.sls states
```

最后我们就可以使用以下命令同时对Minion和Minion1两台机器进行配置管理了。
```
salt '*' state.highstate test=True  #空转，在正式应用配置前用来测试
salt '*' state.highstate
```

## [最佳实战](https://docs.saltstack.com/en/latest/topics/best_practices.html)
## [Salt Stack Formulas](https://github.com/saltstack-formulas)
### 参考文献
1. [SaltStack第一章](http://mp.weixin.qq.com/s?__biz=MzAwNzY4OTgyNA==&mid=401293072&idx=1&sn=65dbad3a8577bd595d8af35c824194e9&scene=0#wechat_redirect)
1. [SaltStack第二章](http://mp.weixin.qq.com/s?__biz=MzAwNzY4OTgyNA==&mid=401319484&idx=1&sn=d82d8e967d5f9338cfbad73e47c060cb&scene=0#wechat_redirect)
1. [SaltStack第三章](http://mp.weixin.qq.com/s?__biz=MzAwNzY4OTgyNA==&mid=401871765&idx=1&sn=face6fb37b14207ecd89e159a527ae43&scene=0#wechat_redirect)
1. [salt-ssh实现原理](http://ju.outofmemory.cn/entry/99803)
