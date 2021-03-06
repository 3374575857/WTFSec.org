+++
title = "悠遊卡寫入任意NFC設備"
author = "Hzllaga"
date = 2020-04-22
categories = [ "原创" ]
tags = ["笔记","IC","NFC"]
+++
去年的事，今年剛好有空就整理整理，大部分非cpu卡的非接觸式ic卡都可以dump出資料，本文以悠遊卡為例

當時悠遊卡只有稍微研究一下，超商可以購得2種卡，一種是有PRNG漏洞的卡，可以暴力破解直接導出，另一種是沒漏洞的全加密卡，需要去機器側錄密碼

第一種沒什麼難度，幾分鐘就可以破解完畢，而第二種卡我是把電腦搬去ubike的地方側錄，使用的工具是proxmark3:

```
proxmark3> hf 14a snoop

#db# Starting to sniff
#db# maxDataLen=172, Uart.state=0, Uart.len=0
#db# traceLen=4661, Uart.output[0]=000000e7
```

pm3放在悠遊卡和讀卡機中間刷幾次卡，刷完看pm3的log文件

![](https://cdn.wtfsec.org/img/20200422123029.png)

可以看見Auth(B) 就是用某個扇區的b密碼對悠遊卡取/寫值

然後利用XOR計算機算出密碼

![](https://cdn.wtfsec.org/img/20200422123410.png)

有了這個密碼就可以對這張卡進行暴力破解，破解出16個扇區的ab密碼就可以導出了。

有了導出的數據，可以找穿戴式NFC裝置來寫入(比如小米手環、NFC戒指等)，出門就不用帶卡了。

以小米手環為例:

先把uid寫到一張白卡不要加密讓手環模擬，模擬好後用pm3把剛才的數據寫入就可以了(小米手環的廠商代碼是寫死的，雖然有辦法破解，但是完全沒必要，因為所有悠遊卡設備都不會管廠商代碼，0扇區塊0只會讀取uid做用戶識別)

雖然悠遊卡會把餘額暫存到卡片裡，但是每次刷卡都會聯網校驗的，切勿以身試法。