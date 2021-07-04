# zookeeper

### 应用场景

是一个经典的分布式数据一致性解决方案，致力于为分布式应用提供一个高性能、高可用、且具有严格顺序访问控制能力的分布式协调存储服务。

- 维护配置信息
- 分布式锁服务
- 集群管理
- 生成分布式唯一ID

#### 1.维护配置信息

​	java中经常遇到配置项，如数据库的url、schema、user和password等。通常这些配置项我们会放置在配置文件中，再将配置文件放置在服务器上当需要更改配置时，就需要到对应的服务器上去修改。但随着分布式的兴起，很多服务器都需要配置文件，为保证配置服务的高可用性和每台服务器配置数据的一致性，就需要一种服务能够高效快速且可靠的完成配置项的更改等操作。而zookeeper就可以提供这样一种服务。

#### 2.分布式锁服务

​	一个集群是一个分布式系统，由多台服务器组成，为了提高并发度和可靠性，多台服务器上运行着同一种服务。当多个服务再运行时就需要协调各服务的进度，有时候需要保证当某个服务再进行某个操作时，其他的服务都不能进行该操作，即对该操作进行加锁，如果当前机器挂掉后，释放锁并fail over到其他的机器继续执行该服务。

#### 3.集群管理

​	一个集群有时会因为各种软硬件故障或者网络故障，出现某些服务器挂掉而被移除集群，而某些服务器加入到集群中的情况，zookeeper会将这些服务器加入/移除的情况通知给集群中的其他正常工作的服务器，以及时调整存储和计算等任务的分配和执行等，此外zookeeper还会对故障的服务器做出诊断并尝试修复。

#### 4.生成分布式唯一ID

​	在过去的单库单表型系统中，通常可以使用数据库自带的auto_increment属性来自动为每条记录生成一个唯一ID。但分库分表后，就无法依靠数据库的auto_increment属性来唯一标识一条记录了。此时我们就可以用zookeeper在分布式环境下生成全局唯一ID。做法：每次要生成一个新ID时，创建一个持久顺序节点，创建操作返回的节点序号，即为新ID，然后把比自己小的节点删除即可。



### 设计目标

​	致力于为分布式应用提供一个高性能、高可用，且具有严格顺序访问控制能力的分布式协调服务

#### 1.高性能

​	zookeeper将全量数据存储在内存中，并直接服务于客户端的所有非事物请求，尤其适用于以读为主的应用场景

#### 2.高可用

​	zookeeper一般以集群的方式对外提供服务，一般3～5台机器就可以组成一个可用的zookeeper集群了，每台机器都会在内存中维护当前的服务器状态，并且每台机器之间，都相互保持着通信。只要集群中超过一半的机器都能够正常工作，那么整个集群就能够正常对外服务。

#### 3.严格顺序访问

​	对于来自客户端的每个更新请求，zookeeper都会分配一个全局唯一的递增编号，这个编号反映了所有事物操作的先后顺序。

### 数据模型

​	zookeeper的数据节点可以视为树状结构（或者目录），树中的各节点被称为znode，一个znode可以有多个字节点。zookeeper节点在结构上表现为树状；使用路径path来定位某个znode，比如/ns-1/itcast/mysql/schema1/table1，此处ns-1、itcast、mysql、schema1、table1分别是根节点、2级节点、3级节点以及4级节点；并且每一个节点都是后一节点的父节点。

​	znode，兼具文件和目录两种特点。既像文件一样维护着数据、元信息、ACL、时间戳等数据结构，又像目录一样可以作为路径标识等一部分。

​	描述一个znode大体上分为3个部分：

- 节点的数据：即znode data（节点path，节点data）的关系就像是java map中（key，value）的关系
- 节点的字节点children
- 节点的状态stat：用来描述当前节点的创建、修改记录。包括cZxid、ctime等

##### **节点状态stat的属性**

在zookeeper shell中使用get命令来查看指定路径节点的data、stat信息

**属性说明**：

- cZxid：数据节点创建时的事务ID
- ctime：数据节点创建时的时间
- mZxid：数据节点最后一次更新时的事务ID
- mtime：数据节点最后一次更新的时间
- pZxid：数据节点的字节点最后一次被修改的事务ID
- cversion：字节点的更改次数
- dataVersion：节点数据的更改次数
- aclVersion：节点的ACL的更改次数，与权限控制相关
- ephemeralOwner：如果节点时临时节点，则表示创建该节点的会话的SessionID；如果节点时持久节点，则该属性值为0
- dataLength：数据内容的长度
- numChildren：数据节点当前的字节点个数

##### 节点类型

