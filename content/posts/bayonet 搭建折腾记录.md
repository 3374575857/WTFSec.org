+++
title = "Bayonet 搭建折腾记录"
author = "此生等不到表哥一句我带你"
date =  2020-02-24T02:47:24+08:00
description = "记录搭建了一个bayonet 记录下踩过的各种坑"
categories = ["原创"]
tags = ["投稿","笔记"]
+++

搭建了一个bayonet 记录下踩过的各种坑
> bayonet是一款src资产管理系统，从子域名、端口服务、漏洞、爬虫等一体化的资产管理系统<!--more-->
[https://github.com/CTF-MissFeng/bayonet](https://github.com/CTF-MissFeng/bayonet)

搭建环境全部在Ubuntu上完成
git获取bayonet
```
git clone https://github.com/CTF-MissFeng/bayonet
```
安装postgresql
```
sudo apt-get install postgresql postgresql-client
```
安装完毕后 把数据库链接写入到WebConfig.py中 顺便把shodan的key也写进去

![](https://cdn.wtfsec.org/img/20200224003827.png)

要修改数据库密码 参照[https://www.cnblogs.com/wuling129/p/4917574.html](https://www.cnblogs.com/wuling129/p/4917574.html)

然后新建bayonet库，在psql终端里
```
CREATE DATABASE bayonet;
```
运行一下app.py 注释掉最后运行函数 如果没报错数据库就链接成功了

![](https://cdn.wtfsec.org/img/20200224003931.png)

再注释掉前三个 运行最后一个app函数 如果是搭建在外网可以像图片里一样绑定host

![](https://cdn.wtfsec.org/img/20200224003948.png)

运行app.py 如果各种依赖库没问题，那么会像下图一样启动一个web端。默认用户名为: `root/qazxsw@123`

![](https://cdn.wtfsec.org/img/20200224004008.png)

接下来安装需要的各种库
```
pip3 install -r requirements.txt
```
安装crawlergo以及Chromium浏览器
```
wget https://github.com/0Kee-Team/crawlergo/releases/download/v0.2.0/crawlergo_linux_amd64.zip
wget https://commondatastorage.googleapis.com/chromium-browser-snapshots/Linux_x64/728511/chrome-linux.zip
```
安装Chromium需要的依赖
```
apt install -yq --no-install-recommends libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 libnss3
```
将chromium的可执行文件路径填到/bayonet/tools/scan/Chromium/下的Run.py中
w13scan这里踩了好久的坑,ssl模块openssl一直报错,重装好几个版本都不行。最后重装了一遍w13scan的所有库
[https://github.com/w-digital-scanner/w13scan/blob/master/requirements.txt](https://github.com/w-digital-scanner/w13scan/blob/master/requirements.txt)

所有环境装完以后准备跑起来
先添加域名

![](https://cdn.wtfsec.org/img/20200224004152.png)

接下来分别执行oneforall，portscan，Chromium，w13scan目录下的Run.py
我这里用bash一键运行全部挂到进程处理，一定要加上绝对路径

![](https://cdn.wtfsec.org/img/20200224004208.png)

当然没法实时看到各个模块的信息了 不嫌麻烦可以开四个screen进程随时观察
附一张结果图,还是挺强的,跟xray比起来不逞多让。

![](https://cdn.wtfsec.org/img/20200224004225.png)

最后说一句
## 全部要用python3
## crawlergo 永远滴神！！！

![](https://cdn.wtfsec.org/img/20200224004243.png)
