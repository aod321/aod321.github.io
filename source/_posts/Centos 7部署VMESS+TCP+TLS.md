---

layout: Network
title: CentOS7 部署VMESS+TCP+TLS
date: 2020-06-17
comments: false
categories: 计算机网络
tags: Network
urlname:  depoly_vmess_tcp_tls_on_centos7
---

# 起因

由于忍不住剁手新入一台韩国Kdatacenter的VPS，从南京联通出去的链路还是不错的，如图所示。
![image.png](https://i.loli.net/2020/06/17/tvlpqiFUckWEjsQ.png)

于是乎，需要给它重新配置一些东西。默认给的操作系统是CentOS6，我发工单改成了CentOS7。它家工单系统蛮有意思的，我半夜三点钟发居然十分钟后就回复了，看来是有其他时区的客服。

最近发现官方社区出了TCP+TLS的指南，不再是以前的Websocket+TLS，而是参照Trojan的TCP+TLS的思路，用Haproxy作为443前端根据流量特征进行中转，具体来说就是，如果接入流量真的是http流量的话，那么就送到nginx后端，如果是其他流量的话，那么就把它送到v2ray的VMESS端口里。以前的方式是固定web path里的统统用websocket反向代理到VMESS端口。现在这种TCP+TLS的方式根据测试延迟会比以前低一些。

我个人第一想法是感觉额外添加一个Haproxy出来不够美观，如果能够只用V2ray就更加优雅了。感觉应该是可行的，比如用V2ray的任意门，但是仔细想想可能是因目前V2ray的代理在TLS的实现上有问题，所以社区大佬才没有去尝试吧。之后估计V2ray core更新后就可以不用Haproxy了。最近时间紧，我就不要去探索了，2333~

##### 以下所有操作默认都在root用户中执行

# 步骤

### 1.域名解析

我的域名是在Cloudflare上解析的，登录Cloudflare，之后创建A记录到我服务器的IP即可。

### 2.安装nginx

最省事的方法是直接用APT或者YUM装

```shell
yum update
yum install -y epel-release
yum install -y vim
yum install -y nginx
```

修改配置`vim /etc/nginx/nginx.conf`，找到server_name这一项，将localhost改为自己的域名

```
server
{
    listen 80;
    server_name example.com;	#记得改成你自己的域名
    .....
```

### 3.申请证书

要用正常的TLS得有签名的证书才行，这里我们采用certbot来实现 lets encrypt证书的申请和自动续期

#### 3.1 下载certbot

```shell
yum install certbot python2-certbot-nginx
```

#### 3.2 选择Certbot的运行方式

由于我们不需要nginx直接监听443，因此这里建议仅生成证书

```
certbot certonly --nginx
```

成功之后证书存在

```shell
ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem; # managed by Certbot
```

#### 3.3 设定自动续期

```shell
echo "0 0,12 * * * root python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```



### 4.配置Haproxy

#### 4.1 编译openssl 1.1.1

##### 我们需要openssl1.1.1要支持TLS 1.3

安装依赖

```shell
yum groupinstall 'Development Tools'
```

下载源码

```shell
wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz
tar -xzf openssl-1.1.1g.tar.gz
```

编译

```shell
cd openssl-OpenSSL_1_1_1g/
# 通常可以直接使用config (from Ubuntu 13.04, x64, 本文在CentOS7.3测试通过):
./config --prefix=/opt/openssl-1.1.1 shared
#编译
make
#安装
make install
```



#### 4.2 编译haproxy

源码下载

```shell
yum install -y make gcc perl pcre-devel zlib-devel pcre2 pcre2-devel
wget http://www.haproxy.org/download/1.9/src/haproxy-1.9.15.tar.gz
tar -zxvf haproxy-1.9.15.tar.gz
cd haproxy-1.9.15/
```

编译

```shell
make TARGET=linux2628 CPU=native USE_PCRE2=1 USE_PCRE2_JIT=1 USE_OPENSSL=1 SSL_LIB=/opt/openssl-1.1.1/lib SSL_INC=/opt/openssl-1.1.1/include USE_ZLIB=1
make install
# Check your sbin path at /usr/local/sbin
cp haproxy /usr/local/sbin/haproxy
```

创建配置文件

```shell
mkdir /etc/haproxy/
vim /etc/haproxy/haproxy.cfg
```

代码内容如下

```shell
global
    log /dev/log local0
    log /dev/log local1 notice
    #chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    #user haproxy
    #group haproxy
    daemon
    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    # 仅使用支持 FS 和 AEAD 的加密套件
    ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
    ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
    # 禁用 TLS 1.2 之前的 TLS
    ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11

    tune.ssl.default-dh-param 2048

defaults
    log global
    # 我们需要使用 tcp 模式
    mode tcp
    option dontlognull
    timeout connect 5s
    # 空闲连接等待时间，这里使用与 V2Ray 默认 connIdle 一致的 300s
    timeout client  300s
    timeout server  300s
frontend tls-in
    # 监听 443 tls，tfo 根据自身情况决定是否开启，证书放置于 /etc/ssl/private/example.com.pem
    bind *:443 tfo ssl crt /etc/ssl/private/example.com.pem
    tcp-request inspect-delay 5s
    tcp-request content accept if HTTP
    # 将 HTTP 流量发给 web 后端
    use_backend web if HTTP
    # 将其他流量发给 vmess 后端
    default_backend vmess

backend web
    server server1 127.0.0.1:8080
  
backend vmess
    server server1 127.0.0.1:40001
```

HaProxy 的证书和密钥放于同一个文件，与 Caddy 和 Nginx 不同，可以使用命令 `cat example.com.crt example.com.key > example.com.pem` 合成证书

修改haproxy系统服务启动脚本：

`vim /etc/systemd/system/haproxy.service `

```shell
# 确保在[Service]这一栏有这样的一项
[Service]
Environment=LD_LIBRARY_PATH=/opt/openssl-1.1.1/lib/
```

测试，首先测试配置是否正确，如果发现错误就根据错误改正

```bash
haproxy -db -f /etc/haproxy/haproxy.cfg
```

运行

```
/etc/init.d/haproxy start
```

如果没有任何提示，说明haproxy配置无误，Ctrl+C退出测试。

启动服务

```shell
systemctl start haproxy
#开机自启
systemctl enable haproxy
```

此时输入`netstat -npl |grep 443`可以看到haproxy已经开始监听443端口

```shell
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      23824/haproxy
unix  2      [ ACC ]     STREAM     LISTENING     290695   23824/haproxy        /run/haproxy/admin.sock.23823.tmp
```

#### Haproxy常见错误排查

如果提示

```shell
[ALERT] 168/131339 (22782) : Starting frontend GLOBAL: cannot bind UNIX socket [/run/haproxy/admin.sock]
```

首先检查443端口是否被占用，先把apache，nginx, caddy关掉，之后的章节会提到，我们会将它的监听端口改为其他端口，用haproxy作实际443监听端口。

使用`netstat -npl |grep 443`检查是否有进程占用443端口。

如果确认没有，但仍然报错，则需要手动创建/run/haproxy/这个目录：

>Haproxy needs to write to `/run/haproxy/admin.sock` but it wont create the directory for you. Create the directory `/run/haproxy/` first or set `stats socket` to a different path.

```shell
mkdir /run/haproxy/
```

##### 请注意:

***1.务必先编译openssl 1.1.1再编译haproxy，可以通过`haproxy -vv |grep OpenSSL`查看当前编译时候openssl的版本，如果版本号不对，请指定正确版本的openssl路径后重新编译***

```shell
haproxy -vv |grep OpenSSL
----
Built with OpenSSL version : OpenSSL 1.1.1g  21 Apr 2020
Running on OpenSSL version : OpenSSL 1.1.1g  21 Apr 2020
OpenSSL library supports TLS extensions : yes
OpenSSL library supports SNI : yes
OpenSSL library supports : TLSv1.0 TLSv1.1 TLSv1.2 TLSv1.3
```



###  配置v2ray服务端

安装

```bash
bash <(curl -L -s https://install.direct/go.sh)
```
配置
`vim /etc/v2ray/config.json`

内容如下:

```json
{
    "inbounds": [
        {
            "protocol": "vmess",
            "listen": "127.0.0.1",
            "port": 40001,	# 可以随便写个端口，只要注意一致就行
            "settings": {
                "clients": [
                    {
                        "id": "f2435e5c-9ad9-4367-836a-8341117d0a5f"(请自己生成一个)
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp"
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom"
        }
    ]
}
```

启动服务

```shell
systemctl start v2ray
#开机自启
systemctl enable v2ray
```



## 修改nginx服务端配置

`vim /etc/nginx/nginx.conf`

 在 http{} 里面添加，这样我们只用nginx监听普通8080端口即可，因为haproxy那边本地转发前已经解包成了明文了。

```text
server {
  listen 8080;
  server_name example.com;		#记得改成你自己的域名
  root /var/www/html;
}
```

重启服务端

```shell
systemctl restart nginx
# 开机自启
systemctl enable nginx
```

### 其他

到目前为止，所有的服务就已经搭建完毕了，在浏览器里输入 https:// + 你的域名，  如果能够正常访问，说明haproxy和nginx的链路已经通了，基本上就代表可以用了。VMESS的路一般不会出什么问题，这时候就可以用客户端试一下了。

客户端的配置以及一些延迟测试就不多介绍了，参考[官方社区](https://guide.v2fly.org/advanced/tcp_tls_web.html)吧。


# 下一步计划

1.整个一套配置下来还是很繁琐的。我打算之后写一套docker-compose，以方便复用。

2.这个机器有100G硬盘呢，反正暂时又不建站，不如下一步在上面建个图床玩玩

# 参考

1.[CentOS 7 源码编译安装 Nginx](https://www.cnblogs.com/stulzq/p/9291223.html)

2.[TCP + TLS + Web](https://guide.v2fly.org/advanced/tcp_tls_web.html#背景)

3.[HAproxy指南之haproxy编译安装](https://gist.github.com/meanevo/742b61031fdf9ce01e50d4b196a3f31e)

4.[wiki_openssl](https://wiki.openssl.org/index.php/Compilation_and_Installation)

5.[Building HAProxy so that it can use TLSv1.3](https://dnsprivacy.org/wiki/display/DP/Building+HAProxy+so+that+it+can+use+TLSv1.3)

6.[Nginx on CentOS/RHEL 7](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx.html)

7.[HAProxy doesn't start, can not bind UNIX socket](https://stackoverflow.com/questions/30101075/haproxy-doesnt-start-can-not-bind-unix-socket-run-haproxy-admin-sock)