两种，分别为**临时节点**和**永久节点**。节点的类型在创建时就确定，不能改变。

- 临时节点：该节点的生命周期依赖于创建它们的会话。一旦会话（session）结束，该节点也随之被删除，当然也可以手动删除。虽然每个临时的Znode都会绑定到一个客户端会话，但他们对所有的客户端还是可见的。另外临时节点不允许拥有字节点。
- 永久节点：该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，他们才能被删除。



### 安装

安装下载

准备配置文件：

```shell
//进入conf目录
cd /usr/local/apache-zookeeper-3.6.2/conf
//复制配置文件
cp zoo_sample.cfg zoo.cfg
//zookeeper根目录下新建data目录
mkdir data
//vi 修改配置文件中的dataDir 
//此路径用于存储zookeeper中数据的内存快照、及事务日志文件
dataDir=/usr/local/apache-zookeeper-3.6.2/data
```

启动zookeeper

```shell
//进入zookeeper的bin目录
cd /usr/local/apache-zookeeper-3.6.2-bin/bin
//启动zookeeper
./zkServer.sh start

//启动:./zkServer.sh start
//停止:./zkServer.sh stop
//查看状态:./zkServer.sh status

连接客户端：
./zkCli.sh

退出会话：
quit

查看节点信息
ls -s /节点名称
get -s path
```



### zookeeper常用命令

#### 新增节点

```shell
create [-s] [-e] path data #其中-s为有序节点，-e临时节点
```

创建持久化节点并写入数据：

```shell
create /hadoop "123456"
```

创建持久化有序节点，此时创建的节点名为指定节点名+自增序号

```shell
//进入zookeeper shell后
create -s /a "aaa"
create -s /b "bbb"
create -s /c "ccc"
```

创建临时节点，临时节点会在会话过期后被删除：

```shell
create -e /tmp "tmp"
```

创建临时有序节点，临时节点再会话结束后会被删除

```shell
create -e -s /aa "aa"
```



#### 修改节点

更新节点的命令是set，可以直接进行修改：

```shell
set /hadoop "345"

每次对节点的修改都会使之版本号+1

也可以基于版本号进行更改，此时类似于乐观锁机制，当你传入的数据版本号(dataVersion)和当前节点的数据版本号不符合时，zookeeper会拒绝本次修改
set -v 版本号 path data
```



#### 删除节点

Delete [-v version] path 

deleteall path 删除该节点以及该节点的子节点



#### 监听器get [-w] path 

使用 get -w path注册的监听器能够在节点内容发生改变的时候，向客户端发出通知，但这样设置的监听器时一次性的，被**触发了一次以后就失效**。

#### 监听器stat [-w] path

使用stat [-w] path 注册的监听器能够在节点状态发生改变的时候，向客户端发出通知

#### 监听器ls [-w] path

使用stat [-w] path 注册的监听器能够在节点状态以及子节点状态发生改变的时候，向客户端发出通知

==监听器都是一次性的==



### zookeeper的acl权限控制

zookeeper类似文件系统，client可以创建节点、更新节点、删除节点，那么如何做权限的控制？

zookeeper的 **access control list** 访问控制可以做到这一点。

acl权限控制，使用scheme： id：permission 来标示，主要涵盖3个方面：

- 权限模式（scheme）：授权的策略
- 授权对象（id）：授权的对象
- 权限（permission）：授予的权限

其特性如下：

- zookeeper的权限控制时基于每个znode节点的，需要对每个节点设置权限
- 每个znode支持设置多种权限控制方案和多个权限
- 子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点

例如：

```shell
setAcl /test2 ip:192.168.0.1:crwda --将节点权限设置为Ip：192.168.0.1的客户端可以对节点进行增删改查管理权限
```

#### 授权模式

| 方案   | 描述                                                  |
| ------ | ----------------------------------------------------- |
| world  | 只有一个用户：anyone，代表登陆zookeeper所有人（默认） |
| ip     | 对客户端使用IP地址认证                                |
| auth   | 使用已添加认证的用户认证                              |
| digest | 使用"用户名：密码"方式认真                            |

#### 授权的对象

给谁授予权限

授权对象ID是指，权限赋予的实体，例如：IP地址或者用户

#### 授予的权限

| 权限   | ACL简写 | 描述                             |
| ------ | ------- | -------------------------------- |
| create | c       | 可以创建子节点                   |
| delete | d       | 可以删除子节点(仅下一级节点)     |
| read   | r       | 可以读取节点数据及显示子节点列表 |
| write  | w       | 可以设置节点数据                 |
| admin  | a       | 可以设置节点访问控制列表权限     |

#### 授权命令

