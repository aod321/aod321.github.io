---
layout: Network
title: Centos7搭建shadowsocksR
categories: 计算机相关技术
tags:Network
---

##前言

最近事情比较多，正好最近要和大家分享一下关于代理工具的知识，懒得做PPT了，干脆就直接用Markdown写，回头还方便直接传到我的博客上面去.
本文仅用来整理自己的步骤。

##介绍

###Shadowsocks

>Shadowsocks（中文名称：影梭）是使用Python、C++、C#等语言开发的、基于Apache许可证的开放源代码软件，用于保护网络流量、加密数据传输。Shadowsocks使用Socks5代理方式。
Shadowsocks分为服务器端和客户端。在使用之前，需要先将服务器端部署到服务器上面，然后通过客户端连接并创建本地代理。
在中国大陆，本工具也被广泛用于突破防火长城（GFW），以浏览被封锁、屏蔽或干扰的内容。在2015年8月22日，Shadowsocks原作者Clowwindy称受到了中国政府的压力，宣布停止维护此项目并移除其用户页面所载的源代码。[2][3]

![Shadowsocks](https://upload.wikimedia.org/wikipedia/commons/8/8d/Shadowsocks_logo.png)

###Shadowsocks-RSS

Shadowsocks原本停止维护后，由@breakwa11继续参与维护的一个shadowsocks的版本。

##搭建方法

 - 脚本一键安装

网上有很多网友自己制作的省心的一键搭建脚本。如果只是想要搭建不想细究和维护，可以考虑使用一键脚本。但手动可以帮助我们了解它。
如：[秋水逸冰的一键脚本](https://shadowsocks.be/9.html)
使用方法参照原文博客，不再赘述。如果只想要有个能用的SSR，到这里就可以止步了。

 - 手动安装
手动安装的方法原作者的博客已经整理的非常详细了，可以参考
[Server-Setup](https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup)
在这里针对CentOS7简单地记录一下过程。

##手动安装的步骤

###需要条件

 - 有一台装有CentOS7且接入互联网的服务器。

	一般都是VPS。如果是用来学习的话国内一些厂商的学生机还是比较便宜的，比如腾讯云的1元/月，阿里云的19.9/月等等，都足够用来学学网络编程或者搭建自己的主页使用了。

 -  有一定的Linux系统知识：ls、cd、wget、git
 - 有爱折腾的精神
 
###具体操作步骤
- 连接上VPS，ssh

> ` yum install git
   ` 
   
   
   
   

 