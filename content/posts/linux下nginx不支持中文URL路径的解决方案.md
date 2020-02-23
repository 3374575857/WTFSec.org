+++
title = "linux下nginx不支持中文URL路径的解决方案"
author = "Hzllaga"
date =  2017-09-23 
description = "linux下nginx不支持中文URL路径的解决方案"
categories = [ "转载" ]
tags = ["Linux"]

+++
## 步骤
确认系统编码 -> 设置nginx编码  -> 将非UTF-8的文件名转换为UTF-8编码
<!--more-->

1.确定你的系统是UTF编码
{{< highlight shell >}}
[root@localhost ~]# echo $LAGN
en_US.UTF-8
{{< /highlight >}}

2.nginx配置文件里默认编码设置为utf-8
{{< highlight nginx >}}
server{
	listen 80;
	server_name .inginx.com ;
     index index.html index.htm index.php;
     root /usr/local/nginx/html/inginx.com;
     charset utf-8;
}
{{< /highlight >}}

3.将非UTF-8的文件名转换为UTF-8编码

做法很简单，把文件名都修改成utf8编码就可以了！
安装convmv，由他去转换编码：
{{< highlight bash >}}
yum install convmv -y
convmv -f GBK -t UTF8 -r --notest 目标路径
{{< /highlight >}}
其中-f是源编码，-t是目标编码，-r是递归处理目录，–notest是不移动，实际上对文件进行改名。