# redis docker

https://hub.docker.com/_/redis

 **start with persistent storage** 

    docker run --name some-redis -d redis redis-server --appendonly yes

 **Additionally, If you want to use your own redis.conf ...** 

    docker run --name redis -v /myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf \
    -d redis redis-server /usr/local/etc/redis/redis.conf

 **其他目录挂载** 

    mkdir -p /usr/local/redis/data
    mkdir -p /usr/local/redis/conf

    docker run --name redis --restart=always -p 6379:6379 \
    -v /usr/local/redis/data:/data \
    -v /usr/local/redis/conf/redis.conf:/etc/redis/redis.conf \
    -d redis redis-server /etc/redis/redis.conf --appendonly yes


此时redis的容器已经创建 但是还没有配置文件,将下面的配置文件复制到宿主机的/usr/local/redis/conf下即可,redis.conf 中daemonize=NO。非后台模式，如果为YES 会的导致 redis 无法启动，因为后台会导致docker无任务可做而退出

https://gitee.com/heguangchuan/rainmeter/blob/master/document/redis/redis.conf
   