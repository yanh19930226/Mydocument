# nginx简介

##  Nginx 概述

Nginx ("engine x") 是一个高性能的 HTTP 和反向代理服务器,特点是占有内存少，并发能力强，事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好

## Nginx 作为 web 服务 

Nginx 可以作为静态页面的 web 服务器，同时还支持 CGI 协议的动态语言，比如 perl、php等。但是不支持 java。Java程序只能通过与 tomcat 配合完成。Nginx 专为性能优化而开发，性能是其最重要的考量,实现上非常注重效率 ，能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数。

## 正向代理

Nginx不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。如果把局域网外的 Internet 想象成一个巨大的资源库，则局域网中的客户端要访问 Internet，则需要通过代理服务器来访问，这种代理服务就称为正向代理

## 反向代理

反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址

##  负载均衡

客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行交互，服务器处理完毕后，再将结果返回给客户端。这种架构模式对于早期的系统相对单一，并发请求相对较少的情况下是比较适合的，成本也低。但是随着信息数量的不断增长，访问量和数据量的飞速增长，以及系统业务的复杂度增加，这种架构会造成服务器相应客户端的请求日益缓慢，并发量特别大的时候，还容易造成服务器直接崩溃。很明显这是由于服务器性能的瓶颈造成的问题，那么如何解决这种情况呢？我们首先想到的可能是升级服务器的配置，比如提高 CPU 执行频率，加大内存等提高机器的物理性能来解决此问题，但是我们知道摩尔定律的日益失效，硬件的性能提升已经不能满足日益提升的需求了。最明显的一个例子，天猫双十一当天，某个热销商品的瞬时访问量是极其庞大的，那么类似上面的系统架构，将机器都增加到现有的顶级物理配置，都是不能够满足需求的。那么怎么办呢？上面的分析我们去掉了增加服务器物理配置来解决问题的办法，也就是说纵向解决问题的办法行不通了，那么横向增加服务器的数量呢？这时候集群的概念产生了，单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个

##  动静分离

为了加快网站的解析速度，可把动态页面和静态页面由不同的服务器来解析，加快解析速度,降低原来单个服务器的压力。

# 压缩包安装

## 安装gcc

### 安装 nginx 

需要先将官网下载的源码进行编译，编译依赖 gcc 环境。安装指令如下

~~~tex
yum install gcc-c++
~~~

### 安装PCRE pcre-devel

    Nginx的Rewrite模块和HTTP核心模块会使用到PCRE正则表达式语法。这里需要安装两个安装包pcre和pcre-devel,
    第一个安装包提供编译版本的库，而第二个提供开发阶段的头文件和编译项目的源代码。安装指令如下：
    yum install -y pcre pcre-devel

### 安装zlib

zlib库提供了开发人员的压缩算法，在Nginx的各种模块中需要使用gzip压缩。安装指令如下:

    yum install -y zlib zlib-devel

###  安装Open SSL

nginx不仅支持 http协议，还支持 https（即在 ssl 协议上传输 http），如果使用了 https，需要安装 OpenSSL 库。

    yum install -y openssl openssl-devel

###  解压nginx压缩包并安装

    cd /opt
    tar -zxvf nginx-1.9.6.tar.gz
    mv nginx-1.9.6 /usr/local/
    cd /usr/local/
    cd nginx-1.9.6/
       
    启用默认配置
    ./configure
    make
    make install
    
    启动nginx
    cd /usr/local/nginx/sbin
    ./nginx
    
    查看状态
    ps -ef|grep nginx
    
    关闭nginx
    ./nginx -s quit  或者 ./nginx -s stop
    
    重启nginx
    ./nginx -s reload

### 开机自启动

    vim /etc/rc.local
    添加: /usr/local/nginx/sbin/nginx

此外，进入/usr/local/nginx/conf目录可修改nginx的配置文件 -> vim nginx.conf, 譬如修改域名以及端口啥的，在server里面进行修改
## 

# docker安装

    mkdir /usr/local/nginx/html
    mkdir /usr/local/nginx/conf/conf.d
    docker run --name tmp-nginx-container -d nginx
    docker cp tmp-nginx-container:/etc/nginx/nginx.conf /usr/local/nginx/conf/nginx.conf
    docker cp tmp-nginx-container:/usr/share/nginx/html /usr/local/nginx/
    docker cp tmp-nginx-container:/var/log/nginx /usr/local/nginx/logs
    docker cp tmp-nginx-container:/etc/nginx/conf.d /usr/local/nginx/conf/
    docker rm -f tmp-nginx-container

  下面是一整行 

    docker run --name nginx --restart=always -p 80:80 -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/nginx/html:/usr/share/nginx/html -v /usr/local/nginx/conf/conf.d:/etc/nginx/conf.d -v /usr/local/nginx/logs:/var/log/nginx -d nginx
    