| 命令    | 使用方式                               | 描述         |
| ------- | -------------------------------------- | ------------ |
| getAcl  | getAcl [-s] path                       | 读取ACL权限  |
| setAcl  | setAcl [-s] [-v version] [-R] path acl | 设置ACL权限  |
| addauth | addauth scheme auth                    | 添加认证用户 |



#### acl超级管理员

zookeeper的权限管理模式有一种叫做super，该模式提供一个超管可以方便的访问任何权限的节点

假设这个超管是：super:admin，需要先为超管生成密码的密文

```shell
echo -n super:admin | openssl dgst -binary -sha1 | openssl base64
```

然后打开zookeeper目录下的/bin/zkServer.sh服务器脚本文件，找到如下一行：

```shell
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=
${ZOO_LOG$_PROP}"
```

这就是脚本启动zookeeper的命令，默认只有以上两个配置项，我们需要添加一个超管的配置项

```java
"-Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxlMiHGwaqBvst5y6rkB6HQs="
```

那么修改以后这条完整命令变成了

```shell
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=
${ZOO_LOG$_PROP}" "-Dzookeeper.DigestAuthenticationProvider.superDigest=super:密码密文="\
	-cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&! < /dev/null&
```

修改之后需要重启zookeeper，之后启动zookeeper，输入如下命令添加权限

```shell
addauth digest super:admin #添加认证用户
```



### zookeeper javaAPI

 znode 是 zookeeper集合的核心组件，zookeeper API提供了一小组方法使用zookeeper集合来操作znode的所有细节。

客户端应该遵循步骤，与zookeeper服务器进行清晰和干净的交互。

- 连接到zookeeper服务器。zookeeper服务器为客户端分配会话ID
- 定期向服务器发送心跳，否则，zookeeper服务器将过去会话ID，客户端需要重新连接。
- 只要会话ID处于活动状态，就可以获取/设置znode。
- 所有任务完成后，断开与zookeeper服务器的连接。如果客户端长时间不活动，则zookeeper服务器将自动断开客户端

#### 连接到zookeeper

```java
    public static void main(String[] args) {
        try{
            //计数器对象
            final CountDownLatch countDownLatch = new CountDownLatch(1);
            //param1 服务器的ip和端口
            //param2 客户端和服务器之间的会话超市时间 以毫秒为单位
            //param3 监视器对象
            org.apache.zookeeper.ZooKeeper zooKeeper = new org.apache.zookeeper.ZooKeeper("121.196.108.172:2181", 5000, new Watcher() {
                public void process(WatchedEvent watchedEvent) {
                    if(watchedEvent.getState() == Event.KeeperState.SyncConnected){
                        System.out.println("连接创建成功！");
                        countDownLatch.countDown();
                    }
                }
            });
            //主线程阻塞等待连接对象的创建成功
            countDownLatch.await();
            //打印会话编号
            System.out.println(zooKeeper.getSessionId());
            zooKeeper.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```



#### 新增节点

```java
//同步方法
create(String path,byte[] data,List<ACL> acl,CreateMode createMode)
//异步方法
create(String path,byte[] data,List<ACL> acl,CreateMode createMode,
      AsyncCallback.StringCallback callBack,Object ctx)
```

- path - znode路径。例如：/node1   或/node1/node11
- data - 要存储在指定znode路径中的数据
- acl - 要创建的节点的访问控制列表。zookeeper API提供了一个静态接口ZooDefs.lds 来获取一些基本的acl列表。 例如： ZooDefs.lds.OPEN_ACL_UNSAFE返回打开znode的acl列表。
- createMode - 节点的类型，这是一个枚举。
- callBack - 异步回调接口
- ctx - 传递上下文参数



## Watcher

### 架构

由三个部分组成：

- Zookeeper服务端

- Zookeeper客户端

- 客户端的ZKWatchManager对象

  客户端首先将Watcher注册到服务端，同时将Watcher对象保存到客户端的Watch管理器中。当Zookeeper服务端监听的数据状态发生变化时，服务端会主动通知客户端，接着客户端的Watch管理器会触发相关Watcher来回调相应处理逻辑，从而完成整体的数据发布/订阅流程

![image-20210105204303419](/Users/didi/Documents/zookeeper.assets/image-20210105204303419.png)



### Watcher特性

| 特性           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| 一次性         | watcher时一次性的，一旦被触发就会移除，再次使用时需要重新注册 |
| 客户端顺序回调 | watcher回调是顺序串行化执行的，只有回调后客户端才能看到最新的数据状态。一个watcher回调逻辑不应该太多，以免影响别的watcher执行 |
| 轻量级         | watchEvent是最小的通信单元，结构上只包含通知状态、事件类型和节点路径，并不会告诉数据节点变化前后的具体内容。 |
| 时效性         | watcher只有在当前session彻底失效时才会无效，若在session有效期内快速重连成功，则watcher依然存在，仍可接受到通知。 |



