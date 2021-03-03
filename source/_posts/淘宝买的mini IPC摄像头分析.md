---

layout: Diy
title: 淘宝买的mini IPC摄像头分析
comments: false
date: 2019-12-13
categories: Diy
tags: Others
urlname: analysis-of-a-mini-camera
---


一个超级mini的摄像头，带夜视、H.264编码、低功耗声称可以续航三天

![image-20190923001151881.jpg](https://i.loli.net/2021/03/03/lpSXdr6InRFQw4o.jpg)
![image-20190923001203049.jpg](https://i.loli.net/2021/03/03/7lxXHrmAyGPYD3I.jpg)


拆开看是这样

![image-20190923001216989.jpg](https://i.loli.net/2021/03/03/nvH3Tbgh765lqyo.jpg)
![image-20190923001224351.jpg](https://i.loli.net/2021/03/03/hKtpbwgyL6M7GAB.jpg)

用的这个芯片很有意思

北京君正芯片

![img](http://img.mp.itc.cn/upload/20170424/9b77e860de47463eae9494bf8adc0786_th.jpeg)

大概抓包分析了一下，貌似是直接把TLS加密的内容用Raw UDP传送。

首先我发现它提供APP上有一个推送新证书的功能，通过抓包成功地把证书和私钥给拿了下来。

![image-20190923001727840.jpg](https://i.loli.net/2021/03/03/YB1W8UZTE2gKt4I.jpg)

然后发现它UDP端口每次都会变，而且APP支持局域网搜索，所以我猜想肯定是有UDP广播，所以就抓广播包，果然抓到了关键信息。这个XML文件的信息量很大，表明了它是用onvif协议（表示在看到这个关键词的时候都不知道这啥啊，搜一下才知道原来是一个定好的标准协议，我之前还以为是作者自定义的协议。毕竟不了解IP摄像头行业，2333）

![image-20190923002052357.jpg](https://i.loli.net/2021/03/03/mt4RfykWX2Y6sHL.jpg)



另外，我还发现了这个板子的串口，哈哈哈，这意味着可以直接拿到shell了，回头焊个排针引出来然后把ssh 打开，真是捡到宝了，开心～

![image-20190923002550162.jpg](https://i.loli.net/2021/03/03/5ajSpGk47undhfv.jpg)

## 总结

这个摄像头的设计很简单又紧凑，值得学习。这个小体积和续航非常符合我的要求，下一步的目标就是对它进行二次开发了～ 加个人脸识别啥的,真是太美好了～～

