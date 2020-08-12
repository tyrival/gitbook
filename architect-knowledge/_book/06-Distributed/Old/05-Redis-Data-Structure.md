# Redis核心数据结构与核心原理

## 1. Redis安装

```bash
下载地址：http://redis.io/download
安装步骤：
# 安装gcc
yum install gcc

# 把下载好的redis-5.0.3.tar.gz放在/usr/local文件夹下，并解压
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
tar xzf redis-5.0.3.tar.gz
cd redis-5.0.3

# 进入到解压好的redis-5.0.3目录下，进行编译与安装
make

# 启动并指定配置文件
src/redis-server redis.conf（注意要使用后台启动，所以修改redis.conf里的daemonize改为yes)

# 验证启动是否成功 
ps -ef | grep redis 

# 进入redis客户端 
src/redis-cli 

# 退出客户端
quit

# 退出redis服务： 
（1）pkill redis-server 
（2）kill 进程号                       
（3）src/redis-cli shutdown 
```



## 2. Redis的单线程和高性能

##### Redis 单线程为什么还能这么快？

因为它所有的数据都在**内存**中，所有的运算都是内存级别的运算，而且单线程避免了多线程的切换性能损耗问题。正因为 Redis 是单线程，所以要小心使用 Redis 指令，对于那些耗时的指令(比如keys)，一定要谨慎使用，一不小心就可能会导致 Redis 卡顿。 

##### Redis 单线程如何处理那么多的并发客户端连接？

Redis的**IO多路复用**：redis利用epoll来实现IO多路复用，将连接信息和事件放到队列中，依次放到文件事件分派器，事件分派器将事件分发给事件处理器。

![redis-io-multiplexing](../../source/images/ch-06/old/redis-io-multiplexing.png)

```bash
# 查看redis支持的最大连接数，
# 在redis.conf文件中可修改，# maxclients 10000
127.0.0.1:6379> CONFIG GET maxclients
1) "maxclients"
2) "10000"
```



## 3. Redis数据结构和命令

### 3.1 数据结构

Redis主要包含以下数据结构：

![redis-data-structure](../../source/images/ch-06/old/redis-data-structure.png)

- 字符串string
- 哈希hash
- 列表list
- 集合set
- 有序集合zset

#### 字符串string

###### 字符串常用操作

```bash
# 存入字符串键值对
SET  key  value
# 批量存储字符串键值对
MSET  key  value [key value ...]
# 存入一个不存在的字符串键值对
SETNX  key  value
# 获取一个字符串键值
GET  key
# 批量获取字符串键值
MGET  key  [key ...]
# 删除一个键
DEL  key  [key ...]
# 设置一个键的过期时间(秒)
EXPIRE  key  seconds
```

###### 原子加减

```bash
# 将key中储存的数字值加1
INCR  key
# 将key中储存的数字值减1
DECR  key
# 将key所储存的值加上increment
INCRBY  key  increment
# 将key所储存的值减去decrement
DECRBY  key  decrement
```

###### 单值缓存

```bash
SET  key  value 	
GET  key
```

##### 应用场景

###### 对象缓存

- 方案A。将对象转成JSON格式，然后以字符串的形式存入redis

```json
SET  user:1  value(json格式数据)
```

- 方案B。将对象的各属性分开存储，

```bash
MSET  user:1:name  tyrival   user:1:balance  1888
MGET  user:1:name  user:1:balance
```

