# Nginx

高性能的HTTP和反向代理web服务器

占有内存少，并发能力强



### 反向代理

#### 正向代理

在客户端（浏览器）配置代理服务器，通过代理服务器进行互联网访问

#### 反向代理

客户端对代理是无感知的，是代理与服务器的的联系。

我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。



### 负载均衡

单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分法到不同服务器，也就是我们所说的负载均衡



### 动静分离

将动态资源和静态资源分到不同的服务器中，交由不同的服务器来加载，加快解析，降低原来单个服务器的压力



### 安装

```shell

yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
pcre在usr/src目录下
openssl
zlib

在/usr/src目录下安装nginx
wget -c https://nginx.org/download/nginx-1.19.0.tar.gz
tar -zxvf nginx-1.19.0.tar.gz

开放端口号
firewall-cmd --list-all

设置开放的端口号
firewall-cmd --add-service=http -permanent
sudo firewall-cmd --add-port=80/tcp --permanent

重启防火墙
firewall-cmd -reload
```



## Nginx基本操作

安装目录:

/usr/local/nginx/sbin/nginx

启动：

```shell
nginx
or
start nginx
```

查看版本：

```shell
nginx -v
```

关闭：

```shell
nginx -s stop
```

重启nginx，重新加载配置文件:

```shell
nginx -s reload
```

nginx命令更多细节

```shell
nginx -h
```

防火墙：

```shell
查看开放的端口号
firewall-cmd --list-all

设置开放的端口号
firewall-cmd --add-service=http -permanent
sudo firewall-cmd --add-port=80/tcp --permanent

重启防火墙
firewall-cmd -reload
```





## 配置文件

Path:

/usr/local/nginx/conf

将nginx.conf配置文件分为三部分：

第一部分：全局块

​	从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行到配置指令，主要包括配置运行Nginx服务器的用户（组）、允许生成的worker process数，进程PID存放路径、日志存放路径和类型以及配置文件的引入等

如

```shell
worker_process  1;    这个值越大，可以支持的并发处理量也就越多
```

第二部分：events块

events块设计的指令主要影响Nginx服务器与用户的网络连接

```shell
比如worker_connections 1024 支持的最大连接数
```

第三部分：http块

这算是Nginx服务器**配置中最频繁的部分**，代理、缓存和日志定义等绝大多数功能和第三方模块等配置都在这里。

其中还包括http全局块，server块



## Nginx配置实例-反向代理

实现效果：

打开浏览器，输入url www.123.com 显示tomcat主页面



准备工作：

安装tomcat服务器

启动tomcat

```shell
/usr/src/apache-tomcat-9.0.40/bin/startup.sh来启动
```



具体配置：

第一步 在系统中的host文件进行域名和ip对应关系的配置

mac下 : /etc/hosts

第二步 在nginx进行请求转发的配置（反向代理配置）

第三步 更改nginx的配置 

使得80端口监听地址改为ip地址



## Nginx配置实例-负载均衡

1、实现效果

（1）浏览器地址栏输入地址http://121.196.108.172/edu/a.html，负载均衡效果，平均到8080和8081端口中

2、准备工作

（1）准备两个服务器，一个为8080，一个为8081

（2）在两台tomcat里面webapps目录中，创建名称是edu文件夹，在edu中创建测试用的a.html文件，用于测试

3、在nginx的配置文件中配置负载均衡的配置

在conf文件中进行添加配置:

```shell
http{

......

	upstream myserver{

			ip_hash;
			server 121.196.108.172:8080 weight=1;
			server 121.196.108.172::8081 weight=1;
			}
......
			server{
				location / {
				proxy_pass htt://myserver;
				proxy_connect_timeout 10;
				}
			}
}
```

集中分配策略：

1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器宕机，能被自动剔除

2、weight

weight代表权重，默认为1，权重越高被分配到的客户端越多

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况

```shell
upstream server_pool{
	server localhost.1 weight=10;
	server localhost.2 weight=10;
}
```

