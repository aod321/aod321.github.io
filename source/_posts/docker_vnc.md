---
layout: Server
title:  利用Docker VNC在无图形服务器上跑图形桌面
date: 2021-08-10 23:14:32
categories: Server
tags: Server
urlname: nogui_linux_docker_vnc_matlab
---
# 利用Docker VNC在无图形服务器上跑图形桌面

现在很多服务器都是不提供用户图形界面的，但是科研人员有时候需要在服务器上运行图形界面，比如使用图形化的Matlab 处理数据等。虽然这时候可以选择安装一个 gnome 或者 xfce4 等桌面环境，但是通常服务器上是有很多用户的，这样会可能会占用一定的资源或者把环境搞乱掉，影响其他用户使用。

比较好的方案是起一个带 VNC 的 Docker 容器，省事还干净，不会影响到其他人。

以下以在一个装有 Docker 的无图形界面服务器上安装运行 Matlab 为例，为大家讲解具体的操作。

假设服务器 IP 是 192.168.101.32
1. 开启 带 VNC 环境的docker容器，将 matlab 所在路径(假设安装到了本机的/share/matlab)映射到容器内的某个路径，把 5901 端口映射到本地服务器的 15901(也可以写成别的)。

````shell
docker run -d -v /share/matlab:/matlab -p 15901:5901 accetto/ubuntu-vnc-xfce-chromium-g3
````

2. VNC 客户端访问192.168.101.32:15901即可访问桌面

3. 访问桌面后，在VNC 连上的桌面上打开一个终端，然后在该终端中输入如下命令

```shell
# 将APT源换成清华镜像源
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
# 安装打开图形的必要依赖：JRE
sudo apt update
sudo apt install default-jre
# 执行 matlab
/matlab/R2021a/bin/matlab
```

   


