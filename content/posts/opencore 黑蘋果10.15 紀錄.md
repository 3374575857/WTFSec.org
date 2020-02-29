+++
author = "3374575857"
title = "opencore 引導黑蘋果10.15紀錄"
date = "2020-02-29 06:00:00"
description = ""
tags = [
    "Hackintosh"
]
categories = [
    "原創"
]
+++
本文沒有要具體的教如何配置，只把過程有比較需要注意的事項記錄下來。<!--more-->

## 推薦需求
- 一個可用的mac系統
- AMD免驅動顯卡
- Intel CPU
- 一個可以英翻中的瀏覽器

## 推薦教程
[使用OpenCore引导黑苹果](https://blog.xjn819.com/?p=543)

[精解OpenCore](https://blog.daliansky.net/OpenCore-BootLoader.html)

[OpenCore 官方文檔](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)

[Opencore Vanilla Desktop Guide](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/)

強烈推薦以[Opencore Vanilla Desktop Guide](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/)為主，xjn大佬的[使用OpenCore引导黑苹果](https://blog.xjn819.com/?p=543)為輔，兩個教程搭配食用。

## 我的配置
cpu：intel i5 4590

主板：asus B85M-G

顯卡：GT730 2G

![](https://cdn.wtfsec.org/img/20200229002.png)

自用efi:[github](https://github.com/3374575857/i5-4590-B85M-gt730-hackintosh)

---
## 必要軟件

- [OpenCore](https://github.com/acidanthera/OpenCorePkg/releases):OpenCore主程序
- [ProperTree](https://github.com/corpnewt/ProperTree):修改配置文件用
- [MountEFI](https://github.com/corpnewt/MountEFI):掛載efi分區
- [MaciASL](https://github.com/acidanthera/MaciASL/releases):提取DSDT以及編譯DSDT
- [Hackintool](https://www.tonymacx86.com/threads/release-hackintool-v2-8-6.254559/):USB訂製、尋找硬件地址...等
- 必要的efi kext文件
## 開工

<b>強烈建議先在當前主機上配置一個clover引導的黑蘋果([黑果小兵的安裝教程](https://blog.daliansky.net/MacOS-installation-tutorial-XiaoMi-Pro-installation-process-records.html))臨時使用，配置完opencore再將引導替換成opencore的即可。</b>

首先打開[Opencore Vanilla Desktop Guide](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/)這份教程開始閱讀，花點時間先大概地看過一遍，看完之後就可以按照他的方法開始配置config.plist，有不明白的部分就去翻閱xjn大佬的[使用OpenCore引导黑苹果](https://blog.xjn819.com/?p=543)再不行就看黑果小兵的[精解OpenCore](https://blog.daliansky.net/OpenCore-BootLoader.html)，每個作者都有不太一樣的詮釋，截長補短的效果顯著。

### 小技巧
使用ProperTree編輯配置文件可以在鍵盤上按 `Cmd/Ctrl + Shift + R` 然後指向配置好的oc目錄，他就會幫你自動填入對應ACPI、efi、kext的設定，並刪除官方默認添加沒有使用的ACPI、efi、kext設定。

（如圖，不但自動填上了還按優先級排了序）

![](https://cdn.wtfsec.org/img/20200229003.png)



### 聲卡注入

這部分注入聲卡推薦閱讀xjn的`2.3.1  声卡`部分，要清楚的多。

這裡還有一個要注意的點，聲卡信息可能不只一個，要注意看清楚不要填錯了，我開始在弄的時候沒仔細看，看到是聲卡的就直接寫上去了，殊不知填到的是顯卡部分的聲卡地址，回去排查的時候才發現。

![](https://cdn.wtfsec.org/img/20200229001.png)

### 開啟高效小睡(小憩)自動喚醒

如果啟用了原生電源管理並且節能設定中勾上了高效小睡(小憩)必須啟用這個官方的補丁，不然會有睡眠自動喚醒的問題。

Kernel -> Patch -> 0 -> Disable RTC wake scheduling

Enabled -> True

![](https://cdn.wtfsec.org/img/2020022900004.png)


### 定製USB接口
推薦閱讀[這篇](https://selfishluck.top/2020/02/19/Hackintools定制USB驱动（OpenCore）/)，其實也沒這麼麻煩
直接在配置文件中啟用這個
`Kernel -> Quirks -> XhciPortLimit -> True`

然後重啟電腦並使用這個配置文件的opencore引導開機，打開hackintool的USB部分

![](https://cdn.wtfsec.org/img/20200229005.png)


會看到所有的usb接口，識別到的是綠的，然後拿著`2.0` `3.0`的USB在電腦上挨個插，把看得到的洞都插完了之後在把不是綠的刪幾個掉，直到總數是15個以內。然後把長年插著的及常用的usb口設定為內置(Internal)，這樣可以解決一部份進入睡眠而自動喚醒的問題。

都設定完了之後就可以右下角導出配置文件

![](https://cdn.wtfsec.org/img/20200229006.png)


配置文件會自動幫你存在桌面

![](https://cdn.wtfsec.org/img/20200229007png.png)


把`USBPorts.kext` 複製到opencore對應的目錄(`efi/oc/Kexts`)中，並且在配置文件中引入這個kext文件。

![](https://cdn.wtfsec.org/img/20200229008.png)


最後再把剛設定的15端口補丁設置為False `Kernel -> Quirks -> XhciPortLimit -> False`
存檔重新開機就好了。

我提供的教學文中說要引入`USBPower.kext`才能使用，本人親測不引入也能使用。

### 注入三碼
使用icloud 及 imessage 相關功能需要使用。

[教程](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/extras/iservices)

日後更新補充。

## 後記
本人也是第一次配置黑蘋果，硬著頭皮耐著性子的把[Opencore Vanilla Desktop Guide](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/)看完了，開始看的時候思緒特別的亂，實際操作搭配著[使用OpenCore引导黑苹果](https://blog.xjn819.com/?p=543)去配置的時候會發現沒有作者們寫得這麼麻煩。也就幾個True False點點就好了，需要特別填寫修改的部分我上面大概都列出來了，如果有缺漏的以後再補上。

---

系統版本：macOS Catalina 10.15.3

OpenCore 版本：0.5.5

HD Graphics 4600：本人電腦的壞了，沒測試。

GT730：正常。原生驅動。注意：gt730 僅支持開普勒架構的。

3.5mm聲音：正常使用，使用AppleALC驅動。 DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x1B,0x0) -> layout-id注入ID 07000000。

有線網卡：正常。使用了kexts/IntelMausi.kext

睡眠喚醒：正常。

關機開機：正常。

iCloud & App Store & iMessage & FaceTime：請自行生成MLB、 ROM、SystemSerialNumber、SystemUUID，並相應的修改PlatformInfo -> Generic。