### watcher接口设计

​	watcher是一个接口，任何实现了watcher接口的类就是一个新的watcher。wathcer内部包含了两个枚举类：KeeperState、EventType。



### 捕获相应的事件

​	在Zookeeper中采用zk.getChildren(path,watch)、zk.exists(path,watch)、zk.getData(path,watcher,stat)这样的方式为某个znode注册监听。

![image-20210105205222144](/Users/didi/Documents/zookeeper.assets/image-20210105205222144.png)



### 检测节点是否存在

```java
//使用连接对象的监视器
exists(String path,boolean b)
//自定义监视器
exists(String path,watcher w)
  
  //NodeCreated:节点创建
  //NodeDeleted:节点删除
  //NodeDataChanged:节点数据发生变化
```

- Path - znode路径
- b - 是否使用连接对象中注册的监视器
- w - 监视器对象



### 查看节点

```java
//使用连接对象的监视器
getData(String path,boolean b ,Stat stat)
//自定义监视器
getData(String path,Wathcher w,Stat stat)  
  
  //NodeDeleted:节点删除
  //NodeDataChanged：节点内容发生变化
```



### 查看子节点

```java
//使用连接对象的监视器
getChildren(String path,boolean b)
//自定义监视器
getChildren(String path,Watcher w)
  
  //NodeChildrenChanged:子节点发生变华
  //NodeDeleted:节点删除
```



## 配置中心案例

​	工作中的场景：数据库用户名和密码信息放在一个配置文件中，应用读取该配置文件，配置文件信息放入缓存。

​	若数据库的用户名和密码改变时，还需要重新加载缓存，较为麻烦，通过Zookeeper可以轻松完成，当数据库发生变化时自动完成缓存同步。

​	设计思路：

1. 连接zookeeper服务器
2. 读取zookeeper中的配置信息，注册watcher监听器，存入本地变量
3. 当zookeeper中的配置信息发生变化时，通过watcher的回调方法捕获数据变化事件
4. 重新获取配置信息



## 分布式唯一id

设计思路：

1. 连接zookeeper服务器
2. 指定路径生成临时有序节点
3. 取序列号及分布式环境下的唯一id



## 分布式锁

​	分布式锁有多种实现方式，比如通过数据库、redis都可以实现。作为分布式协同工具ZooKeeper，也可以实现排他锁。

设计思路：

1. 每个客户端/Locks下创建临时有序节点/Locks/Lock_，创建成功后/Locks下面会有每个客户端对应的节点，如/Locks/Lock_000000001
2. 客户端取得/Locks下子节点，并进行排序，判断排在最前面的是否为自己，如果自己的锁节点在第一位，代表获取锁成功
3. 如果自己的锁节点不在第一位，则监听自己前一位的锁节点。例如，自己的锁节点为/Lock_000000002，那么监听Lock_000000001
4. 当前一位锁节点（Lock_000000001）对应的客户端执行完成，释放了锁，将会触发监听客户端（Lock_000000002）的逻辑
5. 监听客户端重新执行第2步逻辑，判断自己是否获得了锁





## 集群搭建

1、基于zookeeper复制三份zookeeper安装好的服务器文件，目录名称分别为zookeeper2181、zookeeper2182、zookeeper2183。

```shell
cp -r zookeeper zookeeper2181
cp -r zookeeper zookeeper2182
cp -r zookeeper zookeeper2183
```

2、修改对应服务器对应的配置文件zoo.zfg

```shell
#服务器对应端口号
clientPort=2181
#数据快照文件所在路径
dataDir=/home/zookeeper/zookeeper2181/data
#集群配置信息
	#server .A=B:C:D
	#A：是一个数字，表示这个是服务器的编号
	#B：是这个服务器的IP地址
	#：ZooKeeper服务器之间的通信接口
	#D：Leader选举的端口
server .1 =192.168.60.130:2287:3387
server .2 =192.168.60.130:2288:3388
server .3 =192.168.60.130:2289:3389
```

3、在上一步dataDir指定的目录下，创建myid文件，然后在该文件添加上一步server配置的对应A数字

```shell
#zookeeper2181对应的数字是1
#/home/zookeeper/zookeeper2181/data目录下执行命令
echo "1"->myid
```



## 一致性协议.zab协议

​	zab协议的全称是ZooKeeper Atomic Broadcast（zookeeper原子广播）。zookeeper是通过zab协议来保证分布式事务的最终一致性。

