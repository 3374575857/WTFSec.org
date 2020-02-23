+++
title = "C# 多线程网站后台批量扫描工具开源啦"
author = "Hzllaga"
date =  2018-11-01
description = "其实工具也是最近才写的，已经修复了好多问题，不过可能还是有很多地方有问题，代码也写得非常凌乱，毕竟初学嘛~"
categories = ["原创"]
tags = [".NET","工具"]
+++
项目地址：[https://github.com/Hzllaga/BackgroundScanner](https://github.com/Hzllaga/BackgroundScanner)
## BackgroundScanner
初学C#，无聊写的软件，仿御剑后台扫描工具，支持HTTPS批量扫描，默认会将错误记录到错误日志，如果链接不合法有可能导致程序崩溃，因为多个线程同时写入文件是没有权限的，注释掉记录错误就可以避免。
![](https://cdn.wtfsec.org/img/20200223174759.png)
