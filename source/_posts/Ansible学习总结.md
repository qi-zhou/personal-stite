---
title: Ansible学习总结
date: 2016-11-15 18:14:43
tags: DevOps
---
..........
<!--more-->
## 前言
在学习批量管理软件时，首先要明确的知道自己需要什么，网上大神很多，他们都研究到源码上了，写了很多介绍绚丽功能的文档，但其实那些功能基本上我们都用不到，经常被各种文档弄得头脑发晕，此文就是为了简单直白的告诉大家ansible的功能，满足大家的基本需求。

首先确认批量管理我们需要什么：无外乎主机分组管理、实时批量执行命令或脚本、实时批量分发文件或目录、定时同步文件等。

## ansible与saltstack对比

前一段时间用了saltstack，免不得要谈一下他们的优缺点。两者都是安装和使用都非常方便的批量管理软件。

- salt要安装agent；ansible不需要，通过ssh连接，省掉装agent的事。
- salt在server端要启进程；ansible不需要，但这都无所谓差不多。
- salt与ansible都有模块，可使用任意语言开发模块。
- salt与ansible都使用yaml语言格式编写剧本。

ansible由于走的是ssh,所以它有认证的过程，以及加密码的过程，这使得ansible非常慢，不适用于大规模环境（指上千台）。

为什么我放弃salt呢，首先服务器不多（百台左右），其次，salt的master端与minion端TCP连接经常断开，导致有时执行命令时会漏机器，这简直让我忍无可忍。听说最新版的salt好了很多，但由于公司系统是定制的，安装软件特别麻烦（15M的系统，解决依赖就是个大问题），我还是选择了ansible。

## ansible安装
```
yum installparamiko PyYAML jinja2 httplib2  #ansible所需依赖包
git clonehttps://github.com/ansible/ansible.git
cd ansible
python setup.pyinstall
cp ansible/examples/ansible.cfg/etc/ansible/   #拷贝ansible默认的配置文件，也可不拷贝。
```
安装完成

在开始ansible操作受控机器前，需要配置好ssh免密码登陆

## ansible命令参数介绍
Ansible中的临时命令的执行是通过Ad-Hoc来完成，能够快速执行，而且不需要保存执行的命令，例如：
```
ansible-i ～/hosts all -m command -a ‘who’ -u root
```
主要参数如下：
```
-u username          #指定ssh连接的用户名，即执行后面命令的用户
-i inventory_file    #指定所使用的inventory文件的位置，默认为/etc/ansible/hosts
-m module            #指定使用的模块，默认为command
-f 10                #指定并发数,并发量大的时候，提高该值
--sudo [-k]          #当需要root权限执行的化，-k参数用来输入root密码
```

## ansible主机分组管理：
配置好ssh免密码登陆后，就该把那些机器加入到hosts文件中，hosts不限路径，可用-i参数指定路径。
```
[root@yang~]# cat /etc/ansible/hosts 
[KD1]    #分组名
1.1.1.1:62222    #此处对于端口进行指定，毕竟有些服务器ssh端口打开的不一样。
1.1.1.2:62222
[KD2]
1.1.1.3:62222
1.1.1.4:62222
```
分组执行效果：
```
[root@yang~]# ansible -i /etc/ansible/hosts KD1 -m shell -a ‘uptime‘
1.1.1.1| success | rc=0 >>
 11:56:31 up 2 days, 17:42, load average: 0.41,0.34, 0.32
 
1.1.1.2| success | rc=0 >>
 11:57:03 up 2 days, 17:44, load average: 0.34,0.28, 0.25
```
连续的主机名使用";"号分隔，如:`ansible -i /etc/ansible/hosts‘KD1;1.1.1.3‘ -m shell -a ‘uptime‘`

还可以使用脚本动态获取主机的方式：

官方地址：http://docs.ansible.com/intro_dynamic_inventory.html

网络文档：http://noops.me/?p=1446

## ansible实时批量执行命令和脚本：
批量执行命令：
```
ansible-i /etc/ansible/hosts all -m shell -a ‘uptime‘
ansible-i /etc/ansible/hosts all -m command -a ‘uptime‘
#shell模块与command模块细节区别我也没分清在哪儿。
```
批量执行脚本：
```
ansibleall -m script -a "/root/123.sh"  #
#此命令是在远程服务器上执行本地的脚本
```
网上文档说，复杂的命令用playbook管理，我实在不认同，作为运维人员，我写一个脚本多简单，干嘛去花太多时间研究playbook的格式与用法呢？

再说，ansible对于一些安装包的管理，我可以事先做好rpm包，然后使用copy模块分发过去就是，为什么去研究太多的用法？

## ansible实时批量拷贝文件或目录
从ansible-doc copy中的帮助信息得知，ansible的copy模块是围绕rsync的包装，所以它是增量而不是全量的拷贝。
```
ansibleMachineName -m copy -a ‘src=/etc/fstab  dest=/tmp/fstabmode=644 owner=root‘   #拷贝文件
ansibleMachineName -m copy -a ‘src=/etc/test dest=/root/test mode=755 owner=root ‘  #此处test为目录
```

## ansible定时同步文件
既然ansible的copy模块是rsync的包装，那我定期执行copy目录的命令，就能完成文件的定时同步了。如果文件变化，就会同步过去，如果没变化，就不会拷贝。以后只要把要同步的文件放到该目录下即可。

## ansible模块帮助
执行命令用到的那些模块是干嘛的？使用ansible-doc查看帮助吧。
```
ansible-doc-l       #查看模块列表
ansible-doc  copy    #查看copy模块的详细信息
ansible-doc  script #查看script模块的详细信息
```
常用的模块有ping,copy,shell,command,script等。更多模块的使用请自行探索吧，我比较懒，满足我的需求后就不想动脑了。

## ansibleplaybook的使用
其实playbook就是把上述在命令行的操作，以yml格式写在文件中来执行而已。复杂的playbook只是更多的命令行操作的集合。

此例：当某个文件变化后，移走该文件。
```
#cat  playbook.yml
---
- hosts:local  # hosts中指定
  remote_user: yang  # 如果和当前用户一样，则无需指定
  tasks:
      - name: whoami
        copy: src=~/hosts dest=~/hosts.dest  #  本地拷贝到远端
        notify: # 如果copy执行完之后~/hosts.dest文件发送了变化，则执行
            - clear copy  # 调用handler中的clear copy定义的动作
  handlers:
      - name: clear copy
        shell: ‘mv ~/hosts.dest hosts.del‘  # 假装删除
```
注解：
```
tasks定义了playbook中要执行的任务，包括任务名name以及具体的任务内容
notify：类似于Salt的require，表示当前面的任务完成后且有相应的变化时调用后面定义的handler
handlers：与notify结合使用，被调用的handler的具体定义
```
playbook执行方法：
```
ansible-playbookplaybook.yml    
#主机名、执行命令都已在yml中指定了。
```
## ansible网上参考地址：

ansible快速上手：https://linuxtoy.org/archives/hands-on-with-ansible.html

Ansible:自动化工具：http://rangochen.blog.51cto.com/2445286/1425276

自动化工具ansible中文指南：http://www.aikaiyuan.com/6299.html

运维自动化之ansibleplaybook安装nginx：http://dl528888.blog.51cto.com/2382721/1438847

Ansible之安装部署及常用模块的使用介绍：http://yanshisan.blog.51cto.com/7879234/1384405

Ansible状态管理：http://xdays.info/ansible%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86.html

ansible学习笔记（二）初始化操作系统：ansible变量使用：http://laowafang.blog.51cto.com/251518/141847
