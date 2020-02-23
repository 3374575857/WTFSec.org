+++
title = "Msf 同端口接受多个session"
author = "Hzllaga"
date =  2018-07-23
description = ""
categories = ["原创"]
tags = ["Metasploit"]
+++
![](https://cdn.wtfsec.org/img/20200223170158.jpg)

{{< highlight php >}}
exploit -j
{{< /highlight >}}

#后台监听
{{< highlight php >}}
set exitonsession false
{{< /highlight >}}
可以让建立监听的端口继续保持侦听。可以接受多个session