---

layout: Network
title: Centos7搭建shadowsocksR
date: 2016-12-04
categories: 计算机相关技术
tags: Network
urlname: centos-installssr
---

## 前言
本文是在免费学习机房分享的时候现场编写的，现在整理一下顺便发出来，其实就是搬的brewa11的github上的wiki,感兴趣的同学可以去原文看。

本文仅用来整理自己的步骤。

<!-- more -->

## 介绍

### Shadowsocks

>Shadowsocks（中文名称：影梭）是使用Python、C++、C#等语言开发的、基于Apache许可证的开放源代码软件，用于保护网络流量、加密数据传输。Shadowsocks使用Socks5代理方式。
Shadowsocks分为服务器端和客户端。在使用之前，需要先将服务器端部署到服务器上面，然后通过客户端连接并创建本地代理。
在中国大陆，本工具也被广泛用于突破防火长城（GFW），以浏览被封锁、屏蔽或干扰的内容。在2015年8月22日，Shadowsocks原作者Clowwindy称受到了中国政府的压力，宣布停止维护此项目并移除其用户页面所载的源代码。[2][3]

![Shadowsocks](https://upload.wikimedia.org/wikipedia/commons/8/8d/Shadowsocks_logo.png)

### Shadowsocks-RSS

Shadowsocks原本停止维护后，由@breakwa11继续参与维护的一个shadowsocks的版本。

## 搭建方法

 - 脚本一键安装

网上有很多网友自己制作的省心的一键搭建脚本。如果只是想要搭建不想细究和维护，可以考虑使用一键脚本。但手动可以帮助我们了解它。
如：[秋水逸冰的一键脚本](https://shadowsocks.be/9.html)
使用方法参照原文博客，不再赘述。如果只想要有个能用的SSR，到这里就可以止步了。

 - 手动安装
手动安装的方法原作者的博客已经整理的非常详细了，可以参考
[Server-Setup](https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup)
在这里针对CentOS7简单地记录一下过程。

## 手动安装的步骤

### 需要条件

 - 有一台装有CentOS7且接入互联网的服务器。

	一般都是VPS。如果是用来查阅墙外资料的话推荐国外vps，推荐Digitalocean，这家可以申请github学生优惠包，一年才5$，比较划算。DO家的新加坡线路对移动用户还是比较友好的。
	
	有如果是单纯用来学习如何搭建的话国内一些厂商的学生机还是比较便宜的，比如腾讯云的1元/月，阿里云的9.9/月等等，都足够用来学学网络编程或者搭建自己的主页使用了。

 -  有一定的Linux系统知识：ls、cd、wget、git
 - 有爱折腾的精神
 
### 具体操作步骤
- 连接上VPS，ssh

- 输入命令
> ` yum install git
   ` 
> ` git clone -b manyuser https://github.com/shadowsocksr/shadowsocksr.git
   ` 

执行完毕后此目录会新建一个shadowsocksr目录，其中根目录的是多用户版（即数据库版，个人用户请忽略这个），子目录中的是单用户版(即shadowsocksr/shadowsocks)。

根目录即 ./shadowsocksr

子目录即 ./shadowsocksr/shadowsocks

进入根目录初始化配置(假设根目录在~/shadowsocksr，如果不是，命令需要适当调整)：

> ` cd ~/shadowsocksr
   ` 	
> `bash initcfg.sh
	`

以下步骤要进入子目录：

> `cd ~/shadowsocksr/shadowsocks
	`

修改user-config.json中的server_port，password等字段，具体可参见：
[config.json](https://github.com/breakwa11/shadowsocks-rss/wiki/config.json)

> `python server.py
`

如果要在后台运行：
>`python server.py -d start
`

>`python server.py -d stop/restart
`

防火墙设置

运行server.py后发现客户端仍然连接不上，因为指定端口没有开放。
CentOS7以后默认防火墙从原来的iptables变成了firewalld，开放端口的命令如下:

> `firewall-cmd --zone=public --add-port=端口/tcp --permanent
`

把端口换成自行设置的ServerPort即可。

### 设置开机启动

以上步骤完成后，应该就可以愉快地使用ssr了，不过每次开机都需要手动运行比较麻烦，这里还需要配置一下开机自启。

```
[Unit]
Description=ShadowsocksR server
After=network.target
Wants=network.target

[Service]
Type=forking
PIDFile=/var/run/shadowsocks.pid
ExecStart=/usr/bin/python /usr/local/shadowsocksr/shadowsocks/server.py --pid-file /var/run/shadowsocks.pid -c /etc/shadowsocks.json -d start
ExecStop=/usr/bin/python /usr/local/shadowsocksr/shadowsocks/server.py --pid-file /var/run/shadowsocks.pid -c /etc/shadowsocks.json -d stop
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=always

[Install]
WantedBy=multi-user.target

```

请将上述脚本保存为/etc/systemd/system/shadowsocks.service


并执行systemctl enable shadowsocks.service && systemctl start shadowsocks.service

--

### Done!Enjoy it ~