---
layout: Network
title: CentOS6 部署VMESS+TCP+TLS
date: 2020-06-17
comments: false
categories: 计算机网络
tags: Network
urlname:  deeply_vmess_ws_tls_on_centos6
---

# 起因

由于新开了一台韩国的VPS，需要重新配置一些东西。它默认只给了CentOS6的系统，懒得发工单改了，所以就记录一下。

注意，这里不再是以前的Websocket+TLS，而是参照Trojan的TCP+TLS

# 步骤

### 1.域名解析

我的域名是在Cloudflare上解析的，登录Cloudflare，之后创建A记录到我服务器的IP即可。

### 2.安装nginx

这里决定采用编译安装的方式安装最新的1.19.0。

#### 2.1 下载源码

#### 2.2 安装依赖

```shell
yum update
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel vim
```

#### 2.3 编译安装

```shell
#添加用户和组
groupadd www
useradd -g www www

#配置
./configure \
--user=www \
--group=www \
--prefix=/usr/local/nginx \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_realip_module \
--with-threads

#编译
make

#安装
make install

# 创建软链接
ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx



```

#### 2.4 创建init.d脚本文件

`vim /etc/init.d/nginx`

```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/etc/nginx/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $prog -HUP
    retval=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

####2.5 开机自启

```shell
chmod a+x /etc/init.d/nginx
chkconfig --add /etc/init.d/nginx
chkconfig nginx on
```

### 3.申请证书

要用正常的TLS得有签名的证书才行，这里我们采用certbot来实现 lets encrypt证书的申请和自动续期

#### 3.1 下载certbot

```shell
wget https://dl.eff.org/certbot-auto
sudo mv certbot-auto /usr/local/bin/certbot-auto
sudo chown root /usr/local/bin/certbot-auto
sudo chmod 0755 /usr/local/bin/certbot-auto
```

#### 3.2 选择Certbot的运行方式

`sudo /usr/local/bin/certbot-auto --nginx`

成功之后证书存在

```shell
ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem; # managed by Certbot
```





#### 3.3 设定自动续期

1. ```shell
   echo "0 0,12 * * * root python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/local/bin/certbot-auto renew -q" | sudo tee -a /etc/crontab > /dev/null
   ```



### 4.配置V2ray

#### 4.1 编译haproxy

##### 4.1.1 源码下载

```shell
wget http://www.haproxy.org/download/2.2/src/devel/haproxy-2.2-dev9.tar.gz
tar -xzf haproxy-2.2-dev9.tar.gz
```

##### 4.1.2 编译

```shell
make TARGET=linux-glibc  ARCH=X86_64  PREFIX=/usr/haproxy USE_NS=
make install
```

##### 4.1.3 添加环境变量

`vim ~/.bashrc`

最后一行加入

`export PATH=/usr/local/bin:$PATH`

##### 4.1.4 创建配置文件

```shell
mkdir /etc/haproxy/
vim /etc/haproxyhaproxy.cfg
```

代码内容如下

```shell
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
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

##### 4.1.5 创建haproxy系统服务启动脚本：

`vim /etc/init.d/haproxy`

代码内容如下

```shell
#!/bin/sh
#
# haproxy
#
# chkconfig:   - 85 15
# description:  HAProxy is a free, very fast and reliable solution \
#               offering high availability, load balancing, and \
#               proxying for TCP and  HTTP-based applications
# processname: haproxy
# config:      /etc/haproxy/haproxy.cfg
# pidfile:     /var/run/haproxy.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

exec="/usr/local/sbin/haproxy"
prog=$(basename $exec)

[ -e /etc/sysconfig/$prog ] && . /etc/sysconfig/$prog

cfgfile=/etc/haproxy/haproxy.cfg
pidfile=/var/run/haproxy.pid
lockfile=/var/lock/subsys/haproxy

check() {
    $exec -c -V -f $cfgfile $OPTIONS
}

start() {
    $exec -c -q -f $cfgfile $OPTIONS
    if [ $? -ne 0 ]; then
        echo "Errors in configuration file, check with $prog check."
        return 1
    fi
 
    echo -n $"Starting $prog: "
    # start it up here, usually something like "daemon $exec"
    daemon $exec -D -f $cfgfile -p $pidfile $OPTIONS
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    # stop it here, often "killproc $prog"
    killproc $prog 
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    $exec -c -q -f $cfgfile $OPTIONS
    if [ $? -ne 0 ]; then
        echo "Errors in configuration file, check with $prog check."
        return 1
    fi
    stop
    start
}

reload() {
    $exec -c -q -f $cfgfile $OPTIONS
    if [ $? -ne 0 ]; then
        echo "Errors in configuration file, check with $prog check."
        return 1
    fi
    echo -n $"Reloading $prog: "
    $exec -D -f $cfgfile -p $pidfile $OPTIONS -sf $(cat $pidfile)
    retval=$?
    echo
    return $retval
}

force_reload() {
    restart
}

fdr_status() {
    status $prog
}

case "$1" in
    start|stop|restart|reload)
        $1
        ;;
    force-reload)
        force_reload
        ;;
    check)
        check
        ;;
    status)
        fdr_status
        ;;
    condrestart|try-restart)
      [ ! -f $lockfile ] || restart
    ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|try-restart|reload|force-reload}"
        exit 2
esac
```

给该脚本授予可以执行的权限并启动haproxy服务：

```shell
chmod 755 /etc/init.d/haproxy
service haproxy start
#开机启动
chkconfig --add haproxy
chkconfig --level 2345  haproxy on
```

#### 4.2 配置v2ray服务端

```json
{
    "inbounds": [
        {
            "protocol": "vmess",
            "listen": "127.0.0.1",
            "port": 40001,
            "settings": {
                "clients": [
                    {
                        "id": "f2435e5c-9ad9-4367-836a-8341117d0a5f"
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

#### 4.3 修改nginx服务端

`vim /etc/nginx/nginx.conf`

 在 http{} 里面添加

```text
server {
  listen 8080;
  server_name example.com;
  root /var/www/html;
}
```

HaProxy 的证书和密钥放于同一个文件，与 Caddy 和 Nginx 不同，可以使用命令 `cat example.com.crt example.com.key > example.com.pem` 合成证书



不多介绍了，参考链接吧

> [https://guide.v2fly.org/advanced/tcp_tls_web.html#%E8%83%8C%E6%99%AF](https://guide.v2fly.org/advanced/tcp_tls_web.html#背景)

# 下一步计划

采用全docker方案解决，以方便复用

# 参考

1.[CentOS 7 源码编译安装 Nginx](https://www.cnblogs.com/stulzq/p/9291223.html)

2.[Nginx Init Scripts](https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/)

3.[Nginx on CentOS 6](https://certbot.eff.org/lets-encrypt/centos6-nginx.html)

4.[TCP + TLS + Web](https://guide.v2fly.org/advanced/tcp_tls_web.html#背景)

5.[HAproxy指南之haproxy编译安装](https://blog.51cto.com/blief/1750573)

