---
layout: Mac OS
title: DELL inspiron 7447 黑苹果折腾记录
date: 2016-04-05
categories: 计算机相关技术
tags: Mac OS
urlname:  7447-hackintosh
---
动机
---
作为一名穷逼学生总是想蹭点生活费花花。奈何生性懒惰，体质孱弱，洗不了盘子，发不了传单。

前一段时间买了个几块钱的APP玩顿感肉疼的同时突然觉得也许写个APP啥的挂在appstore上卖是个发家致富的好方法~q(≧▽≦q)

于是雄心勃勃的要去学Swift(●'◡'●)，第一步，我需要一个Mac OS系统
Mac 系统获得方案
---

> * 方案1 土豪专属，直接买一台MacBook Air/Pro

![MacBook Air](http://images.apple.com/macbook-air/images/overview_hero_hero_2x.jpg)
（╯－＿－）╯╧╧，看到这价格我内心是绝然崩溃的，我只是想水一波零花钱，这么一来我连成本都收不回来啦~
>方案2 懒人必备，虚拟机安装OS X

这个方案确实方便省时成本低，想尝试的同学可以参考我的帖子。
[虚拟机安装Mac OS](http://fcc.cumt.edu.cn/forum.php?mod=viewthread&tid=58)

但是这个方法最大的缺点就是虚拟显卡带不起来水果的图形加速，于是你一点启动器就各种卡顿，

本人强迫症，不能忍不能忍（╯－＿－）╯╧╧
> * 方案3 屌丝逆袭，普通PC机上装Mac OS

首先看看什么叫黑苹果
>黑苹果 （操作系统） 
自从苹果采用Intel的处理器，OS X被黑客破解后可以安装在Intel CPU与部分AMD CPU的机器上。从而出现了一大批未购买苹果机而使用苹果操作系统的机器，被称为黑苹果(Hackintosh)；在Mac苹果机上面安装原版Mac系统的被称为白苹果（Macintosh），与黑苹果相对。
注：摘自百度百科

本次安装黑苹果将使用一个叫做Clover的引导，这是一个开源的EFI bootloader引导项目，支持OSX的启动和参数更改，具体请参看

[Clvoer efi bootloader](https://sourceforge.net/projects/cloverefiboot/)


以安装*OS X 10.11.3 EI Captin*系统为例

开始折腾
----
配置单以及驱动情况
----
> * 电脑品牌：戴尔灵越7447 Dell Inspiron 7447

> * CPU：I5-4200H Haswell架构
> * 内存：4G+4G
> * 系统：Win10 1151 updated
> * 硬盘：128SSD+1024GBHHD
> * 核心显卡：Intel(R)HD Graphics 4600 (完美驱动支持HDMI输出)
> * 独立显卡（双显卡笔记本独显无解）：NVDIA GTX850M 

>**注意，采用双显卡的笔记本只可能驱动核显，除非部分笔记本出厂的时候在硬件上屏蔽核显。** 

> * 声卡： Realtek ALC255(仿冒为Applehda.kext完美驱动)
 
> * 无线网卡：原：Intel AC无线网卡(无解) 更换为:BCM94352（支持蓝牙4.0，直接安装驱动即可）

> * 有线网卡：板载网卡，免驱
 
> * 分区表：UEFI+GPT

步骤一、制作安装U盘
---
首先下载一个带Clover引导的OSX 10.11.3的包，这里推荐下载网友制作好的，方便省心。

[远景技术论坛下载地址](http://bbs.pcbeta.com/viewthread-1670310-1-1.html)

下载其中的*USB_Clover_Install_OS_X_El_10.11.3_15D21.dmg*

然后**以管理员身份运行**[transmac](http://www.acutesystems.com/scrtm.htm)工具，如图所示

![transmac](http://www.acutesystems.com/images/tmscr.gif)

然后右键点击 U盘，选择Restore from image这个项，然后选中下载，选择dmg映像，确定。这个过程如果U盘一般的话，应该需要20多分钟，请耐心等待。（务必注意事先备份U盘数据）

等制作完毕后，我们的准备工作就算是做好啦。

步骤二、安装系统，结束此步基本就成功了一大半
---
进入U盘，U盘制作完毕后，重启计算机，开机的时候按F12 (戴尔多为F12)，进入boot menu，选择U盘所对应的EFI启动项启动。如图所示
![bootmenu](../../../../img/bootmenu.png)

进入以后出现clover引导界面，和我不一样的是因为主题不一样的原因，clover可以自己更换主题，很炫酷有木有~q(≧▽≦q)
![clover引导界面](http://cdn.pcbeta.attachment.inimc.com/data/attachment/album/201403/02/13595029lk2q26fxf36s2s.png)
大概像这样，图片上的选择很多是因为硬盘里装的系统多，实际安装的时候不会出现这么多选项的，

找到一个叫boot os x install from os x eicaptin后，

**按空格进入选择菜单**

选择verbose mode(啰嗦模式，如果过程出错可以根据相关信息查找资料修改config.plist)

这个安装过程因人而异，有的电脑改都不用改直接就能装，有的电脑比如我的,会出现秒重启的现象，出现这种现象，只要打开U盘中clvoer目录先的config.plist，将其中kernalPM改为ture即可。

其它的错误请大家根据报错信息到各大论坛上发帖求助。一定要沉住气，有耐心，过了这一步，基本上就成功80%,只剩下一些驱动问题。

等加载过程完毕后，将会出现OS X的安装界面（这多半是个好兆头），如图，安装前需要先打开工具里的磁盘工具
![磁盘工具](http://images.weiphone.net/data/attachment/forum/201401/09/100220nr44orrv8farro0c.jpg)
---------------------------------2016年7月23日更新---------------------------
-
虽然最近先是忙考试又是电设的，不过自己写的博文跪着也得更新完呀QAQ
![磁盘工具](/img/Disk.png)
如图，直接把整块硬盘点抹除（好吧，其实可以分区，不过讲起来好麻烦，我就偷偷懒）。嗯，回到安装页面，选择那个分区点安装就好，安装的剩下步骤没什么好说的了，跟着提示走就好。

步骤三、完善驱动
---
装完以后终于可以进去美美哒的Mac OS啦，不过先不要高兴的太早，现在这个Mac各个部分的驱动都还没有安装，wifi、电源管理、关机、亮度调整、声音等等都需要调整，是不是听着就头大了呢？