​	基于zab协议，zookeeper集群中的角色主要有以下三类：

领导者：领导者负责进行投票的发起和决议，更新系统状态。

学习者：

- 跟随者：Follower用于接收客户请求并向客户端返回结果，在选举过程中参与投票
- 观察者：ObServer可以接受客户端连接，将写请求转发给leader节点。但ObServer不参加投票过程，只同步leader的状态。ObServer的目的是为了扩展系统，提高读取速度。

客户端：请求发起方。



工作原理：

1. leader从客户端收到一个写请求
2. ledaer生成一个新的事务并为这个事务生成一个唯一的ZXID
3. leader将这个事务提议（propose）发送给所有的follows节点
4. follower节点将收到的事务请求加入到历史队列中，并发送ack给leader
5. 当leader收到大多数follower（半数以上节点）的ack消息，leader会发送commit请求
6. 当followder收到commit请求时，从历史队列中将事务请求commit



## leader选举

### 服务器状态

looking：寻找leader状态。当服务器处于该状态时，它会认为当前集群中没有leader，因此需要进入leader选举状态。

leader：领导者状态。表明当前服务器角色是leader。

following：跟随者状态。表明当前服务器角色是follower。

observing：观察者状态。表明当前服务器角色是observer。



### 启动时期

​	在集群初始化阶段，当有一台服务器server1启动时，其单独无法进行和完成leader选举，当第二台服务器server2启动时，此时两台机器可以互相通信，每台机器都试图找到leader，于是进入leader选举过程。选举过程如下：

1. ​	每个server发出一个投票。由于是初始情况，server1和server2都会将自己作为leader服务器来进行投票，每次投票会包含所推举的服务器的myid和zxid，使用(myid,zxid)来表示，此时server1的投票为（1，0），server2位（2，0），然后各自将这个投票发给集群中的其他机器。
2. 集群中的每台服务器接收来自集群中各个服务器的投票。
3. 处理投票。针对每一票，服务器都需要将别人的投票和自己的投票进行pk，规则如下：

- 优先检查zxid，zxid比较大的服务器优先作为leader
- 如果zxid相同，那么比较myid，myid较大的服务器作为leader。

1. 统计投票
2. 更改服务器状态。一旦确定了leader，每台服务器就会更新自己的状态。



### 运行时期

​	在运行期间，一旦leader服务器挂了，那么整个集群将暂停对外服务，进行新一轮leader选举，其过程与启动时期的过程基本一致。



## observer角色及其配置

​	observer角色特点：

1. 不参与集群的leader选举
2. 不参与集群中写数据时的ack反馈

为了使用observer角色，在任何想变成observer角色的配置文件中加入如下配置：

```shell
peerType=observer
```

并在所有server的配置文件中，配置成observer模式的server的那行配置追加:observer，例如：

```shell
server .3=192.168.60.130:2289:3389:observer
```



## zookeeperAPI连接集群

```java
ZooKeeper(String connectionString,int sessionTimeout,Watcher watcher)
```

- connectionString - zooKeeper集合主机
- sessionTimeout - 会话超时（毫秒）
- watcher - 实现监视器洁面的对象，Zookeeper集合通过监视器对象返回连接状态



# Curator

​	是开源的zookeeper客户端。提供ZooKeeper各种应用场景（如：分布式锁服务、集群领导选举、共享计数器、缓存机制、分布式队列等）的抽象封装，实现了Fluent风格的API接口，是较为主流的ZK客户端。

​	原生ZooKeeperAPI不足：

- 连接对象异步创建，需要开发人员自行编码等待
- 连接没有自动重连超时机制
- watcher一次注册生效一次
- 不支持递归创建树形节点

Curator特点：

- 解决Session会话超时重连
- watcher的多次注册
- 简化开发API
- 遵循Fluent风格的API
- 提供了分布式锁服务、共享计数器、缓存等待机制等

**maven依赖：**

```java
 <dependencies>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.2.0</version>
        </dependency>
   <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.2</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
        <dependency>
            <groupId>jline</groupId>
            <artifactId>jline</artifactId>
            <version>0.9.94</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```



### create 创建

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.data.ACL;
import org.apache.zookeeper.data.Id;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

/**
 * @author xiongziyang
 * @date 2021/1/13 11:37 上午
 * @describe
 */
public class CuratorCreate {

    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("create")
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void create1() throws Exception {
        //新增节点
        client.create()
                //节点的类型
                .withMode(CreateMode.PERSISTENT)
                //节点权限列表
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                //arg1:节点的路径
                //arg2:节点的数据
                .forPath("/node1","node1".getBytes());
        System.out.println("结束");
    }

