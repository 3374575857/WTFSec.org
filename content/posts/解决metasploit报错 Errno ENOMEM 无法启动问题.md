+++
title = "解决metasploit报错Errno::ENOMEM无法启动问题"
author = "Hzllaga"
date =   2018-07-20 
description = "相信很多人在vps上安装metasploit常会出现这个问题，网上也没有明确的解答，我就来写一个吧。"
categories = ["原创"]
tags = ["Metasploit"]
+++
![](https://cdn.wtfsec.org/img/20200223175104.png)
相信很多人在vps上安装metasploit常会出现这个问题，网上也没有明确的解答，我就来写一个吧。

首先他提示的是内存不足，可以先查看内存：
{{< highlight shell >}}
free -m
{{< /highlight >}}
![](https://cdn.wtfsec.org/img/20200223175208.png)

可以看到内存是足够的，但是Swap却是0，那么问题就知道了。

下面是解决方法：
{{< highlight shell >}}
[root@wtfsec ~]# mkdir /swapfile
[root@wtfsec ~]# cd /swapfile
[root@wtfsec swapfile]# sudo dd if=/dev/zero of=swap bs=1024 count=2000000
2000000+0 records in
2000000+0 records out
2048000000 bytes (2.0 GB) copied, 10.6139 s, 193 MB/s
[root@wtfsec swapfile]# sudo mkswap -f  swap
Setting up swapspace version 1, size = 1999996 KiB
no label, UUID=d042b5bf-0cec-4248-b28c-6aadd6bc434e
[root@wtfsec swapfile]# sudo swapon swap
swapon: /swapfile/swap: insecure permissions 0644, 0600 suggested.
{{< /highlight >}}
然后再次运行：
{{< highlight shell >}}
free -m
{{< /highlight >}}
![](https://cdn.wtfsec.org/img/20200223175323.png)

可以看到，已经增加了。
最后尝试开启metasploit
![](https://cdn.wtfsec.org/img/20200223175331.png)

好了，正常运行了。