3、ip_hash

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session共享问题

```shell
upstream server_pool{
	ip_hash
	server localhost.1:80;
	server localhost.2:80;
}
```

4、fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```shell
upstream server_pool{
	server localhost.1:80;
	server localhost.2:80;
	fair
}
```



## 动静分离

简单来说就是把动态跟静态请求分开，可以理解为用nginx处理静态页面，使用tomacat处理动态页面。

![image-20201216193656768](/Users/didi/Documents/Nginx.assets/image-20201216193656768.png)

两种方式：

- 一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是主流推崇的方案。
- 另外一种就是动态跟静态文件混合在一起发布，通过nginx来分开

通过配置文件中的 location 指定不同的后缀名实现不同的请求转发。通过使用expires参数设置，可以使浏览器缓存过期时间，减少与服务器之间的请求和流量。



首先在linux系统中准备静态资源，用于进行访问

html位于 /data/www/ 中

image位于 /data/image/中



然后在具体配置config

```shell
在server{}中

location /www/{
	root /data/ ;
	index index.html index.htm;
}

location /image/{
	root /data/ ;
	autoindex on;
}
```

然后访问对应的地址

 http://ip.com/



## 高可用的集群

<img src="/Users/didi/Documents/Nginx.assets/image-20201217171059184.png" alt="image-20201217171059184" style="zoom:67%;" />

（1）需要两台nginx服务器

（2）需要keepalived

安装：

yum install keepalived -y

检查：

rpm -q -a keepalived

启动：

systemctl start keepalived.service 

（3）需要虚拟ip



在 /etc/keepalived/中

修改 keepalived.conf 中修改配置文件来配置高可用的集群

```shell
global_defs {
	notification_email {
		acassen@firewall.loc
		failover@firewall.loc
		sysadmin@firewall.loc
	}
	notification_email_from Alexandre.Cassen@firewall.loc
	smtp_server 192.168.17.129
	smtp_connect_timeout 30
	router_id LVS_DEVEL
}

vrrp_script chk_http_port{
	script "/usr/local/src/nginx/check_sh"
	interval 2       #(检测脚本执行的间隔)
	weight 2 
}

vrrp_instance VI_1 {
	state MASTER # 备份服务器上将 MASTER 改为 BACKUP
	interface ens33 //网卡 在linux中用ifconfig命令查看
	virtual_router_id 51 #主、备机的virtual_router_id必须相同
	priority 100 #主、备机取不同的优先级，主机值较大，备份机值较小
	advert_int 
	authentication {
			auth_type PASS
			auth_path 1111
	}
	virtual_ipaddress {
		192.168.17.50		//VRRP H虚拟地址
	}
}
```

在/usr/local/src 中添加检测脚本

```shell
#!/bin/bash
A=`ps -C nginx -no-header |wc -l`
if [ $A -eq 0 ];then
	/usr/local/nginx/sbin/nginx
	sleep 2
	if[ `ps -C nginx --no-header |wc -l` -eq 0 ];then
		killall keepalived
	fi
fi
```

将nginx和keepalived启用即可



## 原理

master-workers

一个master和多个worker

1. 可以使用nginx -s reload 热部署，利用nginx进行热部署操作
2. 每个worker是独立的进程，如果有其中一个worker出现问题，其他worker是独立的，可以继续进行争抢，实现请求过程，不会造成服务中断。

设置多少个worker合适？

每个worker的线程可以把一个cpu的性能发挥到极致。所以**worker数和服务器的cpu数相等是最合适的**，否则少了会浪费cpu，多了会造成cpu频繁切换上下文带来多的损耗。

连接数worker_connection？

发送一个请求，占用了worker的几个连接数？

2个或者4个

nginx有一个master，四个worker，每个worker支持最大的连接数据是1024，支持的最大并发数是多少？

最大连接数4*1024=4096，最大并发数为4096/2或4096/4