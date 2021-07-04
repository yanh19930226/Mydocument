###  **安装** 

mysql8

    mkdir -p /usr/local/mysql/conf/conf.d
    mkdir -p /usr/local/mysql/data
    mkdir -p /usr/local/mysql/logs

    docker run -d --name=mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
    docker cp mysql:/etc/mysql/my.cnf /usr/local/mysql/conf/my.cnf
    docker cp mysql:/etc/mysql/conf.d /usr/local/mysql/conf/
    
    docker rm -f mysql -v
    
    docker run -d --name=mysql --restart=always \
    -v /usr/local/mysql/data:/var/lib/mysql \
    -v /usr/local/mysql/conf/my.cnf:/etc/mysql/my.cnf \
    -v /usr/local/mysql/conf/conf.d:/etc/mysql/conf.d  \
    -v /usr/local/mysql/logs:/logs --privileged=true -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql --lower_case_table_names=1

如需指定容器日志大小，可以加上参数

    --log-opt max-size=10m --log-opt max-file=3

mysql 5.7

    docker run --name mysql --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
    docker cp mysql:/etc/mysql/conf.d /usr/local/mysql/conf
    docker rm -f mysql
    docker run --name mysql --restart=always -p 3306:3306 \
    -v /usr/local/mysql/conf/conf.d:/ect/mysql/conf.d \
    -v /usr/local/mysql/logs:/logs \
    -v /usr/local/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
### 修改密码认证方式

~~~
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges;
cd /usr/local/mysql/conf/conf.d
vim mysql.cnf
[mysqld]
character-set-server=utf8
default_authentication_plugin=mysql_native_password
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
~~~

## mysql 主从复制

    主库：172.20.39.58
    从库：172.20.39.63

### 修改主数据库的配置文件

    cd /usr/local/mysql/conf
    vim my.cnf
    
    [mysqld]    
    
    ### 这是数据库ID,此ID是唯一的，ID值不能重复，否则会同步出错；
    
    server-id = 1  
    
    ####二进制日志文件，此项为必填项，否则不能同步数据；
    
    log-bin = mysql-bin 
    
    ## 需要同步的数据库，如果还需要同步另外的数据库，那么继续逐条添加，如果不写，那么默认同步所有的数据库；               
    
    binlog-do-db = testcreate  
    
    #不需要同步的数据库；
                               
    binlog-ignore-db = mysql 
    
    修改完成之后重启mysql服务。
### 添加主数据库用于同步的账号

 mysql8以前

    GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'172.20.39.63' IDENTIFIED BY 'slave1';

 mysql8

    CREATE USER 'slave1'@'172.20.39.63' IDENTIFIED WITH mysql_native_password BY 'slave1';
    GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'172.20.39.63';
    flush privileges;

### 刷新权限并找到File 和 Position 的值记录下来

    flush privileges;
    show master status; 

### 修改从库的配置文件

同上  server-id 不能相同，并重启服务，在从库执行 

    CHANGE MASTER TO MASTER_HOST='172.20.39.58',MASTER_USER='slave1',MASTER_PASSWORD='slave1',     
    MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=1345; 
    
    master_host= 这里填你主库的IP。 
    master_user=’slave1’ 刚才我们创建的那个用户。 
    master_user=’slave1’ ..不解释。 
    这就是我们刚才 在主库里面 show master status；得到的值了,自行根据实际情况填写 
    master_log_file=’mysql-bin.000001’ 
    master_log_pos=423 
    如果你的主库还有是其他端口的话， 
    master_port=端口号 

从库 执行 stop slave ；再执行 start slave; 

从库查看 ：show slave status 

### mysql创建只读账户

 mysql8以前

    GRANT SElECT ON *.* TO 'username'@'%' IDENTIFIED BY "password";
    flush privileges;

 mysql8

    create user 'slave1'@'%' identified by 'slave1';
    flush privileges;
    grant select on *.* to 'slave1'@'%' with grant option;
    flush privileges;


​    