    @Test
    public void create2() throws Exception {
        //自定义权限列表
        List<ACL> list = new ArrayList<ACL>();
        //授权模式和授权对象
        Id id = new Id("ip","121.196.108.172");
        list.add(new ACL(ZooDefs.Perms.ALL,id));
        client.create()
                .withMode(CreateMode.PERSISTENT)
                .withACL(list)
                .forPath("/node2","node2".getBytes());
        System.out.println("结束");
    }

    @Test
    public void create3() throws Exception {
        //递归创建节点树
        client.create().creatingParentsIfNeeded()
                .withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .forPath("/node3/node31","node31".getBytes());
        System.out.println("结束");
    }

    @Test
    public void create4() throws Exception {
        //异步方式创建节点
        client.create().creatingParentsIfNeeded()
                .withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .inBackground(new BackgroundCallback() {
                    @Override
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        System.out.println(curatorEvent.getPath());
                        System.out.println(curatorEvent.getType());
                    }
                })
                .forPath("/node4","node4".getBytes());
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### set 更新

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

/**
 * @author xiongziyang
 * @date 2021/1/13 4:11 下午
 * @describe
 */
public class CuratorSet {

    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("set")
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void set1() throws Exception {
        //更新节点
        client.setData()
                .forPath("/node1","node11".getBytes());
        System.out.println("结束");
    }

    @Test
    public void set2() throws Exception {
        client.setData().withVersion(-1).forPath("/node1","node1111".getBytes());
        System.out.println("结束");
    }

    @Test
    public void set3() throws Exception {
        //异步方式修改节点
        client.setData().withVersion(-1).inBackground(new BackgroundCallback() {
            @Override
            public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                System.out.println(curatorEvent.getPath());
                System.out.println(curatorEvent.getType());
            }
        })
                .forPath("/node1","node1".getBytes());
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### delete 删除

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

/**
 * @author xiongziyang
 * @date 2021/1/13 4:21 下午
 * @describe
 */
public class CuratorDelete {

    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("delete")
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void delete1()throws Exception{
        client.delete()
                .forPath("/node1");
        System.out.println("结束");
    }

    @Test
    public void delete2()throws Exception{
        //删除指定版本号的子节点
        client.delete()
                .withVersion(-1)
                .forPath("/node1");
        System.out.println("结束");
    }

    @Test
    public void delete3()throws Exception{
        //删除带有子节点的节点
        client.delete()
                .deletingChildrenIfNeeded()
                .withVersion(-1)
                .forPath("/node1");
        System.out.println("结束");
    }

    @Test
    public void delete4()throws Exception{
        client.delete()
                .withVersion(-1)
                .inBackground(new BackgroundCallback() {
                    @Override
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        System.out.println(curatorEvent.getPath());
                        System.out.println(curatorEvent.getType());
                    }
                }).forPath("node1");

        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### get 查看节点

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.data.Stat;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

/**
 * @author xiongziyang
 * @date 2021/1/13 4:27 下午
 * @describe
 */
public class CuratorGet {


    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("get")
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void Get1()throws Exception{
        //读取节点数据
        byte[] bys=client.getData().forPath("/node1");
        System.out.println(new String(bys));
    }

    @Test
    public void Get2()throws Exception{
        //读取数据时读取数据的属性
        Stat stat = new Stat();
        byte[] bys = client.getData().storingStatIn(stat).forPath("/node1");
        System.out.println(new String(bys));
        System.out.println(stat.getAversion());
    }

    @Test
    public void Get3()throws Exception{
        //异步读取数据
        client.getData()
                .inBackground(new BackgroundCallback() {
                    @Override
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        System.out.println(curatorEvent.getPath());
                        System.out.println(curatorEvent.getType());
                        System.out.println(new String(curatorEvent.getData()));
                    }
                })
                .forPath("/node1");
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### 读取子节点名称

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.util.List;

/**
 * @author xiongziyang
 * @date 2021/1/13 4:34 下午
 * @describe
 */
public class CuratorGetChild {
    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void getChild1()throws Exception{
        //读取子节点数据
        List<String> list = client.getChildren().forPath("/get");
        for(String s:list){
            System.out.println(s);
        }
    }

