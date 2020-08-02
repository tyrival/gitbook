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

#### Redis 单线程为什么还能这么快？

因为它所有的数据都在**内存**中，所有的运算都是内存级别的运算，而且单线程避免了多线程的切换性能损耗问题。正因为 Redis 是单线程，所以要小心使用 Redis 指令，对于那些耗时的指令(比如keys)，一定要谨慎使用，一不小心就可能会导致 Redis 卡顿。 

#### Redis 单线程如何处理那么多的并发客户端连接？

Redis的**IO多路复用**：redis利用epoll来实现IO多路复用，将连接信息和事件放到队列中，依次放到文件事件分派器，事件分派器将事件分发给事件处理器。

![img](../../source/images/ch-06/old/io-multiplexing.png)

```bash
# 查看redis支持的最大连接数，在redis.conf文件中可修改，# maxclients 10000
127.0.0.1:6379> CONFIG GET maxclients
1) "maxclients"
2) "10000"
```

#### 其他高级命令

**keys：全量遍历键**，用来列出所有满足特定正则字符串规则的key，当redis数据量比较大时，性能比较差，要避免使用

![img](https://note.youdao.com/yws/public/resource/ac721d97d0b6c27ee57b14e58542c560/xmlnote/1C0AAFB3256E482980B1F594CF4356F0/80857)

**scan：渐进式遍历键**

```bash
SCAN cursor [MATCH pattern] [COUNT count]
```

scan 参数提供了三个参数，第一个是 cursor 整数值，第二个是 key 的正则模式，第三个是一次遍历的key的数量，并不是符合条件的结果数量。第一次遍历时，cursor 值为 0，然后将返回结果中第一个整数值作为下一次遍历的 cursor。一直遍历到返回的 cursor 值为 0 时结束。

![img](https://note.youdao.com/yws/public/resource/ac721d97d0b6c27ee57b14e58542c560/xmlnote/CF1423AD5A8B4D5E973D17176A70A2D2/80858)

![img](https://note.youdao.com/yws/public/resource/ac721d97d0b6c27ee57b14e58542c560/xmlnote/0DD7A48767FD49D19E75362E74331D50/80856)

**Info：**查看redis服务运行信息，分为 9 大块，每个块都有非常多的参数，这 9 个块分别是: 

Server 服务器运行的环境参数 

Clients 客户端相关信息 

Memory 服务器运行内存统计数据 

Persistence 持久化信息 

Stats 通用统计数据 

Replication 主从复制相关信息 

CPU CPU 使用情况 

Cluster 集群信息 

KeySpace 键值对统计数量信息

![img](https://note.youdao.com/yws/public/resource/ac721d97d0b6c27ee57b14e58542c560/xmlnote/F2578ACC62BB41358BDBC85983DCFB37/80478)

