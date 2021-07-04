# ==========  redis数据类型   ===========

## 0 Redis-Key
~~~
127.0.0.1:6379> keys * # 查看所有的key 
(empty list or set) 
127.0.0.1:6379> set name hello # set key 
OK
127.0.0.1:6379> keys * 
1) "name" 
127.0.0.1:6379> set age 1 
OK
127.0.0.1:6379> keys * 
1) "age" 
2) "name" 
127.0.0.1:6379> EXISTS name # 判断当前的key是否存在 
(integer) 1 
127.0.0.1:6379> EXISTS name1 
(integer) 0 
127.0.0.1:6379> move name 1 # 将当前key移动到其他库并移除当前的key 
(integer) 1 
127.0.0.1:6379> keys * 
1) "age" 
127.0.0.1:6379> set name heiheihei 
OK
127.0.0.1:6379> keys * 
1) "age" 
2) "name" 
127.0.0.1:6379> clear 
127.0.0.1:6379> keys * 
1) "age" 
2) "name" 
127.0.0.1:6379> EXPIRE name 10 # 设置key的过期时间，单位是秒 
(integer) 1 
127.0.0.1:6379> ttl name # 查看当前key的剩余时间 
(integer) 4 
127.0.0.1:6379> type name # 查看当前key的一个类型！ 
string
~~~
## 1 字符串

string是redis最基本的类型，一个key对应一个value。string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象。string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M
~~~
127.0.0.1:6379> set key1 v1 # 设置值 
OK
127.0.0.1:6379> get key1 # 获得值 
"v1"
127.0.0.1:6379> APPEND key1 "hello" # 追加字符串，如果当前key不存在，就相当于setkey
127.0.0.1:6379> STRLEN key1 # 获取字符串的长度

127.0.0.1:6379> incr views # 自增1 浏览量变为1
127.0.0.1:6379> decr views # 自减1 浏览量-1
127.0.0.1:6379> INCRBY views 10 # 可以设置步长，指定增量！
127.0.0.1:6379> DECRBY views 5

127.0.0.1:6379> GETRANGE key1 0 3 # 截取字符串 [0,3]
127.0.0.1:6379> GETRANGE key1 0 -1 # 获取全部的字符串 和 get key是一样的

127.0.0.1:6379> SETRANGE key2 1 xx # 替换指定位置开始的字符串！


# setex (set with expire) # 设置过期时间 
# setnx (set if not exist) # 不存在在设置 （在分布式锁中会常常使用！）
127.0.0.1:6379> setex key3 30 "hello" # 设置key3 的值为 hello,30秒后过期
127.0.0.1:6379> setnx mykey "redis" # 如果mykey 不存在，创建mykey
127.0.0.1:6379> setnx mykey "MongoDB" # 如果mykey存在，创建失败！


127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3 # 同时设置多个值
127.0.0.1:6379> mget k1 k2 k3 # 同时获取多个值

127.0.0.1:6379> getset db mongodb # 如果存在值，获取原来的值，并设置新的值 不存在返回空
~~~

## 2 哈希

Redis hash 是一个键值对集合。Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。类似Java里面的Map<String,Object>

## 3 列表

Redis List列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。
它的底层实际是个链表,所有的list命令都是用l开头的，Redis不区分大小命令

## 4 集合

Redis的Set是string类型的无序集合。它是通过HashTable实现的

## 5 有序集合

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。