    @Test
    public void getChild2()throws Exception{
        //异步
        client.getChildren()
                .inBackground(new BackgroundCallback() {
                    @Override
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        System.out.println(curatorEvent.getType());
                        System.out.println(curatorEvent.getPath());
                        List<String> list = curatorEvent.getChildren();
                        System.out.println(list);
                    }
                })
                .forPath("/get");
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### 检查节点是否存在

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.data.Stat;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

/**
 * @author xiongziyang
 * @date 2021/1/13 4:41 下午
 * @describe
 */
public class CuratorExists {

    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("get")
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void exists1()throws Exception{
        //判断节点是否存在
        Stat stat = client.checkExists().forPath("/node1");
        System.out.println(stat);
    }
    @Test
    public void exists2()throws Exception{
        client.checkExists()
                .inBackground(new BackgroundCallback() {
                    @Override
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        System.out.println(curatorEvent.getPath());
                        System.out.println(curatorEvent.getType());
                        System.out.println(curatorEvent.getStat());
                    }
                }).forPath("/node2");
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### watcherAPI

curator提供了两种Watcher(Cache)来监听结点的变化

- Node Cache：只是监听某一个特定的节点，监听结点的新增和修改
- PathChildren Cache：监控一个ZNode的子节点，当一个子节点增加，更新，删除时，Path Cache会改变它的状态，会包含最新的子节点，子节点的数据和状态

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.*;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

/**
 * @author xiongziyang
 * @date 2021/1/13 4:49 下午
 * @describe
 */
public class curatorWatcher {

    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(10000)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void watcher1()throws Exception{
        //监视某个节点的数据变化
        //arg1：连接对象
        //arg2：监视的节点路径
        final NodeCache nodeCache = new NodeCache(client,"/watcher1");
        //启动监听器
        nodeCache.start();
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                //节点变化时回调的方法
                System.out.println(nodeCache.getCurrentData().getPath());
                System.out.println(new String(nodeCache.getCurrentData().getData()));
            }
        });
        Thread.sleep(10000);
        System.out.println("结束");
        //关闭监视器
        nodeCache.close();
    }

    @Test
    public void watcher2()throws Exception{
        //监视子节点的方法
        final PathChildrenCache pathChildrenCache = new PathChildrenCache(client,"/watcher1",true);
        pathChildrenCache.start();
        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            //当子节点发生变化时回调的方法
            @Override
            public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
                //节点的时间类型
                System.out.println(pathChildrenCacheEvent.getType());

                System.out.println(pathChildrenCacheEvent.getData().getPath());

                System.out.println(new String(pathChildrenCacheEvent.getData().getData()));
            }
        });
        Thread.sleep(10000);
        System.out.println("结束");
        pathChildrenCache.close();
    }
}

```



### Curator 事务

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

/**
 * @author xiongziyang
 * @date 2021/1/13 5:15 下午
 * @describe
 */
public class CuratorTransaction {
    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(10000)
                .retryPolicy(retryPolicy)
                .namespace("create")
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void tra1() throws Exception {
        client.inTransaction()
                .create().forPath("/node1","node111".getBytes())
                .and()
                .setData().forPath("/node2","node222".getBytes())
                .and().commit();
    }
}
```



### 分布式锁

InterProcessMutex：分布式可重入排它锁

InterPrecessReadWriteLock：分布式读写锁

```java
package com.xzy.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessLock;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.framework.recipes.locks.InterProcessReadWriteLock;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

/**
 * @author xiongziyang
 * @date 2021/1/13 5:25 下午
 * @describe
 */
public class CuratorLock {

    String IP = "121.196.108.172:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void lock1()throws Exception{
        //排它锁
        //arg1:连接对象
        //arg2:节点路径
        InterProcessLock lock = new InterProcessMutex(client,"/lock1");
        System.out.println("等待获取锁对象");
        //获取锁
        lock.acquire();

        for(int i = 1;i<=5;i++){
            Thread.sleep(3000);
            System.out.println(i);
        }

        //释放锁
        lock.release();
        System.out.println("等待释放锁");
    }

    @Test
    public void lock2()throws Exception{
        //读写锁
        InterProcessReadWriteLock lock = new InterProcessReadWriteLock(client,"/lock1");
        //获取读锁对象
        InterProcessLock interProcessLock = lock.readLock();
        System.out.println("等待获取锁对象");
        interProcessLock.acquire();
        for(int i = 1;i<=5;i++){
            Thread.sleep(3000);
            System.out.println(i);
        }
        interProcessLock.release();
        System.out.println("等待释放锁对象");
    }

    @Test
    public void lock3()throws Exception{
//读写锁
        InterProcessReadWriteLock lock = new InterProcessReadWriteLock(client,"/lock1");
        //获取写锁对象
        InterProcessLock interProcessLock = lock.writeLock();
        System.out.println("等待获取锁对象");
        interProcessLock.acquire();
        for(int i = 1;i<=5;i++){
            Thread.sleep(3000);
            System.out.println(i);
        }
        interProcessLock.release();
        System.out.println("等待释放锁对象");
    }
}
```



