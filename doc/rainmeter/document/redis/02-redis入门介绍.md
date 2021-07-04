### redis入门介绍

 **Redis的安装** 
 
 **docker**安装redis

https://gitee.com/heguangchuan/rainmeter/blob/master/document/redis/09-redis-docker.md

安装包方式

    cd /opt
    tar -zxvf redis-5.0.5.tar.gz
    mv redis-5.0.5 /usr/local/
    cd /usr/local/redis-5.0.5/
    make
    make install

 **查看默认安装目录：usr/local/bin** 

- redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何
- redis-check-aof：修复有问题的AOF文件
- redis-check-dump：修复有问题的dump.rdb文件
- redis-cli：客户端，操作入口
- redis-sentinel：redis集群使用
- redis-server：Redis服务器启动命令

 **Redis启动** 

修改redis.conf文件将里面的daemonize no 改成daemonize yes，让服务在后台启动

    cd /usr/local/redis-5.0.5/
    cp redis.conf redis.conf.bak
    vim redis.conf

启动redis

    cd /usr/local/bin/
    redis-server /usr/local/redis-5.0.5/redis.conf

    ##客户端连接 ##
    redis-cli -p 6379

 **停止redis** 

    redis-cli -p 6379 shutdown


redis 是C语言写得，官方提供的数据为 100000+ 的QPS, redis是基于内存操作的，CPU不是redis的性能瓶颈，redis的性能瓶颈是网络带宽和内存。

**Redis** **为什么单线程还这么快**

redis 是将所有的数据全部放在内存中的，所以说使用单线程去操作效率就是最高的，多线程 （CPU上下文会切换：耗时的操作！！！），对于内存系统来说，如果没有上下文切换效率就是最高 的！多次读写都是在一个CPU上的，在内存情况下，这个就是最佳的方案。