> **性能测试** [源码](https://github.com/tyrival/architect-knowledge/blob/master/06-distributed/src/main/java/com/tyrival/distributed/old/lession05/RedisOpsController.java)
>
> SET：方案A性能高于方案B，因为方案A只需要set一个key，而方案B需要根据属性的数量mset多个key；但是当方案B只需要修改一个属性值时候
>
> GET：经过多次测试，二者性能差不多
>
> ![redis-object-cache](../../source/images/ch-06/old/redis-object-cache.png)

###### 分布式锁

```bash
# 返回1代表获取锁成功
SETNX  product:10001  true
# 返回0代表获取锁失败
SETNX  product:10001  true

###### 执行业务操作 ######

# 执行完业务释放锁
DEL  product:10001
# 防止程序意外终止导致死锁
SET product:10001 true  ex  10  nx
```

> **注意**
>
> 这只是一个简单的分布式锁示意，实际上真正的分布式锁实现方式比这个复杂的多。

###### 计数器

例如统计一篇文章的阅读量，可以用redis进行实现。

```bash
INCR article:readcount:{文章id}  	
GET article:readcount:{文章id} 
```

###### Web集群session共享

spring session + redis实现session共享

###### 分布式系统全局序列号

当某个服务ServiceA部署了多个节点时，需要解决各节点上序列号的一致性问题。由于redis是单线程的，可以使用redis来生成序列号提供给ServiceA的各节点，从而解决一致性问题。

但是每次只获取1个序列号，会导致redis资源被大量消耗，所以可以一次申请大量序列号（例如1000个），存储在ServiceA的各节点的缓存中，等消耗完后再次申请。

```bash
# redis批量生成序列号提升性能
NCRBY  orderId  1000
```

#### 哈希hash

![redis-hash](../../source/images/ch-06/old/redis-hash.png)

###### Hash常用操作

```bash
# 存储一个哈希表key的键值
HSET  key  field  value
# 存储一个不存在的哈希表key的键值
HSETNX  key  field  value
# 在一个哈希表key中存储多个键值对
HMSET  key  field  value [field value ...]
# 获取哈希表key对应的field键值
HGET  key  field
# 批量获取哈希表key中多个field键值
HMGET  key  field  [field ...]
# 删除哈希表key中的field键值
HDEL  key  field  [field ...]
# 返回哈希表key中field的数量
HLEN  key
# 返回哈希表key中所有的键值
HGETALL  key
# 为哈希表key中field键的值加上增量increment
HINCRBY  key  field  increment
```

> **优点**
>
> 1. 同类数据归类整合储存，方便数据管理
>
> 2. 相比string操作消耗内存与cpu更小
>
> 3. 相比string储存更节省空间
>
> **缺点**
>
> 1. 过期功能不能使用在field上，只能用在key上
>
> 2. Redis集群架构下不适合大规模使用

##### 应用场景

###### 对象缓存

```bash
HMSET  user  {userId}:name  tyrival {userId}:balance  1888

## 例如
HMSET  user  1:name tyrival  1:balance 1888
HMGET  user  1:name  1:balance  
```

> **性能测试** [源码](https://github.com/tyrival/architect-knowledge/blob/master/06-distributed/src/main/java/com/tyrival/distributed/old/lession05/RedisOpsController.java)
>
> ![redis-hash-cache](../../source/images/ch-06/old/redis-hash-cache.png)

###### 电商购物车

- 以用户id为key
- 商品id为field
- 商品数量为value

```bash
# 购物车操作
## 添加商品
hset cart:1001 10088 1
## 增加数量
hincrby cart:1001 10088 1
## 商品总数
hlen cart:1001
## 删除商品
hdel cart:1001 10088
## 获取购物车所有商品
hgetall cart:1001
```

#### 数组List

![redis-list](../../source/images/ch-06/old/redis-list.png)

###### 常用操作

```bash
# 将一个或多个值value插入到key列表的表头(最左边)
LPUSH  key  value [value ...]
# 将一个或多个值value插入到key列表的表尾(最右边)
RPUSH  key  value [value ...]
# 移除并返回key列表的头元素
LPOP  key
# 移除并返回key列表的尾元素
RPOP  key
# 返回列表key中指定区间内的元素，区间以偏移量start和stop指定
LRANGE  key  start  stop
# 从key列表表头弹出一个元素，若列表中没有元素，阻塞等待timeout秒，如果timeout=0，一直阻塞等待
BLPOP  key  [key ...]  timeout
# 从key列表表尾弹出一个元素，若列表中没有元素，阻塞等待timeout秒，如果timeout=0，一直阻塞等待
BRPOP  key  [key ...]  timeout
```

###### 常用数据结构

- Stack(栈) = LPUSH + LPOP àFILO

- Queue(队列）= LPUSH + RPOP

- Blocking MQ(阻塞队列）= LPUSH + BRPOP

##### 应用场景

###### 微博消息和微信公号消息

我关注了MacTalk、备胎说车等大V，当他们发微博时，就会在属于我的timeline中插入一条消息数据，当我登录微博时，通过加载我的timeline，就可以看到所有的微博内容序列。

1. MacTalk发微博，消息ID为10018

```bash
LPUSH msg:{TYRIVAL_ID} 10018
```

2. 备胎说车发微博，消息ID为10086

```bash
LPUSH msg:{TYRIVAL_ID} 10086
```

3. 查看最新微博消息

```bash
LRANGE msg:{TYRIVAL_ID} 0 5
```

#### 集合Set

![redis-set](../../source/images/ch-06/old/redis-set.png)

###### 常用操作

```bash
# 往集合key中存入元素，元素存在则忽略，若key不存在则新建
SADD  key  member  [member ...]
# 从集合key中删除元素
SREM  key  member  [member ...]
# 获取集合key中所有元素
SMEMBERS  key
# 获取集合key的元素个数
SCARD  key
# 判断member元素是否存在于集合key中
SISMEMBER  key  member
# 从集合key中选出count个元素，元素不从key中删除
SRANDMEMBER  key  [count]
# 从集合key中选出count个元素，元素从key中删除
SPOP  key  [count]
```

###### 运算操作

```bash
# 交集运算
SINTER  key  [key ...]
# 将交集结果存入新集合destination中
SINTERSTORE  destination  key  [key ..]
# 并集运算
SUNION  key  [key ..]
# 将并集结果存入新集合destination中
SUNIONSTORE  destination  key  [key ...]
# 差集运算
SDIFF  key  [key ...]
# 将差集结果存入新集合destination中
SDIFFSTORE  destination  key  [key ...]
```

> **示例**
>
> ```bash
> set1={a, b, c}，set2={b, c, d}，set3={c, d, e}
> SINTER set1 set2 set3 # { c }
> SUNION set1 set2 set3 # { a,b,c,d,e }
> SDIFF set1 set2 set3	# { a }
> ```

##### 应用场景

###### 微信抽奖小程序

1. 点击参与抽奖加入集合

```bash
SADD key {userlD}
```

2. 查看参与抽奖所有用户

```bash
SMEMBERS key  
```

3. 抽取count名中奖者

```bash
SRANDMEMBER key [count] / SPOP key [count]
```

###### 微信微博点赞，收藏，标签

1. 点赞

```bash
SADD like:{消息ID} {用户ID}
```

2. 取消点赞

```bash
SREM like:{消息ID} {用户ID}
```

3. 检查用户是否点过赞

```bash
SISMEMBER like:{消息ID} {用户ID}
```

4. 获取点赞的用户列表

```bash
SMEMBERS like:{消息ID}
```

5. 获取点赞用户数

```bash
SCARD like:{消息ID}
```

###### 集合操作实现微博微信关注模型

1. Tom关注的人: 

```bash
TomSet -> {Jerry, Droopy, Spike}
```

2. Jerry关注的人:

 ```bash
JerrySet -> {Tom, Droopy, Tuffy}
 ```

3. Droopy关注的人: 

```bash
DroopySet -> {Tom, Jerry, Droopy)
```

4. Tom和Jerry共同关注: 

```bash
SINTER TomSet JerrySet -> {Droopy}
```

5. Tom关注的人也关注Jerry: 

```bash
SISMEMBER DroopySet Jerry 
```

5. Tom可能认识的人: 

```bash
SDIFF JerrySet TomSet -> (Tuffy}
```

###### 集合操作实现电商商品筛选

```bash
# 将P30增加到huawei品牌集合
SADD  brand:huawei  P30
# 将mi-6x增加到xiaomi品牌集合
SADD  brand:xiaomi  mi-6X
# 将iphone8增加到iphone品牌集合
SADD  brand:iPhone iphone8
# 将P30和mi-6x增加到android系统集合
SADD  os:android  P30  mi-6X
# 将P30和mi-6x增加到cpu品牌为intel的集合
SADD  cpu:brand:intel  P30  mi-6X
# 将P30、mi-6x和iphone8增加到8G内存的集合
SADD  ram:8G  P30  mi-6X  iphone8
## 筛选安卓系统，cpu品牌为intel，8G内存的手机，结果为P30和mi-6x
SINTER  os:android  cpu:brand:intel  ram:8G  =>  {P30，mi-6X}
```

#### 有序集合ZSet

![redis-zset](../../source/images/ch-06/old/redis-zset.png)

###### 常用操作

```bash
# 往有序集合key中加入带分值元素
ZADD key score member [[score member]…]
# 从有序集合key中删除元素
ZREM key member [member …]
# 返回有序集合key中元素member的分值
ZSCORE key member
# 为有序集合key中元素member的分值加上increment 
ZINCRBY key increment member
# 返回有序集合key中元素个数
ZCARD key
# 正序获取有序集合key从start下标到stop下标的元素
ZRANGE key start stop [WITHSCORES]
# 倒序获取有序集合key从start下标到stop下标的元素
ZREVRANGE key start stop [WITHSCORES]
```

###### 集合操作

```bash
# 并集计算
ZUNIONSTORE destkey numkeys key [key ...]
# 交集计算
ZINTERSTORE destkey numkeys key [key ...]
```

##### 应用场景

###### 实现排行榜 

1. 点击新闻

```bash
ZINCRBY hotNews:20190819 1 守护香港
```

2. 展示当日排行前十

```bash
ZREVRANGE hotNews:20190819 0 10 WITHSCORES 
```

3. 七日搜索榜单计算

```bash
ZUNIONSTORE hotNews:20190813-20190819 7 
hotNews:20190813 hotNews:20190814... hotNews:20190819
```

4. 展示七日排行前十

```bash
ZREVRANGE hotNews:20190813-20190819 0 10 WITHSCORES
```



### 3.2 其他高级命令

##### keys：全量遍历键

用来列出所有满足特定正则字符串规则的key，当redis数据量比较大时，性能比较差，**而且会阻塞后续命令**，要避免使用

![redis-cmd-keys](../../source/images/ch-06/old/redis-cmd-keys.png)

##### scan：渐进式遍历键

```bash
SCAN cursor [MATCH pattern] [COUNT count]
```

scan 参数提供了三个参数，第一个是 cursor 整数值，第二个是 key 的正则模式，第三个是一次遍历的key的数量，并不是符合条件的结果数量。第一次遍历时，cursor 值为 0，然后将返回结果中第一个整数值作为下一次遍历的 cursor。一直遍历到返回的 cursor 值为 0 时结束。

![redis-cmd-scan](../../source/images/ch-06/old/redis-cmd-scan.png)

##### Info：查看redis服务运行信息

分为 9 大块，每个块都有非常多的参数，这 9 个块分别是: 

- Server 服务器运行的环境参数 

- Clients 客户端相关信息 

- Memory 服务器运行内存统计数据 

- Persistence 持久化信息 

- Stats 通用统计数据 

- Replication 主从复制相关信息 

- CPU CPU 使用情况 

- Cluster 集群信息 

- KeySpace 键值对统计数量信息

