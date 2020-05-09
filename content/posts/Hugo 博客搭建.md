+++
author = "3374575857"
title = "Hugo 博客搭建"
date = "2020-05-09 08:00:00"
description = ""
tags = [
    "Hugo"
]
categories = [
    "原创"
]
+++


前段時間網站從Wordpress遷移到Hugo，一直想把折騰的紀錄寫出來，但是有點懶拖到現在才寫。<!--more-->

搭建完成的成果：暫時沒有 (跟[本站](https://wtfsec.org/)一樣的主題)

## 全新安裝

### 安裝Hugo

Mac: `brew install hugo`

Windows: `choco install hugo -confirm`  或者  `scoop install hugo`

詳細請參考[官網](https://gohugo.io/getting-started/installing/)

### 創建Hugo網站

```
hugo new site wtfsec-test
cd wtfsec-test
git init
```

### 添加主題
本文以二次修改過的hugo-notepadium主題為例

```
git submodule add https://github.com/3374575857/hugo-notepadium.git themes/hugo-notepadium
```
在config.toml 中添加 `theme = "hugo-notepadium"`

### 添加首篇文章
```
hugo new posts/my-first-post.md
```
默認生成的文章中有`draft: true`設定為true就是草稿，這篇文章網站就不會顯示，所以得設置成false或者刪除這句。

### 運行Hugo本地服務

```
hugo server --watch  --verbose
```
### 生成網站靜態文件

```
hugo -D
```
命令運行後，會在網站根目錄下創建一個public目錄，將裡面的所有文件上傳到空間上就行了。

### 提交源代碼到github倉庫

將源碼放在github上能再換環境或者電腦炸了的情況下不丟失心血，並且快速的上手。
日後換環境可以參考我文章下方的方法把源代碼下載回來。

```
git add .
git commit -m "first commit"
# 設定遠程倉庫（請在自己的github上創建一個自己的倉庫）
git remote add origin https://github.com/3374575857/wtfsec-test.git
git push -u origin master
```

## config模板

提供了一個自用的config模板供參考

```toml
#網站網址
baseURL = "http://example.org/"
#網站名稱
title = "My New Hugo Site"
#主題
theme = "hugo-notepadium"
#網站底部版權
copyright = "©2020 My New Hugo Site."

hasCJKLanguage = true

enableRobotsTXT = true
paginate = 5

defaultContentLanguage = "zh-tw"

	
# Enable Disqus
#disqusShortname = "XXX"

# Google Analytics
#googleAnalytics = ""

[markup.highlight]
codeFences = true
noClasses = false

[markup.goldmark.renderer]
unsafe = true  # enable raw HTML in Markdown

[params]
style = "light"  # default: auto. light: light theme, dark: dark theme, auto: based on system.
dateFormat = "Monday, January 2, 2006"  # if unset, default is "2006-01-02"
logo = ""  # if you have a logo png
#slogan = ""
license = "show License"  # CC License

[params.comments]
enable = false  # En/Disable comments globally, default: false. You can always enable comments on per page.

[params.math]
enable = false  # optional: true, false. Enable globally, default: false. You can always enable math on per page.
use = "katex"  # option: "katex", "mathjax". default: "katex"

[params.syntax]
use = "none"  # builtin: "prismjs", "hljs". "none" means Chroma
theme = "xcode"
darkTheme = "xcode-dark"  # apply this theme in dark mode

[params.nav]
showCategories = true       # /categories/
showTags = true             # /tags/

# custom navigation items

#[[params.nav.custom]]
#title = "About"
#url = "/about"

[[params.nav.custom]]
    title = "友連"
    url = "/link/"

[[params.link]]
    title = "WTFSec"
    url = "https://wtfsec.org/"

	
#Google 相關
[params.google]
	#Google 自訂搜尋
	#搜尋引擎 ID
	#cx = ""
	#Google 自動化廣告 
	#adsense = ""
```
## hugo-notepadium 二次開發主題
### 特性
- 添加了友情連接功能
- 添加了google站內搜尋功能
- 修改了列表頁底部分頁邏輯
- 添加了html meta標籤：keyword、description
- 其他細節修改
### 友情連接
使用友情連接功能，必須要在`content`目錄下添加`link`目錄並且添加`_index.md`文件
內容如下：
```md
+++
title = "友情連接"
type = "link"
+++
```

### 站內搜尋
使用站內搜尋功能，必須要在`content`目錄下添加`search`目錄並且添加`_index.md`文件
內容如下：
```md
+++
title = "站內搜尋"
type = "search"
+++
```
在`config.toml` 最下面的·`cx = ""`中填上自訂搜索的id。

## 使用本站提供好的環境搭建

本站提供的[代碼](https://github.com/3374575857/wtfsec-test.git)，只需要修改配置文件中少數的地方，再把文章刪除掉，就可以開始寫博客了。

Github repo : [https://github.com/3374575857/wtfsec-test.git](https://github.com/3374575857/wtfsec-test.git)

### 克隆項目
```
git clone https://github.com/3374575857/wtfsec-test.git && cd wtfsec-test
```

### 下載子模塊主題

在網站目錄下運行

```
git submodule init
git submodule update --recursive
```

如果主題有更新，就在主題的目錄下運行這個命令，更新git子模塊中的主題：

```
cd themes/hugo-notepadium
git pull origin master
```
### 運行Hugo本地服務

```
hugo server --watch  --verbose
```

## 後記
文章寫一個段落了，基本上使用應該不會有太大的問題，有機會再寫一篇hugo 搭配 github action 持續集成自動生成自動部署的文章，更加方便省事一些。