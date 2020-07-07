---

layout: MacOS
title: MacOS Acrobat Pro DC 屏蔽更新
comments: false
date: 2020-07-07
categories: 实用技巧
tags: Adobe
urlname: acrobat-updater-remove
---

#  安装与去付费

磁力链：

```
magnet:?xt=urn:btih:AD362136562FC716C6CACACF926CEA55EF5E25B2&dn=Adobe_Acrobat_DC_v20.006.20042__TNT_Torrentmac.net.dmg
```

# 屏蔽更新

Reference:https://www.reddit.com/r/AdobeZii/comments/fkzqol/solved_how_to_disable_adobe_acrobat_reader_dc/

> I found that this solution helps:
>
> Right-click Adobe Acrobat.app, go to "Contents" -> "Plugins" and delete Updater.acroplugin.

运行以下命令即可：

`rm -rf /Applications/Adobe Acrobat DC/Adobe Acrobat.app/Contents/Plugins/Updater.acroplugin`

