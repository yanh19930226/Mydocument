## **什么是fastDFS**

FastDFS是用c语言编写的一款开源的分布式文件系统，它是由淘宝资深架构师余庆编写并开源。FastDFS专为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。为什么要使用fastDFS呢？上边介绍的NFS、GFS都是通用的分布式文件系统，通用的分布式文件系统的优点的是开发体验好，但是系统复杂性高、性能一般，而专用的分布式文件系统虽然开发体验性差，但是系统复杂性低并且性能高。fastDFS非常适合存储图片等那些小文件，fastDFS不对文件进行分块，所以它就没有分块合并的开销，fastDFS网络通信采用socket，通信速度很快

## fastDSF工作原理

FastDFS架构包括 Tracker server和Storageserver。客户端请求Tracker server进行文件上传、下载，通过Tracker
server调度最终由Storage server完成文件上传和下载

   ![image](https://gitee.com/heguangchuan/rainmeter/raw/master/img/fastdfs/yuanli.png)

### Tracker

Tracker Server作用是负载均衡和调度，通过Tracker server在文件上传时可以根据一些策略找到Storage server提供文件上传服务。可以将tracker称为追踪服务器或调度服务器。FastDFS集群中的Tracker server可以有多台，Tracker server之间是相互平等关系同时提供服务，Tracker server不存在单点故障。客户端请求Tracker server采用轮询方式，如果请求的tracker无法提供服务则换另一个tracker。

### Storage

Storage Server作用是文件存储，客户端上传的文件最终存储在Storage服务器上，Storage server没有实现自己的文件系统而是使用操作系统的文件系统来管理文件。可以将storage称为存储服务器。Storage集群采用了分组存储方式。storage集群由一个或多个组构成，集群存储总容量为集群中所有组的存储容量之和。一个组由一台或多台存储服务器组成，组内的Storage server之间是平等关系，不同组的Storage server之间不会相互通信，同组内的Storage server之间会相互连接进行文件同步，从而保证同组内每个storage上的文件完全一致的。一个组的存储容量为该组内存储服务器容量最小的那个，由此可见组内存储服务器的软硬件配置最好是一致的。

采用分组存储方式的好处是灵活、可控性较强。比如上传文件时，可以由客户端直接指定上传到的组也可以由tracker进行调度选择。一个分组的存储服务器访问压力较大时，可以在该组增加存储服务器来扩充服务能力（纵向扩容）。当系统容量不足时，可以增加组来扩充存储容量（横向扩容）。

Storage状态收集
Storage server会连接集群中所有的Tracker server，定时向他们报告自己的状态，包括磁盘剩余空间、文件同步
状况、文件上传下载次数等统计信息。

**文件上传流程**

   ![image](https://gitee.com/heguangchuan/rainmeter/raw/master/img/fastdfs/liucheng.png)

客户端上传文件后存储服务器将文件ID返回给客户端，此文件ID用于以后访问该文件的索引信息。文件索引信息包括：组名，虚拟磁盘路径，数据两级目录，文件名

**文件下载流程**

   ![image](https://gitee.com/heguangchuan/rainmeter/raw/master/img/fastdfs/liuchengx.png)

tracker根据请求的文件路径即文件ID 来快速定义文件

# fastDFS安装

tracker和storage使用相同的安装包，fastDFS的下载地址在：https://github.com/happyfish100/FastDFS

## FastDFS--tracker安装

### 必须环境

    yum install gcc-c++
    yum -y install libevent

### 安装libfastcommon

    cd /usr/local
    tar -zxvf libfastcommonV1.0.7.tar.gz
    cd libfastcommon-1.0.7
    ./make.sh
    ./make.sh install

注意：libfastcommon安装好后会自动将库文件拷贝至/usr/lib64下，由于FastDFS程序引用usr/lib目录所以需要将/usr/lib64下的库文件拷贝至/usr/lib下

~~~
cp /usr/lib64/libfastcommon.so /usr/lib
~~~

## tracker编译安装

    将FastDFS_v5.05.tar.gz拷贝至/usr/local/下
    tar -zxvf FastDFS_v5.05.tar.gz
    cd FastDFS
    ./make.sh
    ./make.sh install
    
    安装成功将安装目录下的conf下的文件拷贝到/etc/fdfs/下。
    cd /usr/local/FastDFS/conf
    cp client.conf /etc/fdfs
    cp http.conf /etc/fdfs
    cp mime.types /etc/fdfs/
    cp storage.conf /etc/fdfs/
    cp storage_ids.conf /etc/fdfs/
    cp tracker.conf /etc/fdfs/
    
    安装成功后进入/etc/fdfs目录：
    cd /etc/fdfs
    
    修改tracker.conf
    
    vim tracker.conf
    base_path=/home/yuqing/FastDFS   改为：base_path=/home/FastDFS
    配置http端口：
    http.server_port=80
    
    启动 tracker
    /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
    查看状态
    ps aux | grep tracker

### 安装storage

如果不是在同一台机子的情况下 需要重复安装上面tracker的过程

    修改storage.conf
    cd /etc/fdfs
    
    vi storage.conf
    group_name=group1
    base_path=/home/yuqing/FastDFS改为：base_path=/home/FastDFS
    store_path0=/home/yuqing/FastDFS改为：store_path0=/home/FastDFS/fdfs_storage
    #如果有多个挂载磁盘则定义多个store_path，如下
    #store_path1=.....
    #store_path2=......
    tracker_server=192.168.101.3:22122   #配置tracker服务器:IP
    #如果有多个则配置多个tracker
    tracker_server=192.168.101.4:22122
    
    #配置http端口
    http.server_port=80
    
    启动 storage
    /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
    查看状态
    ps aux | grep storage

### 开机自启动

    在centos7中, /etc/rc.d/rc.local 文件的权限被降低了，需要给rc.local 文件增加可执行的权限；
    chmod +x /etc/rc.d/rc.local
    
    vim /etc/rc.d/rc.local  
    增加如下
    # fastdfs start
    /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
    /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart

# 整合nginx 

##  安装 FastDFS-nginx-module

    cd /opt
    tar -zxvf fastdfs-nginx-module_v1.16.tar.gz
    mv fastdfs-nginx-module /usr/local/
    cd /usr/local/
    cd fastdfs-nginx-module/src
    
    修改config文件将/usr/local/路径改为/usr/
    vim config    
    cp mod_fastdfs.conf /etc/fdfs/
    cd /etc/fdfs/
    修改刚刚复制的mod_fastdfs.conf配置文件
    
    vi /etc/fdfs/mod_fastdfs.conf
    修改内容如下:
    base_path=/home/FastDFS
    tracker_server=ip:22122    #可以配置多个
    url_have_group_name=true		#url中包含group名称
    store_path0=/home/FastDFS/fdfs_storage   #指定文件存储路径
    
    将libfdfsclient.so拷贝至/usr/lib下
    cp /usr/lib64/libfdfsclient.so /usr/lib/
    
    创建nginx/client目录
    mkdir -p /var/temp/nginx/client

## 安装nginx时添加fastdfs-nginx-module模块 

    ./configure \
    --prefix=/usr/local/nginx \
    --pid-path=/var/run/nginx/nginx.pid \
    --lock-path=/var/lock/nginx.lock \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --with-http_gzip_static_module \
    --http-client-body-temp-path=/var/temp/nginx/client \
    --http-proxy-temp-path=/var/temp/nginx/proxy \
    --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
    --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
    --http-scgi-temp-path=/var/temp/nginx/scgi \
    --add-module=/usr/local/fastdfs-nginx-module/src

##  nginx配置文件添加节点

    location /group1/M00/{
    	root /home/FastDFS/fdfs_storage/data;
    	ngx_fastdfs_module;
    }

##  docker fastdfs

假设服务器ip 192.168.1.221

    mkdir -p /usr/local/fastdfs/conf
    
    docker run -ti -d --name tracker \
    -v /usr/local/fastdfs/tracker_data/:/fastdfs/tracker/data --net=host season/fastdfs tracker
    
    docker cp tracker:/fdfs_conf/. /usr/local/fastdfs/conf/
    
    docker run -tid --name storage --restart=always \
    -v /usr/local/fastdfs/conf:/fdfs_conf \
    -v /usr/local/fastdfs/storage_data:/fastdfs/storage/data \
    -v /usr/local/fastdfs/store_path:/fastdfs/store_path \
    --net=host -e TRACKER_SERVER:192.168.1.221:22122 season/fastdfs storage
    
    docker rm -f tracker 
    
    docker run -ti -d --name tracker --restart=always \
    -v /usr/local/fastdfs/conf:/fdfs_conf \
    -v /usr/local/fastdfs/tracker_data/:/fastdfs/tracker/data --net=host season/fastdfs tracker

##  docker fastdfs nginx

### 安装 fastdfs  参照上面的安装方式

    修改配置文件
    cd /usr/local/fastdfs/conf/
    
    vim tracker.conf
    修改端口
    http.server_port=80
    
    vim storage.conf
    修改端口  文件存储路径  及tracker的服务器地址
    http.server_port=80
    store_path0=/fastdfs/store_path
    tracker_server=192.168.1.221

### 安装nginx 并挂载文件目录

    mkdir /usr/local/nginx/conf
    
    docker run --name tmp-nginx-container -d nginx
    docker cp tmp-nginx-container:/etc/nginx/nginx.conf /usr/local/nginx/conf/nginx.conf
    docker cp tmp-nginx-container:/usr/share/nginx/html /usr/local/nginx/
    docker cp tmp-nginx-container:/var/log/nginx /usr/local/nginx/logs
    docker cp tmp-nginx-container:/etc/nginx/conf.d /usr/local/nginx/conf/
    
    docker rm -f tmp-nginx-container
    
    下面是一整行 
    docker run --name nginx --restart=always -p 80:80 \
    -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v /usr/local/nginx/html:/usr/share/nginx/html \
    -v /usr/local/nginx/conf/conf.d:/etc/nginx/conf.d \
    -v /usr/local/nginx/logs:/var/log/nginx \
    -v /usr/local/fastdfs/store_path:/usr/share/nginx/html/store_path -d nginx
    
    修改nginx配置文件
    cd /usr/local/nginx/conf/conf.d/
    
    在server中添加映射
    
    location /group1/M00/ {
        proxy_pass http://192.168.1.221/store_path/data/;
    }


####################################################################################

##  **springboot 与 fastdfs** 

### 引入jar


    <!-- https://mvnrepository.com/artifact/com.github.tobato/fastdfs-client -->
    <dependency>
        <groupId>com.github.tobato</groupId>
        <artifactId>fastdfs-client</artifactId>
        <version>1.26.6</version>
    </dependency>

### 在yml文件中进行配置

    fdfs:
      soTimeout: 1500
      connectTimeout: 600
      pool:
        max-total: 153
        max-wait-millis: 102
      thumbImage:
        width: 150
        height: 150
      trackerList:
        - 112.74.107.0:22122
      web-server-url: http://112.74.107.0/

### 通过配置解决jmx重复注册bean的问题
~~~java
    import com.github.tobato.fastdfs.FdfsClientConfig;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.EnableMBeanExport;
    import org.springframework.context.annotation.Import;
    import org.springframework.jmx.support.RegistrationPolicy;
    @Configuration
    @Import(FdfsClientConfig.class)
    // 解决jmx重复注册bean的问题
    @EnableMBeanExport(registration = RegistrationPolicy.IGNORE_EXISTING)
    public class FastDFSConfig {
    }
~~~
### IOC容器中注入FastDFSClient

    package com.ruoyi.fastdfs;
    import com.github.tobato.fastdfs.domain.conn.FdfsWebServer;
    import com.github.tobato.fastdfs.domain.fdfs.StorePath;
    import com.github.tobato.fastdfs.domain.proto.storage.DownloadByteArray;
    import com.github.tobato.fastdfs.service.FastFileStorageClient;
    import org.apache.commons.io.FilenameUtils;
    import org.apache.commons.lang3.StringUtils;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;
    import org.springframework.web.multipart.MultipartFile;
    import java.io.*;
    
    @Component
    public class FastDFSClient {
        private static Logger log = LoggerFactory.getLogger(FastDFSClient.class);
        @Autowired
        private FastFileStorageClient fastFileStorageClient;
        @Autowired
        private FdfsWebServer fdfsWebServer;
    
        /**
         * @param multipartFile 文件对象
         * @return 返回文件地址
         * @author qbanxiaoli
         * @description 上传文件
         */
        public String uploadFile(MultipartFile multipartFile) {
            try {
                StorePath storePath = fastFileStorageClient
                        .uploadFile(multipartFile.getInputStream()
                                , multipartFile.getSize()
                                , FilenameUtils.getExtension(multipartFile.getOriginalFilename())
                                , null);
                return storePath.getFullPath();
            } catch (IOException e) {
                log.error(e.getMessage());
                return null;
            }
        }
    
        /**
         * @param multipartFile 图片对象
         * @return 返回图片地址
         * @author qbanxiaoli
         * @description 上传缩略图
         */
        public String uploadImageAndCrtThumbImage(MultipartFile multipartFile) {
            try {
                StorePath storePath = fastFileStorageClient
                        .uploadImageAndCrtThumbImage(multipartFile.getInputStream()
                                , multipartFile.getSize()
                                , FilenameUtils.getExtension(multipartFile.getOriginalFilename())
                                , null);
                return storePath.getFullPath();
            } catch (Exception e) {
                log.error(e.getMessage());
                return null;
            }
        }
    
        /**
         * @param file 文件对象
         * @return 返回文件地址
         * @author qbanxiaoli
         * @description 上传文件
         */
        public String uploadFile(File file) {
            try {
                FileInputStream inputStream = new FileInputStream(file);
                StorePath storePath = fastFileStorageClient
                        .uploadFile(inputStream
                                , file.length()
                                , FilenameUtils.getExtension(file.getName())
                                , null);
                log.info("storePath:" + storePath.getFullPath());
                return storePath.getFullPath();
            } catch (Exception e) {
                log.error(e.getMessage());
                return null;
            }
        }
    
        /**
         * @param file 图片对象
         * @return 返回图片地址
         * @author qbanxiaoli
         * @description 上传缩略图
         */
        public String uploadImageAndCrtThumbImage(File file) {
            try {
                FileInputStream inputStream = new FileInputStream(file);
                StorePath storePath = fastFileStorageClient
                        .uploadImageAndCrtThumbImage(inputStream
                                , file.length()
                                , FilenameUtils.getExtension(file.getName())
                                , null);
                return storePath.getFullPath();
            } catch (Exception e) {
                log.error(e.getMessage());
                return null;
            }
        }
    
        /**
         * @param bytes         byte数组
         * @param fileExtension 文件扩展名
         * @return 返回文件地址
         * @author qbanxiaoli
         * @description 将byte数组生成一个文件上传
         */
        public String uploadFile(byte[] bytes, String fileExtension) {
            ByteArrayInputStream stream = new ByteArrayInputStream(bytes);
            StorePath storePath = fastFileStorageClient
                      .uploadFile(stream, bytes.length, fileExtension, null);
            return storePath.getFullPath();
        }
    
        /**
         * @param fileUrl 文件访问地址
         * @param file    文件保存路径
         * @author qbanxiaoli
         * @description 下载文件
         */
        public boolean downloadFile(String fileUrl, File file) {
            try {
                StorePath storePath = StorePath.parseFromUrl(fileUrl);
                byte[] bytes = fastFileStorageClient
                        .downloadFile(storePath.getGroup()
                                , storePath.getPath()
                                , new DownloadByteArray());
                FileOutputStream stream = new FileOutputStream(file);
                stream.write(bytes);
            } catch (Exception e) {
                log.error(e.getMessage());
                return false;
            }
            return true;
        }
    
        /**
         * @param fileUrl 文件访问地址
         * @author qbanxiaoli
         * @description 删除文件
         */
        public boolean deleteFile(String fileUrl) {
            if (StringUtils.isEmpty(fileUrl)) {
                return false;
            }
            try {
                StorePath storePath = StorePath.parseFromUrl(fileUrl);
                fastFileStorageClient.deleteFile(storePath.getGroup(), storePath.getPath());
            } catch (Exception e) {
                log.error(e.getMessage());
                return false;
            }
            return true;
        }
    
        // 封装文件完整URL地址
        public String getResAccessUrl(String path) {
            String url = fdfsWebServer.getWebServerUrl() + path;
            return url;
        }
    }