### 监控命令

​	zookeeper支持某些特点的四字命令与其的交互。它们大多是查询命令，用来获取zookeeper服务的当前状态及相关信息。用户在客户端可以通过telnet或nc向zookeeper提交相应的命令。zookeeper常用四字命令见下表：

| 命令 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| conf | 输出相关服务配置的详细信息。比如端口、zk数据及日志配置路径、最大连接数，session超时时间、serverId等 |
| cons | 列出所有连接到这台服务器的客户端连接/会话的详细信息。包括"接收/发送"的包数量、session id、操作延迟、最后的操作执行等信息 |
| crst | 重置之当前这台服务器所有连接/会话的统计信息                  |
| dump | 列出未经处理的会话和临时节点                                 |
| envi | 输出关于服务器的环境详细信息                                 |
| ruok | 测试服务是否处于正确运行状态。如果正常返回"imok"，否则返回空 |
| stat | 输出服务器的详细信息：接收/发送包数量、连接数、模式(leader/follower)、节点总数、延迟。所有客户端的列表 |
| srst | 重置server状态                                               |
| wchs | 列出服务器watches的简洁信息：连接总数、watching节点总数和watches总数 |
| wchc | 通过session分组，列出watch的所有节点，它的输出是一个与watch相关的会话的节点列表 |
| mntr | 列出集群的健康状态。包括"接收/发送"的包数量、操作延迟、当前服务模式(leader/follower)、节点总数、watch总数、临时节点总数 |

nc命令工具安装：

```shell
#root用户安装
#下载安装包
wget http://vault.centos.org/6.6/os/x86_64/Packages/nc-1.84-22.el.x86_64.rpm
#rpm安装
rpm -iUv nc-1.84-22.el6.x86_64.rpm
```



telnet用法：

```shell
#现在服务器上登陆
telnet 121.196.108.172 2181
#登陆后使用对应的四字命令
```



#### conf 服务配置的详细信息

clientPort 客户端端口号
dataDir 数据快照文件目录 默认10万次操作生成一次快照
dataLogDir 事务日志文件目录，生产环境放在独立磁盘上
tickTime 服务器之间或客户端与服务器之间维持心跳的时间间隔
maxClientCnxns 最大连接数
minSessionTimeout 最小session超时时间 心跳时间x2 指定时间小于该时间默认使用此时间
maxSessionTimeout 最大session超时时间 心跳时间x20 指定时间大于该时间默认使用此时间
serverId 服务器编号
initLimit 集群中follow与leader之间初始连接能容忍的最大心跳数
syncLimit 集群中follow与leader之间请求和应答能容忍的最大心跳数
electionAlg 选举算法 默认值为3 基于TCP 另外0、1、2三种算法已经被弃用
electionPort 选举端口
quorumPort 集群之间的通信端口
peerType 是否观察者 1为观察者



#### cons 所有连接到这台服务器的客户端连接/会话的详细信息

```shell
#服务器中输入
echo cons|nc 121.196.108.172 2181
```

45692 客户端发送请求的端口号
queued 等待处理的请求数
received 收到的包数
sent 发送的包数
sid 会话id
lop 最后的操作的操作类型
est 连接时间戳
to 超时时间
lcxid 当前会话的操作id
lzxid 最大事务id
lresp 最后响应时间戳
llat 最新延时
minlat 最小延时
avglat 平均延时
maxlat 最大延时

#### crst 重置当前服务器所有连接/会话的统计信息

#### dump 列出未经处理的会话和临时结点

#### envi 输出服务器环境配置信息

从上到下分别为：
zookeeper版本
host名字
Java版本
供应商
jre目录
Java classpath
java第三方类库路径
Java临时文件路径
JIT编译器名称
操作系统名字
操作系统位数
操作系统版本
用户名
用户目录
用户bin目录

#### ruok 测试服务器是否在处在运行状态

#### stat 输出服务器详细信息

从上到下依次为：
zookeeper版本
延时 最小/平均/最大
收包
发包
连接数 2 因为nc命令也会创建一个
堆积的未处理请求数
最大事务id
服务器模式
结点数

#### srvr 类似stat，少了连接的会话信息

#### srst 重置服务器

#### wchs 列出watcher信息

#### wchc 通过session分组列出watch的节点

#### wchp 类似wchc，根据节点路径分组

#### mntr 列出服务器的健康状态

版本

平均延时

最大延时

最小延时

收包数

发包数

连接数

堆积请求数

leader/follower状态

znode数量

watch数量

临时节点（znode）

数据大小

打开的文件描述符数量

最大文件描述符数量