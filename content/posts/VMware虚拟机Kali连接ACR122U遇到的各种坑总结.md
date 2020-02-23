+++
title = "VMware虚拟机Kali连接ACR122U遇到的各种坑总结"
author = "Hzllaga"
date =  2019-04-08 
description = ""
categories = ["原创"]
tags = ["Linux","Kali","IC"]
+++
Kali安装过程就省略不说了，直接讲怎么连接，这部分网上还是比较少的，都是PN532比较多
驱动直接去官网找acr122u-nfc读写器-usb接口的[PC/SC 驱动包装](https://www.acs.com.hk/download-driver-unified/10476/ACS-Unified-PKG-Lnx-116-P.zip)

然后直接用apt安装一下libtool
{{< highlight shell >}}
sudo apt install libtool
{{< /highlight >}}

接着连接后你会发现执行nfc-list卡在那边没反应

![](https://cdn.wtfsec.org/img/20200223175740.png)

接着设备移除后变这样

![](https://cdn.wtfsec.org/img/20200223175750.png)

原因是内核版本(>3.5)会自动加载pn533驱动导致报错

解决方法很简单，首先执行
{{< highlight shell >}}
lsmod
{{< /highlight >}}
看看哪些驱动被安装了

常见的就这几种 pn533_usb,pn533,pn533_usb,nfc

再来就把这几个模块给移除掉，会导致读卡机灯熄灭，不过不影响操作

{{< highlight shell >}}	
sudo modprobe -r pn533_usb pn533 nfc
{{< /highlight >}}
然后把这几个加入黑名单
{{< highlight shell >}}
sudo vi /etc/modprobe.d/blacklist-libnfc.conf
{{< /highlight >}}
 加上
{{< highlight shell >}}	
blacklist nfc
blacklist pn533
blacklist pn533_usb
{{< /highlight >}}

然后重新执行nfc-list，就会发现正常读取了，并且读卡机会亮红灯
<hr>
如果是提示No NFC device found这种情况的话

是因为VMware在新版是使用共享ccid的方式，导致libnfc无法读取，不过pcsc_scan是可以正常读取的

![](https://cdn.wtfsec.org/img/20200223175921.jpg)

这种方式VMware官方给出的方式还是比较简单粗暴的

关闭虚拟机后直接修改这个虚拟机的vmx文件，添加上这两句
{{< highlight shell >}}	
usb.generic.allowCCID = "TRUE"
usb.ccid.disable = "TRUE"
{{< /highlight >}}
然后开机就可以直接用USB直通模式把读卡机连接进去了

![](https://cdn.wtfsec.org/img/20200223175952.jpg)

接下来会遇到的问题就在上面了，折腾我2天就TM只是驱动问题= =