# #########        nginx核心知识                #########

## 配置文件

nginx配置文件由三部分组成

## 全局块

~~~text
## 配置运行nginx服务器的用户（组）
user  nginx;
## 配置允许生成的 worker_processes 数目，值越大处理的并发越高
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

~~~

从配置文件开始到events块中间的内容，主要设置一些影响nginx运行的配置指令

## events块

~~~html
events {
    ##	配置允许连接的最大数  ##
    worker_connections  1024;
}
~~~

主要配置nginx与用户的网络连接

## http块

nginx服务器中配置最频繁的部分，包括 http 全局块，server块

### http全局块

~~~text
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
}
~~~

http全局块配置的指令包括文件引入，MIME-TYPE定义、日志定义、连接超时时间、单链接请求上线数等。

### server块

这块和虚拟主机有关系，是为了降低互联网服务器硬件成本

~~~htm
server {
    listen       80;
    server_name  localhost;
    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
~~~

## 配置反向代理

实现效果一：在浏览器中访问nginx,跳到我的gitee首页

~~~text
location / {
  root   /usr/share/nginx/html;
  proxy_pass https://gitee.com/heguangchuan;
  index  index.html index.htm;
}
~~~

## 配置负载均衡

在http块中添加 upstream ，配置要负载的内容

~~~text
upstream myserver{
  server 192.168.198.1:20001;
  server 192.168.198.1:20002;
}
~~~

然后在server块中引用

~~~text
location / {
  root   /usr/share/nginx/html;
  proxy_pass http://myserver;
  index  index.html index.htm;
}
~~~

### 负载均衡分配策略

#### 轮询

每个请求按照顺序逐一分配到不同的服务器，如果服务器down掉，自动剔除，这是默认策略

#### weight

指定轮询几率，用户后端服务器性能不佳的时候

~~~text
upstream myserver{
  server 192.168.198.1:20001 weight=3;
  server 192.168.198.1:20002 weight=5;
}
~~~

### ip-hash

根据ip的哈希值访问，有效的解决session问题

~~~text
upstream myserver{
  ip_hash;
  server 192.168.198.1:20001;
  server 192.168.198.1:20002;
}
~~~

### fair

根据后端服务器的响应时间分配，响应时间越短，优先分配

~~~text
upstream myserver{
  server 192.168.198.1:20001;
  server 192.168.198.1:20002;
  fair;
}
~~~

## 配置动静分离

略

## 配置高可用集群

本次集群配置的ip分别为:

192.168.198.129   -------Master

192.168.198.130 ---------Slave

192.168.198.131 ---------Slave

192.168.198.50  ----------Vip

![image](https://gitee.com/heguangchuan/rainmeter/raw/master/img/nginx/gky.png)

上图为nginx高可用集群模式的架构图，从图上可以看出需要如下的准备工作

- 安装keepalived
- 安装nginx
- 配置虚拟ip

### 安装keepalived

在三台服务器上安装keepalived

~~~text
yum install -y keepalived
~~~

keepalived安装后，配置文件的路径为: /etc/keepalived/keepalived.conf

### 高可用配置（主从配置）

#### 修改keepalived配置文件

~~~
! Configuration File for keepalived
global_defs {
   notification_email {
     596767880@qq.com
   }
   notification_email_from 596767880@qq.com
   smtp_server 596767880@qq.com
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_pid.sh"   # 检查nginx状态的脚本
    interval 2
    weight 3
}

vrrp_instance VI_1 {
    state MASTER     #备份服务器上将MASTER改为BACKUP
    interface ens32
    virtual_router_id 51
    priority 100       #备份服务上将100改为小于100，可配置成90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.198.50    #有多个vip可在下面继续增加
    }
    track_script {
        chk_nginx
    }
}   
~~~

### 添加检测脚本

 vim /etc/keepalived/nginx_pid.sh 

~~~html
#!/bin/bash
#version 0.0.1
#
A=`ps -C nginx --no-header |wc -l`
if [ $A -eq 0 ];then
     systemctl restart docker
      sleep 3
            if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
                  systemctl stop keepalived
fi 
fi
~~~

脚本说明：当nginx进程不存在时，会自动重启docker服务，docker服务启动时会自动启动nginx容器；再次检查nginx进程，如果不存在，就停止keepalived服务，然后NGINX_BACKUP主机会自动接替NGINX_MASTER的工作 

~~~text
chmod +x /etc/keepalived/nginx_pid.sh
~~~

在浏览器地址栏输入 虚拟 **ip** **地址** **192.168.192.50**  查看即可

