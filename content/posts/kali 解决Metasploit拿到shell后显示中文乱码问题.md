+++
title = "kali 解决Metasploit拿到shell后显示中文乱码问题"
author = "Hzllaga"
date =  2018-07-23
description = ""
categories = ["转载"]
tags = ["Metasploit"]
+++

拿到对方shell后显示的问题如下:
![](https://cdn.wtfsec.org/img/20200223170458.png)

中文乱码解决:
{{< highlight shell >}}
chcp 65001
{{< /highlight >}}

![](https://cdn.wtfsec.org/img/20200223170511.png)
然后
![](https://cdn.wtfsec.org/img/20200223170524.png)
上传下载文件
![](https://cdn.wtfsec.org/img/20200223170531.png)

