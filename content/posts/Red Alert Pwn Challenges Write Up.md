+++
title = "Red Alert Pwn Challenges Write Up"
author = "Hzllaga"
date =  2019-07-14 
description = ""
categories = [ "原创" ]
tags = ["WriteUP"]
+++
## 題目1-日誌分析
> 在2019年的05的某一天資安設備突然出現有異常的日誌告警，發現我們的ＷebServer 疑似被人家攻擊可能也被駭客植入後門，我們備份了Apache 相關日誌，請協助我們找到問題 請嘗試尋找Flag (Red_Alert_XXXX)

很簡單的一題，直接從Access Log裡面去撈相關Payload就好了。
![](https://cdn.wtfsec.org/img/20200222170135.png)
Flag: `Red_Alert_72_2a5db88fd54297fae361a4d227c4691d`

## 題目2-惡意程式檔案分析
> 在今年的五月中，我們收到一封來自國內包裹email ，不知道附檔是否有問題， 請協助我們分析看看裡面是否有問題， 如有找到問題請嘗試著幫我們分析是哪個漏洞所(CVE編號所造成的攻擊
> #解壓縮密碼為:infected
>Flag格式為： Red_Alert_CVE編號_該檔案的md5 exp: Red_Alert_CVE-2010-11231_2cd6ee2c70b0bde53fbe6cac3c8b8bb1

這題基本上很難從檔案去判斷，所以可以用防毒軟體來判斷特徵，比方說我電腦安裝的卡巴斯基：
![](https://cdn.wtfsec.org/img/20200222170327.png)
最後算個文件MD5值就可以了：
![](https://cdn.wtfsec.org/img/20200222170349.png)
另解（應該也是最多人用的解）：
丟到virustotal之類的網站去掃，有些防毒會直接跟你講是哪個CVE漏洞：
![](https://cdn.wtfsec.org/img/20200222170416.png)
Flag: `Red_Alert_CVE-2017-11882_43025a9793aededdd45db93ec313cab5`

## 題目3-流量分析
> 在今年的五月份，資安設備收到異常的流量告警，我們找到了被攻擊的主機並且進行封包錄製，請嘗試著從封包中找到駭客藏匿的Flag
> 檔案下載：[https://drive.google.com/drive/folders/1QSoKR75BmteyPBcsf_kLxnaBc4JTZypE?usp=sharing](https://drive.google.com/drive/folders/1QSoKR75BmteyPBcsf_kLxnaBc4JTZypE?usp=sharing )
> 
> Flag格式為： Red_Alert_DOS_{md5}

TBC
## 題目4-VM分析
> 在上半年，公司發現有一台電腦有異常的連線行為，我們將這台電腦初步進行了複製，並且打包成VM，請嘗試著幫我們分析這台受害電腦，並且設法找到駭客中繼站，並且將flag找出。
> VM 需要連到VPN 匯入VM時網路請選擇NAT Flag 將放在外面的中繼站底下。 Flage格式為:Red_Alert_72_{Flag}
> 虛擬機載點:[https://drive.google.com/file/d/1oWl3KmARElASynrIe1-ijOYgwKzaM9AM/view?usp=sharing](https://drive.google.com/file/d/1oWl3KmARElASynrIe1-ijOYgwKzaM9AM/view?usp=sharing)

題目中提到了有異常連線行為，可以很明確的定位到殭屍網路相關的木馬病毒，本來想說用火絨劍來分析一波，或是逆向病毒程序來找出中繼站，不過後來想到可以直接觀察tcp連接來找到駭客的中繼站。

首先，電腦快速掃毒確認病毒在哪邊：
![](https://cdn.wtfsec.org/img/20200222170701.png)
可以看到沒幾秒就找到了，再來就去看看這個病毒路徑：
![](https://cdn.wtfsec.org/img/20200222170722.png)
空的，很明顯，檔案被隱藏了，把顯示隱藏檔案/資料夾的選項開下去就可以了：
![](https://cdn.wtfsec.org/img/20200222170751.png)
然後直接運行程序，讓他跟駭客的主控端做連線，然後執行tasklist /svc查看運行的程序與pid：
![](https://cdn.wtfsec.org/img/20200222170816.png)
根據pid去找port連線，執行netstat -ano：
![](https://cdn.wtfsec.org/img/20200222170837.png)
可以看到駭客的主控端是運行在172.16.67.250這個server上的8080 port，那題目要求在中繼站找到flag，那其實蠻明顯的應該是從web server下去找，直接看80 port有沒有東西：
![](https://cdn.wtfsec.org/img/20200222170902.png)
Bingo，果然是要從這邊找flag，那其實根據CTF的套路，flag要嘛在/flag要嘛就在/flag.txt，果然找到了
![](https://cdn.wtfsec.org/img/20200222170920.png)

Flag: `Red_Alert_72_a743cca38f00eb008108bf76aed1e21b`
## 題目5-VM分析(2)
> 承題目4請嘗試分析該受害VM 中藏的Flag Flage格式為:Red_Alert_72_{Flag}

像這種大海撈針的題目，相信有點經驗的都知道要用everything之類的工具下去找，果不其然直接就搜出flag位置了：
![](https://cdn.wtfsec.org/img/20200222171031.png)
![](https://cdn.wtfsec.org/img/20200222171046.png)
Flag: `Red_Alert_72_228f9880186cb1b17578b91c64debd2b`
## 題目6-VM分析(3)
> 承題目5 請嘗試分析該VM “user” 帳號的密碼 Flag 格式為：Red_Alert_72_password

破解windows用戶組密碼肯定是用mimikatz這個神器了，直接privilege::debug給他權限，然後sekurlsa::logonpasswords抓取密碼：
![](https://cdn.wtfsec.org/img/20200222171145.png)
flag: `Red_Alert_72_1qaz2wsx3edc`