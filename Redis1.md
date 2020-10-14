# 一、简介
## 定义
Redis（全称：Remote Dictionary Server 远程字典服务）是当前最为流行的NoSQL数据库之一，它是一个使用C语言编写的开源、包含多种数据结构、支持网络交互、基于内存、可选持久性的键值对存储数据库。
## 背景
因为传统的关系型数据库如MySQL目前已无法满足所有场景，例如秒杀的库存扣减，直接就会把数据库打崩，所以缓存中间件如Redis应运而生。


SQL 数据库的数据都是存在磁盘中的，虽然在数据库底层做了对应的缓存，但是这种数据库层次的缓存一般针对的是查询的内容，而且粒度也比较小，一般只有当表中的数据没有发生变动时，数据库对应的缓存才会发挥作用，但这并不能减少业务系统对数据库产生的增删改查的IO压力，因此缓存数据库应运而生。该技术实现了对热点数据的高速缓存，提高了应用的响应速度，极大缓解了后端数据库的压力。


出生于西西里岛的意大利人（antirez）在2007年和朋友共同创建一个网站LLOOGG.com，为了解决这个网站的负载问题，而在**2009**年开发了Redis数据库。
## 赞助商
从2010年3月15日起，Redis的开发工作由VMware主持。


从2013年5月开始，Redis的开发由Pivotal赞助。


从2015年6月开始，Redis的开发由Redis Labs赞助。
## 请求流程

- 当客户端，向后端发送请求时，会先去缓存层查看是否有该数据。
   - 如果有则直接从缓存层返回（减轻存储层的压力）。
   - 如果没有则会进行穿透查询，就是穿透缓存层去访问存储层进行查询。
      - 如果在存储层中查询到该数据，就将该数据回写到缓存层中，以便下次客户端再次查询时，能够直接从缓存层中获取该数据。回写的过程就叫回种。回种完成之后，就将结果返回至客户端。
      - 如果存储层故障（挂掉或者无法提供服务）可以使用熔断机制，让客户端的请求直接打在缓存层上，不管有没有获取到数据，都直接返回。这样就可以在有损的情况下，对外提供服务。
### Memcache和Redis的区别




Redis存储100k以下的小数据时性能高于Memcache。而Memcache存储100k以上的数据时性能高于Redis。


Redis使用单线程，而Memcached是多线程。


Redis使用直接申请内存的方式来存储数据，并且可以配置虚拟内存。而Memcached使用预分配内存池的方式。


Redis支持数据持久化。而Memcached不支持，只是存放在内存中，因此服务器关机后数据就会消失。


Redis支持主从与分片。Memcached并不支持。


MemCached可以修改最大内存，采用LRU算法。而Redis增加了VM的特性，突破了物理内存的限制。


Memcached单个key-value大小有限，一个value最大只支持1MB。而Redis最大支持512MB


Redis是计算向数据移动，而Memcache是数据向计算移动。 这是因为MemCached数据结构单一，只有String，如果需要存储其他类型数据就需要使用json格式进行存储，但是弊端就是每次操作数据都要全量IO，然后再反序列化，才可以进行操作。而Redis支持更加丰富的数据类型，也可以在服务器端直接对数据进行丰富的操作，这样可以减少网络IO次数和数据体积。


只是简单的key-value的存储可以选择Memcache。有持久化需求且对数据结构和处理有高级要求的可以选择Redis


**Redis的优势是什么？**


- 速度快
   - 将所有数据储存在内存里。读写数据的时候都不会受到硬盘IO速度的限制，所以速度极快。
   - 采用C语言实现的。一般来说C语言实现的程序“距离”操作系统更近，执行速度相对会更快
   - 工作线程采用单线程模型。单线程在内存中是效率最高的。避免了因多进程或者多线程导致的切换而消耗 CPU。不用去考虑各种锁的问题，不存在加锁释放锁操作，不会出现因死锁而导致的性能消耗。如果想要发挥多核CPU性能可以启动多个实例。
   - 使用多路 I/O 复用模型，为非阻塞 IO。Redis可通过多路复用器去监听客户端发起的socket连接的状态，而无需自己去一个个的确认socket连接的状态；然后再根据正确的连接状态去执行revc、read操作。在Redis6中引入多线程IO，解决了网络I/O瓶颈问题，性能大增。
   - 数据结构简单，对数据操作也简单，节省时间。Redis不使用表，它不会强制用户对各个关系（存储的不同数据）进行关联，不会有复杂的关系限制。其存储结构就是键值对，类似于 HashMap，HashMap 最大的优点就是存取的时间复杂度为 O(1)。
   - 自己的VM机制，避免系统调用系统函数时浪费一定的时间去移动和请求。
- 支持多种编程语言
   - Redis提供了简单的TCP通信协议，很多编程语言可以很方便地接入到Redis。
   - 例如：Java、PHP、Python、C、C++、Ruby、Lua、Node.js等
- 功能丰富
   - 提供了键过期功能，可以用来实现缓存。
   - 提供了发布订阅功能，可以用来实现消息系统。
   - 支持Lua脚本功能，可以利用Lua创造出新的Redis命令。
   - 提供了简单的事务功能，能在一定程度上保证事务特性。
   - 提供了流水线（Pipeline）功能，这样客户端能将一批命令一次性传到Redis，减少了网络的开销。
- 支持数据持久化
   - 通常看，将数据放在内存中是不安全的，一旦发生断电或者机器故障，重要的数据可能就会丢失，因此Redis为了保证数据的可持久性，将内存中的数据每隔一段时间就保存在磁盘中，重启的时候会重新加载到内存中，提供了两种持久化方式：RDB和AOF。
- 简单稳定
   - Redis的源码很少，只有23000行。
   - Redis使用单线程模型，这样不仅使得Redis服务端处理模型变得简单，而且也使得客户端开发变得简单。
   - Redis不需要依赖于操作系统中的类库（例如Memcache需要依赖libevent这样的系统类库），Redis自己实现了事件处理的相关功能。
   - Redis虽然很简单，但是不代表它不稳定。维护的上千个Redis为例，没有出现过因为Redis自身bug而宕掉的情况。
- 数据结构丰富
   - Redis主要提供了5种数据结构：字符串（String）、散列表（Hash）、列表（List）、集合（Set）、有序集合（Zset）。
- 支持主从同步
   - 主从同步就是一台主机多台从机，主机可以进行写、读，而从机只能读。如果主机故障，那么从机就会自动进行选举出一个从机做为主机，然后将主机的东西同步到备机中。当故障的主机被修复后，重新加入进来时，将沦为从机。
   - 它是分布式Redis的基础，高可用的基石。
      - 提高数据吞吐量
      - 提升效率
- 高可用和分布式
   - Redis从2.8版本正式提供了高可用实现Redis Sentinel，它能够保证Redis节点的故障发现和故障自动转移。
   - Redis从3.0版本正式提供了分布式实现Redis Cluster，它是Redis真正的分布式实现，提供了高可用、读写和容量的扩展性。



## NoSQL


### 定义
NoSQL(全称：Not Only SQL 不仅仅是SQL)，是一项全新的数据库理念，泛指非关系型的数据库。
### 相比关系型数据库的优缺点有那些？
关系型数据库与非关系型数据库并非对立而是互补的关系。


非关系型数据库基于键值对，数据之间没有耦合性，而关系型数据库有类似join这样的多表查询机制的限制，因此关系型数据库水平扩展性低于非关系型数据库


非关系型数据库将数据存储于内存中，而关系型数据库将数据存储在硬盘中，因此关系型数据库的查询速度低于非关系型数据库，但是关系型数据库容量高于非关系型数据库。


非关系型数据库的存储格式可以是key-value形式、文档形式、图片形式等等，所以可以存储基础类型以及对象或者是集合等各种格式，而数据库则只支持基础类型。
### 主流产品

- 键值(Key-Value)存储数据库
相关产品： Tokyo Cabinet/Tyrant、Redis、Voldemort、Berkeley DB
典型应用： 内容缓存，主要用于处理大量数据的高访问负载。
数据模型： 一系列键值对
优势： 快速查询
劣势： 存储的数据缺少结构化
- 列存储数据库
相关产品：Cassandra, HBase, Riak
典型应用：分布式的文件系统
数据模型：以列簇式存储，将同一列数据存在一起
优势：查找速度快，可扩展性强，更容易进行分布式扩展
劣势：功能相对局限
- 文档型数据库
相关产品：CouchDB、MongoDB
典型应用：Web应用（与Key-Value类似，Value是结构化的）
数据模型： 一系列键值对
优势：数据结构要求不严格
劣势： 查询性能不高，而且缺乏统一的查询语法
- 图形(Graph)数据库
相关数据库：Neo4J、InfoGrid、Infinite Graph
典型应用：社交网络
数据模型：图结构
优势：利用图结构相关算法。
劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。



# 二、准备工作
## 安装与卸载
### Linux


```shell
1) 选择位置
cd /usr/src

2) 下载Redis
wget http://download.redis.io/releases/redis-6.0.4.tar.gz

3) 解压
tar -zxvf redis-6.0.4.tar.gz

4) 进入解压后的目录
cd redis-6.0.4/ 

5) 安装GCC。
	1) 使用yum源安装
	yum install -y gcc
	2) 使用scl源临时安装
		1) 安装scl软件集
		yum -y install centos-release-scl
		2) 安装gcc、gcc-c++、binutils
		yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
		3) 切换版本
		#注意：只对本次shell有效。使用exit命令也可退出当前scl bash环境，恢复成系统bash环境。
		scl enable devtoolset-9 bash #使用scl创建一个scl包的bash会话环境，格式：scl enable <scl-package-name> <command>  #使用scl来执行command命令
		source /opt/rh/devtoolset-9/enable
		4) 查看版本
		gcc -v
		5) 长期生效
		echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile #永久覆盖
		
6) 编译并安装
make && make install
#编译提示致命错误：jemalloc/jemalloc.h：没有那个文件或目录
make MALLOC=libc
#指定安装到某个目录
make install PREFIX=/usr/local/redis

7) 查看相关文件
find / -name "redis"

8) 删除文件
rm -rf 文件
```


## 修改配置文件


```conf
daemonize yes	#是否启用后台运行。默认为no。
port 6379		#监听端口。默认为6379。
bind 0.0.0.0	#绑定主机地址。如果没有绑定，那么所有IP都可以连接（允许远程连接）。默认127.0.0.1（本机，拒绝任何远程连接）。
timeout 2000	#客户端空闲多少秒后关闭连接。默认为0（表示永不关闭）。
loglevel warning  #日志级别
logfile redis_log.log	#日志文件的名称。默认为""（表示不保存日志）。
databases 20	#数据库总数量。默认为16。从0开始，如果总数为16，那么最大为15号库。
requirepass 123456	#连接Redis数据库的密码。
pidfile /var/run/redis_6379.pid  #pid文件的路径
```


## 启动与关闭


### Linux


```shell
1) 启动服务并指定配置文件
./redis-server redis.conf
#如果要使用后台运行，请修改配置文件中daemonize的值为yes。如果未指定配置文件，走的就是默认配置

2) 查看Redis服务是否已在运行
ps -ef|grep redis 
ps aux|grep redis
netstat -tulpn | grep redis

3) 查看端口是否在被使用
netstart -tunpl|grep 端口号

4) 进入Redis客户端
./redis-cli
#如果修改了配置文件中的端口号，请指定端口号，否则提示Could not connect to Redis at 127.0.0.1:6379: Connection refused）
./redis-cli -p 端口号 
#如果不是连接本机Redis，请指定主机名
./redis-cli -h 主机名
#如需对中文进行转码（客户端可以查看中文），请配置参数
./redis-cli --raw
#如果需要输入密码，请指定密码
./redis-cli -a 密码

5) 退出客户端
quit

6) 关闭服务
./redis-cli shutdown
pkill redis-server
kill 进程号
```


# 、命令


## 数据库命令


```
1) 切换到某个库
select 库的编号（默认0~15）

2) 清空当前库中所有数据
flushDB

3) 清空所有库的数据
flushAll
```


## key相关命令


```
1) 删除某个（些）key
del key [key...]

2) 判断某个key是否存在
exists key

3) 给某个key设置过期时间（秒）
expire key seconds

4) 根据匹配规则查看当前库的所有key
keys 匹配规则（*表示匹配所有，?表示匹配任意一个字符，[]表示匹配[]中包含的字符） 

5) 将当前库中的某个key移动到某个库中
move key 库的编号

6) 给某个key设置过期时间（毫秒） 
pexpire key 

7) 给某个key设置过期时间（时间戳）
pexpireat key

8) 获取某个key过期剩余时间（秒）
ttl key 
返回值： >=0剩余时间 -1永不过期 -2当前key不存在 

9) 获取某个key过期剩余时间（毫秒）
pttl key 

10) 随机返回redis中一个存在 key
randornkey 

11) 修改某个已存在key的名称 
rename keyold keynewnarne

12) 查看某一个key所对应的value的数据类型
type key
```


# 、数据类型


## (一) 五大基本数据类型


### 1. String（字符串）


String最大容量为512mb，建议单个键值不超过100kb


String是redis中最基本的数据类型，它能存储任何形式的字符串，包括二进制数据。


#### (1) 命令




#### (2) 应用场景


##### ① 单值缓存


- 示例```redis
1) 存入
set name wzh

2) 查询
get name
```




##### ② 对象缓存


- 使用JSON序列化的方式
缺点：
   - 更新部分属性困难
   - 序列化开销


优点：

   - 简单易用
   - 节约内存


示例
​```redis
1) 将对象转换成json数据，进行存入
set user:1 {"name":"wzh","age":"17"}
```

- 使用多键存储的方式
优点：
   - 可部分更新：当你想更新对象中的某个值时，你可以针对单key取出来改值，不用取全部。


缺点：

   - 维护繁琐


示例
```redis
1) 将对象中的属性批量存入，来达到存入对象的目的
mset user:1:name wzh user:1:age 17

2) 批量查询
mget user:1:name user:1:age
```


##### ③ 分布式锁


- 使用setnx的方式
缺点：
   - 当服务器故障导致的业务中断时，因业务未完成，无法释放锁，就会造成死锁。


优点：

   - 简单易用


示例
```redis
1) 当setnx返回1时表示获取锁成功，返回0表示获取锁失败。
setnx product:1001 true		

2) 当执行完业务后需要释放锁。
del product:1001
```

- 使用set方式
优点：
   - 可以防止程序意外终止造成死锁


示例


1. 设置一个带过期时间的锁
set product:1001 true ex 10 nx



```


##### ④ 计数器

- 使用具有原子操作的incr命令的方式

示例：

​```redis
1) 阅读量加1，使用计数器做阅读量功能。
incr article:readcount:1001

2) 获取当前阅读量
get article:readcount:1001
```


##### ⑤ Web集群session共享


- SpringSession + redis 实现session共享



##### ⑥ 分布式系统全局序列号


- 使用incr的方式
缺点：
   - 每次生成id都要访问Redis，这样会大大增加Redis的负担。


示例
```redis
  1) 生成自增1后的id
  incr orderId
```

- 使用incrby的方式
优点：
   - redis批量生成序列号提升性能。使用incrby命令一次生成一个id区间，然后将不同批次的id区间分发给不同的JVM，由JVM具体生成ID，这样就减轻了Redis的压力。


缺点：

   - 如果其中一个Tomcat突然故障，那么保存在里面的id也就没了，造成的后果就是id不连续。


示例
```redis
  1) 生成一批id
  incrby orderId 1000
```


### 2. Hash（哈希）


#### (1) 命令


#### (2) 应用场景


##### ① 对象缓存


- 示例```redis
1) 存入一个用户对象，该用户的id为1，name为wzh，age为17。
hmset user 1:name wzh 1:age 17

2) 查询
hmget user 1:name 1:age
```




##### ② 电商购物车


- 示例```redis
1) 添加商品，以cart:{用户id}为key，商品id为field，商品数量为value。
hset cart:1 1001 1

2) 增加商品数量
hincrby cart:1 1001 1

3) 查询商品总数
hlen cart:1

4) 删除商品
hdel cart:1 1001

5) 获取用户购物车中的所有商品
hgetall cart:1
```




#### (3) 优缺点


- 优点：
同类数据归类整合储存，方便数据管理
相比string操作消耗内存与cpu更小
相比string储存更节省空间
- 缺点
过期功能不能使用在field上，只能用在key上
Redis集群架构下不适合大规模使用



### 3. List（列表）


#### (1) 命令


#### (2) 应用场景


##### ① 常用数据结构


- Stack(栈) =lpush + lpop		filo(先进后出)
- Queue(队列) = lpush + rpop           队列是先进先出的
- Blocking MQ(阻塞队列) = lpush + brpop



##### ② 消息流


微博在2014年那会用的就是Redis，内存容量达到几百T，但是还会经常崩。


- 缺点：
不推荐使用此方案，如果某个用户的粉丝量很大的话，给每个人都执行这样操作的话，会出大问题的。一般粉丝量是在一千以下的使用push模式的话，是没有问题的。粉丝如果多的话，可以使用pull模式，即大V将消息发送至单独的消息列表中，当粉丝进入页面时，就会主动去拉消息
- 示例```redis
1) 添加消息，以msg:{用户id}为key，{消息id}为value。
lpush msg:1 100012
lpush msg:1 100053

2) 查看最新消息
lrange msg:1 0 5
```




### 4. Set（集合）


#### (1) 命令


#### (2) 应用场景


##### ① 抽奖


- 示例```
1) 添加参与抽奖的用户，以raffle:{奖品id}为key，{用户id}为value
sadd raffle:1 1901

2) 查看参与抽奖的所有用户
smembers raffle:1

3) 抽取2名中奖者，如果有一二等奖的话，使用spop方式。
srandmember raffle:1 2
spop raffle:1 2
```




##### ② 点赞，收藏，标签


- 示例```redis
1) 点赞，以like:{消息id}为key，{用户id}为value
sadd like:100012 419

2) 取消点赞
srem like:100012 419

3) 检查用户是否点过赞
sismember like:100012 419

4) 获取点赞的用户列表
smembers like:100012

5) 获取点赞用户数
scard like:100012
```




##### ③ 集合操作


- 示例```redis
1) 交集，所有集合中共同拥有的值
sinter set1 set2 set3 

2) 并集，所有集合经过去重后的值
sunion set1 set2 set3

3) 差集，第一个集合在所有集合中独有的值
sdiff set1 set2 set3
```




##### ④ 关注模型


六度人脉指的就是你通过合适的选择最多通过6个人就可以认识他。


- 示例



1. 共同关注，交集
2. 我关注的人也关注他，
3. 我可能认识的人，差集



```
##### ⑤ 电商商品筛选

- 示例

​```redis
1) 添加商品规则
sadd brand:vivo iQOO3
sadd brand:xiaomi mi10
sadd brand:iPhone iPhone11
sadd os:Android mi10 iQOO3
sadd cpu:Qualcomm mi10 iQOO3

2) 筛选商品
sinter os:Android cpu:Qualcomm     ->  {iQOO3，mi10}
```


### 5.ZSet（有序集合）


#### (1) 命令


#### (2) 应用场景


##### ① 排行榜


- 示例

- ```redis
  1) 点击新闻
  zincrby hot:News:20200420 1 全球新冠肺炎感染人数近230万例
  2) 展示当日排行榜前十
  zrevrange host:News:20200420 0 10 WITHSCORES
  
  3) 七日搜索榜单计算
  zunionstore hot:News:20200413-20200420 7 hot:News:20200413 hot:News:20200414 hot:News:20200415 hot:News:20200416 hot:News:20200417 hot:News:202004138 hot:News:20200419 hot:News:20200420
  
  4) 展示七日排行前十
  zrevrange hot:News:20200413-20200420 0 10 WITHSCORES
  ```





## (二) 三大特殊数据类型


# 、持久化机制


**由来**：因为Redis的数据是以内存为载体来存储的，为了解决Redis停止服务，数据就会消失的问题。所以Redis提供了持久化机制，就是将内存中的数据保存到磁盘当中，避免数据意外丢失。


## (一) . RDB（快照持久化机制）


### 1. 保存


#### (1) 定义


将Redis这一时刻在内存中的数据集快照保存到rdb文件中，该文件是一个**被压缩过**的**二进制文件**。RDB是Redis默认的持久化方式


#### (2) 配置

```

```
dbfilename dump.rdb #rdb文件名称
dir ./	#文件生成目录。
```


#### (3) 快照时机


##### ① 手动快照


**执行`bgsave`命令**


当Redis接收到bgsave命令时，会调用fork函数创建一个当前进程的子进程，父进程继续处理客户端的命令请求，并等待子进程的完成信号；而子进程则调用rdbSave函数来将快照写入到磁盘中，完成后通知父进程保存完毕。


> fork函数由操作系统提供，它的作用是使系统创建一个当前进程的副本作为子进程。
> rdbSave函数是先把数据集先写入临时文件，写入成功之后，再替换之前的rdb文件。
> 父子进程先共享相同内存，当父进程或子进程对内存进行了写操作之后，对被写入的内存的共享才会结束服务



**执行`save`命令**


当Redis接收到save命令时，会调用rdbSave函数来将快照写入到磁盘中，并阻塞所有客户端（不处理客户端的任何请求），直到快照创建完毕。


**执行`shutdown`命令**


当Redis接收到shutdown命令时，会自动执行save命令，快照创建完毕后自动关闭Redis服务。


##### ② 自动快照


**设置配置文件中的触发规则自动间隔执行bgsave命令**


当满足配置文件中设置的任意一个save规则都将自动执行bgsave命令。


​```conf
save <seconds> <changes> #表示到seconds秒时，至少有changes次写操作，就会自动触发gbsave命令
```


#### (4) 缺点


如果Redis宕机，那么距离上一次持久化之后的数据都将丢失。


### 2. 载入


## (二) . AOF（只追加日志文件机制）


### 1. 保存


#### (1) 定义


将每个执行更改Redis数据的命令，都追加到AOF文件的末尾。


#### (2) 配置


```nginx
# 是否开启AOF持久化。默认为no。
appendonly yes

# aof文件的名称。默认是appendonly.aof
appendfilename "appendonly.aof"

# aof文件的保存位置和rdb文件的位置相同，都是通过dir参数设置的
dir ./

# 同步策略。
# always（立即同步）：将每一个执行更改Redis数据的命令，都立即同步到aof文件中。虽然保证数据不丢失，但是严重降低Redis速度。
# everysec（每秒同步）：将每一个执行更改Redis数据的命令，一秒后批量同步到aof文件中。虽然性能开销低，但是最多会丢失1s内的数据。
# no（随缘同步）：由操作系统决定何时同步。
appendfsync everysec

# AOF重写期间是否同步
no-appendfsync-on-rewrite no

# 重写触发配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 加载aof出错如何处理
aof-load-truncated yes

# 文件重写策略
aof-rewrite-incremental-fsync yes
```


> 如果使用固态硬盘可以有效提升aof性能，但是因为Redis会大量处理小文件，频繁的读写会使固态硬盘的寿命大大降低。



### 2. 重写机制


#### (1) 由来


为了解决因Redis执行的写命令的增多，在aof文件中产生冗余命令，而导致的文件体积庞大。所以Redis提供了AOF重写机制。


> 冗余命令包括过期数据的命令、无效的命令（被重复设置、已删除）、多个命令可合并为一个命令（批处理命令）。



#### (2) 重写机制


不是对原始文件进行压缩，而是通过读取服务器当前的数据库状态，来生成新的aof文件对原始的aof文件进行替换。


#### (3) 触发时机


##### ① 手动触发


**执行`bgrewriteaof`命令**


当Redis接收到bgrewriteaof命令时，会调用fork函数创建一个当前进程的子进程。此时会有两个进程，一个是父进程，一个是子进程。父进程继续处理客户端的命令请求，当接收到新的写命令时，会将新的写命令进行缓存并写入到原始的aof文件中；而子进程则会快照这一时刻在内存中的数据状态，然后将快照内容以命令的方式向写入到临时文件中，完成后通知父进程保存完毕。当父进程接收到子进程的完成信号时，会将缓存的写命令写入到临时文件中，然后再将临时文件重命名并替换原始aof文件。


进行


> 将写命令写入到原始的aof文件中是为了保证子进程重写失败而不出问题。



##### ② 自动触发


**设置配置文件中的触发规则自动间隔执行bgrewriteaof命令**


```
# 表示当AOF文件的体积大于64MB，且AOF文件的体积比上一次重写后的体积大了一倍（100%）时，会执行`bgrewriteaof`命令 
auto-aof-rewrite-percentage 100 #当前aof文件大小超过上一次重写时的aof文件大小的百分之多少时会再次进行重写。如果之前没有重写，则以启动时的aof文件大小为依据。
auto-aof-rewrite-min-size 64mb #允许重写的最小aof文件大小
```


# 操作Redis


## jedis


## SpringDataRedis


### 快速入门


#### 引入依赖


在 `pom.xml` 文件中，引入相关依赖。


```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <!-- 去掉对 Lettuce 的依赖，因为 Spring Boot 优先使用 Lettuce 作为 Redis 客户端 -->
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```


#### 配置文件


在 `application.yml` 中，添加 Redis 配置。


```yaml
#主从模式或单实例模式
spring:
  # 对应 RedisProperties 类
  redis:
    host: 192.168.30.136
    port: 6379
    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedisProperties.Jedis 内部类
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数，默认为 8 。使用负数表示没有限制。
        max-idle: 8 # 默认连接数最小空闲的连接数，默认为 8 。使用负数表示没有限制。
        min-idle: 0 # 默认连接池最小空闲的连接数，默认为 0 。允许设置 0 和 正数。
        max-wait: -1 # 连接池最大阻塞等待时间，单位：毫秒。默认为 -1 ，表示不限制。
        
#哨兵模式
spring:
  # 对应 RedisProperties 类
  redis:
    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedisProperties.Jedis 内部类
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数，默认为 8 。使用负数表示没有限制。
        max-idle: 8 # 默认连接数最小空闲的连接数，默认为 8 。使用负数表示没有限制。
        min-idle: 0 # 默认连接池最小空闲的连接数，默认为 0 。允许设置 0 和 正数。
        max-wait: -1 # 连接池最大阻塞等待时间，单位：毫秒。默认为 -1 ，表示不限制。
    sentinel:
      master: aa #哨兵名称
      nodes: 192.168.30.136:26379,192.168.30.136:26380,192.168.30.136:26381
```


#### 调用API


##### Lua 脚本执行器


###### scriptExecutor


```java
private @Nullable ScriptExecutor<K> scriptExecutor;
```


##### 数据结构操作类


```
opsForXxx().xxx
boundXxxOps("key").xxx（绑定key。即针对特定key进行操作）
```


### 序列化机制


#### JDK 序列化方式


##### JdkSerializationRedisSerializer


`org.springframework.data.redis.serializer.JdkSerializationRedisSerializer`


由`RedisTemplate#afterPropertiesSet()`方法可得，RedisTemplate的序列化方式默认采用的是JdkSerializationRedisSerializer。


```java
// java
@SpringBootTest
class DemoApplicationTests {
    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        redisTemplate.opsForValue().set("name","wzh");
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println(name);// 控制台输出：wzh。
	}
}

// redis客户端
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04name"
127.0.0.1:6379> get "\xac\xed\x00\x05t\x00\x04name"
"\xac\xed\x00\x05t\x00\x03wzh"
```


由`ObjectOutputStream#writeString(String str, boolean unshared)`方法可得，实际是标志位 + 字符串长度 + 字符串内容。所以才会看到具体字符串内容前会有一些十六进制字符。


#### String 序列化方式


##### StringRedisSerializer


`org.springframework.data.redis.core.StringRedisTemplate`


由`StringRedisTemplate#StringRedisTemplate()`方法（无参构造函数）可得，StringRedisTemplate的默认序列化是StringRedisSerializer。


由`StringRedisSerializer#deserialize(@Nullable byte[] bytes)`方法和`StringRedisSerializer#serialize(@Nullable String string)`方法可得，该序列化方式的序列化与反序列化其实就是字符串和二进制数组的相互转换。


Redis Client 传递给 Redis Server 是传递的 KEY 和 VALUE 都是二进制值数组。


#### JSON 序列化方式


##### Jackson2JsonRedisSerializer


`org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer`


```java
@RunWith(SpringRunner.class)
@SpringBootTest
class DemoApplicationTests {
    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        // 将数据反序列化为具体类时，需要传入具体的对象类型。
        // 将设置的对象序列化为json字符串。当设置为Object，表示所有对象都可以序列化，因为Object是所有类的基类。
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        //objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL);
        // 允许序列化时记录除了自然类型以外的非常量对象及属性的类型，以便反序列化时得到对应的具体对象。
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        // 设置日期格式化
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd"));
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        redisTemplate.setKeySerializer(StringRedisSerializer.UTF_8);
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(StringRedisSerializer.UTF_8);
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        User user = new User().setName("王责豪").setBirthday(new Date());
        redisTemplate.opsForValue().set("user",user);
    }

}

@Data
@Accessors(chain = true)
class User {
    private String name;
    private Date birthday;
}
```


#### XML 序列化方式


`org.springframework.data.redis.serializer.OxmSerializer` ，使用 Spring OXM 实现将对象和 String 的转换，从而 String 和二进制数组的转换。


没用过，可忽略。


# 、缓存问题


## (一) 缓存穿透


### 1. 定义


用户不断的请求一个在缓存和数据库中都不存在的数据，也就是无效数据。


### 2. 解决方案


1. 使用Nginx将单个IP每秒访问次数超出阈值的都拉黑。
   - 缺点：肉机足够多，就会被无视。
2. 缓存空对象。将查询到的无效数据加载到缓存中，将值设成空对象，而不是null。缓存有效时间最长不超过五分钟。
   - 缺点：只能解决大量访问同一个无效数据的问题。如果大量访问不同无效数据时，第一次仍然要查数据库的，那么请求就又落到数据库上了，假如数据库抗住了，又因为每个不同的无效数据被访问时，都会被缓存，然而这些数据没到失效时间，是不会被回收的，所以就会导致Redis中有大量的空数据，最终被撑爆。
3. 采用分布式锁或者互斥锁，只让一个请求落到数据库，其他请求自旋等待，当这个请求拿到数据并加载到缓存中时，在放开其他请求，此时它们会直接访问缓存。
4. 使用布隆过滤器。将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。
   - 缺点：请求无效数据，可能因为参数不合法造成的。布隆过滤器有误判的可能性，所以只能避免大部分的不同无效数据；可是当某个值被误判，那么该值就会一直被误判，因此这些漏网之鱼，可能会持续的大量访问同一个无效数据。
5. 在接口层增加校验。比如用户鉴权校验，参数校验，不合法的参数直接代码Return，比如：id 做基础校验，id <=0的直接拦截等。



### 3. 影响


导致数据库压力过大，严重会击垮数据库。


## (二) 缓存击穿


### 1. 定义


某个数据的缓存突然失效，但是此时涌来大量请求。


它其实就是缓存穿透的另一种形式。


### 2. 解决方案


1. 设置热点数据永远不过期。
2. 采用分布式锁或者互斥锁，只让一个请求落到数据库，其他请求自旋等待，当这个请求拿到数据并加载到缓存中时，在放开其他请求，此时它们会直接访问缓存。
3. 做一个兜底的本地缓存。



### 3. 影响


大量请求被发送到数据库上，轻则数据库出现阻塞的情况，重则出现数据库宕机的情况。


## (三) 缓存雪崩


### 1. 定义


大量数据的缓存同时失效，但是此时涌来大量请求。


### 2. 原因


如果大量数据的失效时间设置的过于集中，又刚好在失效的时间点上涌入大量用户。


### 3. 解决方案


1. 将key的过期时间分散开来，你可以把每个Key的失效时间都加个随机值，这样就可以保证数据不会在同一时间大面积失效。
2. 将key设置成永不过期，有更新操作就更新缓存就好了。（比如运维更新了首页商品，那你刷下缓存就完事了，不要设置过期时间）
3. 使用锁或者队列来保证不会有大量的线程对数据库一次性进行读写。
4. Redis使用集群，将热点数据均匀分布在不同的Redis库中



### 4. 影响


大量请求将全部落到数据库上，数据库无慢SQL并做了分库分表以及一些其他的措施，那么也可能只是出现短暂的卡顿。否则，数据库刚报一下警就直接挂了。其他依赖该库的接口也都会报错，如果没做熔断限流等策略就是直接挂一片，系统直接瘫痪。如果没有解决方案，就算重启数据库，那么数据库也会被新的流量打崩，因此你只能等待用户高峰过去之后再重启。


一般避免以上情况发生我们从三个时间段去分析下：


- 事前：Redis 高可用，主从+哨兵，Redis cluster，避免全盘崩溃。
- 事中：本地 ehcache 缓存 + Hystrix 限流+降级，避免MySQL 被打死。
- 事后：Redis 持久化 RDB+AOF，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。



# 、应用示例


## (一) 分布式锁


**锁是干嘛的？**


所谓的锁就是将并行执行的转换成串行执行的。


**什么情况下使用锁？**


只有满足共享资源，共享资源互斥，多任务环境时才需要使用锁。


**什么场景下使用分布式锁？**


电商秒杀抢购，接口幂等性


**为什么要使用分布式锁？**


JDK中的锁，只能解决JVM内部中的排队问题，而无法解决JVM之间的，所以这个时候我们使用分布式锁，来让不同的JVM去排队。分布式锁能够解决多个节点上多个进程之间的排队问题。


**死锁了怎么办？**


使用过期时间来解决死锁问题。如果设置的过长，可能会单位时间内不能进行更多的操作。如果设置的过短，可能会引发并发问题，如：超卖。


**会出现并发问题吗？**


Redis因为是个单线程模型，因此不管多少个请求，它都会按照先后到达顺序的来排队，然后在服务端依次去执行


### 1. setnx


**1.0**


- 缺点：如果Tomcat在finally块执行之前突然宕机，导致锁无法被释放，那么其他的Tomcat也就无法上锁，所以就会造成死锁。
- 解决方案：设置键过期时间
- 示例```java
@Controller
public class RedisLock {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 减库存
     * @return
        */
      @RequestMapping("/deduct_stock")
      public String deductStock() {

        String productId = "1000001";
        // 上锁
        Boolean result = stringRedisTemplate.opsForValue().setIfAbsent(productId,"true");
        if (!result) {
            return "当前抢购的人数过多，服务器繁忙，请稍后再试！";
        }

        try {
            // 检查库存是否够量
            int stock = Integer.parseInt(stringRedisTemplate.opsForValue().get("stock"));
            if(stock > 0) {
                // 减库存
                int realStock = stock - 1;
                stringRedisTemplate.opsForValue().set("stock", realStock+"");
                System.out.println("扣减成功！剩余库存：" + realStock);
            } else {
                System.out.println("扣减失败！库存不足！");
            }
        }finally {
            stringRedisTemplate.delete(productId);
        }
        return "恭喜您！抢购到了该商品！";
      }
}
```




**2.0**


- 缺点：如果Tomcat在设置键过期之前突然宕机，导致锁无法被释放，那么其他的Tomcat也就无法上锁，所以就会造成死锁。
- 解决方案：将上锁与设置键过期的操作合并
- 示例```java
@Controller
public class RedisLock {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 减库存
     * @return
     */
    @RequestMapping("/deduct_stock")
    public String deductStock() {

        String productId = "1000001";
        // 上锁
        Boolean result = stringRedisTemplate.opsForValue().setIfAbsent(productId,"true");
        // 设置键过期时间
        stringRedisTemplate.expire(productId, 10, TimeUnit.SECONDS);
        if (!result) {
            return "当前抢购的人数过多，服务器繁忙，请稍后再试！";
        }

        try {
            // 检查库存是否够量
            int stock = Integer.parseInt(stringRedisTemplate.opsForValue().get("stock"));
            if(stock > 0) {
                // 减库存
                int realStock = stock - 1;
                stringRedisTemplate.opsForValue().set("stock", realStock+"");
                System.out.println("扣减成功！剩余库存：" + realStock);
            } else {
                System.out.println("扣减失败！库存不足！");
            }
        }finally {
            stringRedisTemplate.delete(productId);
        }
        return "恭喜您！抢购到了该商品！";
    }
}
```




**3.0**


- 缺点：当第一个线程的业务未在键过期时间之内完成时，锁会被Redis释放，然后由第二个线程持有，这样会造成第一个线程与第二个线程同时执行，当第一个线程执行完后会去释放锁，结果把第二个线程的锁给释放了，然后第三个线程的锁被第二个线程释放了，当前线程的锁不是被自己释放的，而是被上一个锁持有者释放的，这样就会造成锁失效的问题。
- 解决方案：加入唯一标识，自己只能释放自己的锁。将值设为唯一标识，当删除键值时（释放锁），验证保存的值是否跟当前线程保存的唯一标识是否一致，如果一致，才可以删除。
- 示例```java
@Controller
public class RedisLock {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 减库存
     * @return
        */
      @RequestMapping("/deduct_stock")
      public String deductStock() {

        String productId = "1000001";
        // 上锁与设置键过期合并
        Boolean result = stringRedisTemplate.opsForValue().setIfAbsent(productId,"true",10, TimeUnit.SECONDS);
        if (!result) {
            return "当前抢购的人数过多，服务器繁忙，请稍后再试！";
        }

        try {
            // 检查库存是否够量
            int stock = Integer.parseInt(stringRedisTemplate.opsForValue().get("stock"));
            if(stock > 0) {
                // 减库存
                int realStock = stock - 1;
                stringRedisTemplate.opsForValue().set("stock", realStock+"");
                System.out.println("扣减成功！剩余库存：" + realStock);
            } else {
                System.out.println("扣减失败！库存不足！");
            }
        }finally {
            stringRedisTemplate.delete(productId);
        }
        return "恭喜您！抢购到了该商品！";
      }
}
```




**4.0**


- 缺点：如果锁的过期时间设置的过高，当Tomcat宕机了，就需要让其他等很久。如果设置的过短了，就会造成业务未执行完，锁就被释放了。
- 解决方案：如果当前线程拿到锁，就开启一个分线程，在分线程里面使用定时器去扫描当前线程的锁是否还存在，如果还存在，就重置键的过期时间。当主线程消失，那么分线程也会消失。
- 示例```java
@Controller
public class RedisLock {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 减库存
     * @return
     */
    @RequestMapping("/deduct_stock")
    public String deductStock() {

        String productId = "1000001";
        String clientId = UUID.randomUUID().toString();
        // 上锁与设置键过期合并
        Boolean result = stringRedisTemplate.opsForValue().setIfAbsent(productId,clientId,10, TimeUnit.SECONDS);
        if (!result) {
            return "当前抢购的人数过多，服务器繁忙，请稍后再试！";
        }

        try {
            // 检查库存是否够量
            int stock = Integer.parseInt(stringRedisTemplate.opsForValue().get("stock"));
            if(stock > 0) {
                // 减库存
                int realStock = stock - 1;
                stringRedisTemplate.opsForValue().set("stock", realStock+"");
                System.out.println("扣减成功！剩余库存：" + realStock);
            } else {
                System.out.println("扣减失败！库存不足！");
            }
        }finally {
            // 验证是否是自己加的锁
            if (clientId.equals(stringRedisTemplate.opsForValue().get(productId))){
                stringRedisTemplate.delete(productId);
            }
        }
        return "恭喜您！抢购到了该商品！";
    }
}
```




### 2. redisson


- redisson加锁的默认过期时间是30秒，redisson自己生成客户端的线程id作为value，获取锁对象时为redisson提供的name作为key。
- 只有拿到锁的线程，才能继续执行，其他未拿到锁的线程将自旋等待，当该线程释放锁后，其他线程将争抢该锁。
- redisson底层其实就是lua脚本
- 缺点：当使用主从架构时，可能在主节点中刚设置完锁，还没来得及同步到从节点中时，主节点挂了，那么从节点变为主节点，因为从节点中没有该数据，那么其他线程进来时，就可以上锁成功，这样就又出现了锁失效的问题（资源共享访问的问题）。
- 解决方案：使用ZooKeeper，当你把锁加到了主节点上，它不会马上返回告诉你加锁成功，而是让集群里面的大多数节点都同步成功了，才会返回给用户加锁成功，当主节点挂了，它会保证一定是从已经同步了这把锁的那个节点里面选出一个新的主节点。但是ZooKeeper的性能没有Redis的高。或者是是用Redlock。
- redlock实现原理



- redisson实现原理



- 示例```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Bean
    public Redisson redisson () {
        Config config = new Config();

        // 此为单机模式
        config.useSingleServer().setAddress("redis://localhost:6379").setDatabase(0);
    
        // 集群模式
        /*config.useClusterServers()
                .addNodeAddress("")
                .addNodeAddress("")
                .addNodeAddress("")
                .addNodeAddress("");*/
        return (Redisson) Redisson.create(config);
    }
}



@Controller
public class RedisLock {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    private Redisson redisson;

    /**
     * 减库存
     * @return
     */
    @RequestMapping("/deduct_stock")
    public String deductStock() {
        String productId = "1000001";
        String clientId = UUID.randomUUID().toString();
    
        // 生成锁对象。在Redis中获取一个锁对象，此时并没有加锁。
        RLock redissonLock = redisson.getLock(productId);
    
        try {
            // 加锁
            redissonLock.lock();
            // 检查库存是否够量
            int stock = Integer.parseInt(stringRedisTemplate.opsForValue().get("stock"));
            if(stock > 0) {
                // 减库存
                int realStock = stock - 1;
                stringRedisTemplate.opsForValue().set("stock", realStock+"");
                System.out.println("扣减成功！剩余库存：" + realStock);
            } else {
                System.out.println("扣减失败！库存不足！");
            }
        }finally {
            // 释放锁
            redissonLock.unlock();
        }
        return "恭喜您！抢购到了该商品！";
    }
}
```




## (二) 分布式库存


使用分布式库存的方案，减少单个key的访问次数。


## (三) 分布式Session


利用了Redis的单线程的特性，在跨Tomcat之间不会存在并发情况。


**引入依赖**


​```xml
<!-- 全局session管理 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```


**启动类追加注解**


```java
@EnableRedisHttpSession(redisNamespace = "redis:session",maxInactiveIntervalInSeconds = 3600*,) // 开启Redis的全局Session管理
maxInactiveIntervalInSeconds: 设置 Session 失效时间。默认时间为1800秒，也就是30分钟。使用 Redis Session 之后，原 Spring Boot 的 server.session.timeout 属性不再生效。
redisNamespace：配置key的前缀。默认是spring:session，如果不同的应用共用一个redis，应该为应用配置不同的namespace，这样才能区分这个Session是来自哪个应用的
flushMode：配置刷新Redis中Session的方式，默认是ON_SAVE模式。
    ON_SAVE：只有当Response提交后才会将Session提交到Redis。
    IMMEDIATE：所有对Session的更改会立即更新到Redis。
cleanupCron：清理过期Session的定时任务。默认一分钟一次。使用Cron表达式。
```


**Memcache与Redis管理session机制的区别？**


Memcache


- Memcache是与Tomcat整合，是全局的session管理机制，也就是整个服务器中所有应用全部交由Memcache管理。
- Memcache仅仅是做一个session的备份，真正做session管理的还是Tomcat。当Tomcat创建完session之后Memcache会备份一下。当负载均衡到另外一个Tomcat上时，Memcache会把备份的session复制到该Tomcat中。当下一次负载均衡还是到该Tomcat中，那么Memcache将不会复制session复制进来，因为已经有了该session。
- 如果由Tomcat管理session，因为session是保存在JVM内存中的，每次操作都是操作它本身，所以只需第一次请求时保存session。



session


- Redis是与应用整合，是单一应用的session管理，也就是整个服务器中的某些应用交由Redis管理。
- 当Tomcat创建了session，会将session保存进Redis中，每次请求都需要去Redis中获取session，因此是由Redis来管理session。
- 如果由Redis管理session，因为session是保存在Redis中的，每次操作都是操作它的副本，所以必须每次请求都保存session。



## (四) 分布式缓存


### 缓存


**什么是缓存**


缓存就是存在计算机内存中的一段数据。


**为什么要使用缓存**


使用缓存可以提高网站的响应速度。减少请求数据库的次数。


**什么情况下建议使用缓存**


建议在查询多，修改少的情况下使用缓存。因为每次修改后需要将修改后的数据同步到缓存中，修改次数小还可以接受，如果修改次数多便是个灾难。


#### 本地缓存


**什么是本地缓存**


本地缓存就是存储在当前运行的jvm内存中的数据。


例如mybatis的二级缓存就是本地缓存。使用方式就是在mapper文件中加入标签即可。


**本地缓存的缺点**


程序停止，缓存消失。


如果缓存数据过多，占用的jvm内存也多，也就会导致应用运行缓慢。


不能在分布式系统中做到缓存共享。


#### 分布式缓存


**使用分布式缓存的好处是什么？**


本地缓存的缺点，分布式缓存都进行了完善。


**mybatis的分布式缓存**


实现类


```java
public class RedisCache implements Cache {// 在自定义的类中获取工厂对象
    /**
     * 如何设计？
     * 其实我们到项目做完之后才考虑到缓存的，因为这属于项目优化层面，我们在一开始编码的时候，并没有考虑到缓存。
     * 后来把整个项目做完之后，我们发现项目中有些模块的业务数据是不需要经常发生变化的，所以这个时候为了减轻数
     * 据库的压力，同时提高程序的运行效率。我们就对整个项目中的某些模块做了缓存。当时做缓存的时候比较头疼，因
     * 为我们当时做缓存的时候就考虑到mybatis自身的缓存，但是mybatis的缓存是本地缓存，它是存在jvm内存中的，
     * 因此如果时间久了，数据量大了，那么就会导致程序运行变慢，而且不能在分布式系统中共享缓存。后来我们就想，
     * 能不能将他的缓存改写或者做一个进一步的设计，让它现有的缓存实现是基于分布式缓存的。因为我们项目用到了
     * Redis，而且Redis它本身是对内存型这样的场景，如果我们要把缓存放到Redis中里中，可以保证缓存不在jvm里面，
     * 同时又能实现分布式系统的缓存共享。所以后来我们就想，能不能把mybatis自身的缓存放到Redis中。后来通过追
     * 源码得知Cache标签默认使用Cache接口的PerpetualCache实现，它是通过map来存储缓存的。因为得知有个Cache
     * 接口然后我们就想自定义一个实现类，然后在cache标签指定类型为这个实现类。这样我们就给mybatis实现了分布式
     * 缓存。
     * 这样mybatis在实现缓存时会像自身的接口给传一个namespace和查询的key以及查询的结果，这样我们就可以自定义、
     * 实现类使用Redis做实现。
     */

    /**
     * mybatis缓存中如何保证同一个方法多次查询时key始终一致？
     * 通过追源码得知mybatis它提供了CacheKey类。
     * 它之所以能保证每次生成的是一样的，是因为它的生成规则，
     * 它的生成规则时mapper的namespace的hashcode + 调用方法的完全限定名 + sql语句。
     */

    /**
     * 缓存模型如何设计的？
     * 我们在设计时使用的是双重hash结构，就是region（分区）
     * 我们最初的时候就是使用redisTemplater，把缓存以键值的形式直接存进Redis，但后来发现在Redis散列了很多的key，
     * 而且不同模块的key都放在了一起，因此我们在清除的时候遇到了比较头疼的问题，就是我只想清楚当前模块的缓存，但是
     * 同时会把其他模块的缓存一并清掉，所以最初的缓存模型设计的并不好。后来我们把这个缓存模型改造了一下，就是基于
     * region分区结构的，我们是利用了Redis的hash数据结构，我们将当前mapper的namespace作为key，当前mapper的方
     * 法名作为field，查询结果作为value。当我们想清除某个模块的缓存时，只需删除对应的key即可。
     */

    /**
     * 怎么解决的缓存穿透问题？
     * mybatis集成Redis缓存，其实自身已经解决缓存穿透问题。
     * 它不管在数据库中查没查到都会回写至Redis，只不过值是否为null罢了。
     * 当进行增删改操作时，mybatis会自动清除缓存，所以不用担心缓存数据与数据库数据不匹配的问题。
     */

    /**
     * 如何解决缓存雪崩问题？
     * 我们现有的解决方案是缓存数据永久存储。
     * 因为我们现在这个项目的业务模块并不是特别多，业务数据量也不大，我们经常被使用的缓存基本都是永久存储。
     *
     * 您说的这个问题，我们当时也考虑到了。
     * 可以根据不同的业务数据，设置不同的失效时间
     */
    private String id;
    public RedisCache() { }

    /**
     * 构造函数
     * @param id 该值为调用RedisCache对象的mapper的namespace（完全限定名）。
     */
    public RedisCache(String id) {
        System.out.println("当前加入缓存的namespace"+id);
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    // 写入某缓存
    @Override
    public void putObject(Object key, Object value) {
        System.out.println("key："+key);
        System.out.println("value:"+value);
        // 获取对象
        RedisTemplate redisTemplater = (RedisTemplate) ApplicationContextUtil.getBean("redisTemplater");
        // 设置序列化
        redisTemplater.setKeySerializer(RedisSerializer.string());
        redisTemplater.setHashKeySerializer(RedisSerializer.string());
        // 存储缓存数据
        redisTemplater.opsForHash().put(id,key.toString(),value);
        // 根据不同的业务模块，设置不同的超时时间
        if ("com.example.demo.mapper.UserMapper".equals(id)){
            redisTemplater.expire(id,10, TimeUnit.DAYS);
        }
        if ("com.example.demo.mapper.EmpMapper".equals(id)){
            redisTemplater.expire(id,10, TimeUnit.MINUTES);
        }

    }

    // 获取某缓存
    @Override
    public Object getObject(Object key) {
        // 获取对象
        RedisTemplate redisTemplater = (RedisTemplate) ApplicationContextUtil.getBean("redisTemplater");
        // 设置序列化
        redisTemplater.setKeySerializer(RedisSerializer.string());
        redisTemplater.setHashKeySerializer(RedisSerializer.string());
        // 获取缓存数据
        Object value = redisTemplater.opsForHash().get(id, key.toString());
        return value;
    }

    // 删除某缓存
    @Override
    public Object removeObject(Object o) {
        System.out.println("删除缓存");
        return null;
    }

    // 清空缓存
    @Override
    public void clear() {
        // 获取对象
        RedisTemplate redisTemplater = (RedisTemplate) ApplicationContextUtil.getBean("redisTemplater");
        // 设置序列化
        redisTemplater.setKeySerializer(RedisSerializer.string());
        redisTemplater.setHashKeySerializer(RedisSerializer.string());
        // 删除当前mapper的所有缓存
        redisTemplater.delete(id);
    }

    // 缓存命中率计算
    @Override
    public int getSize() {
        // 获取对象
        RedisTemplate redisTemplater = (RedisTemplate) ApplicationContextUtil.getBean("redisTemplater");
        // 设置序列化
        redisTemplater.setKeySerializer(RedisSerializer.string());
        redisTemplater.setHashKeySerializer(RedisSerializer.string());
        // 获取当前mapper的所有缓存的数量
        int size = redisTemplater.opsForHash().size(id).intValue();
        return size;
    }

    // 读写锁  ReadWriteLock：读读不互斥 读写不互斥 写写互斥。Synchronized：读读互斥 写写互斥 读写互斥。ReadWriteLock比Synchronized的性能高
    @Override
    public ReadWriteLock getReadWriteLock() {
        return new ReentrantReadWriteLock();
    }
}
```


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mapper接口的完全限定名">
    <!--cache跟cache-ref不同出现在同一个mapper文件中。一般在外键这边写cache-ref，主键这边写cache-->
    <!-- 使用缓存 -->
    <cache type="自定义cache的完全限定名"/>
    <!-- 引用其他模块共同使用当前缓存。就是让多个有关联关系的DAO共享同一个缓存 -->
    <cache-ref namespace="mapper接口的完全限定名">
    <select id="方法名" resultType="类型的完全限定名">
		sql语句
    </select>
</mapper>
```


## (五) 分布式ID


## (六) 计数器


# 、集群


这里演示使用的是伪分布式。


## () 主从模式


### 定义


主从模式就是，由一个主节点和一个或多个从节点组建而成的Redis集群。


### 核心


主从模式的核心是主从复制。主从复制就是将主节点(master)的数据实时同步到从节点(slave)中。主从复制支持主从同步和从从同步两种。


### 作用


- **数据备份：**通过主从复制实现数据的热备份，是持久化之外的一种数据冗余方式。
- **负载均衡：**在主从复制的基础上，配合读写分离，就是由主节点提供写服务，由从节点提供读服务 ，来分担服务器负载。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。



### 地位


主从复制是哨兵和集群能够实施的基础，所以说主从复制是 Redis 高可用的基石。


### 缺陷


无法自动完成故障转移。


### 快速体验


#### 方式


主从复制的开启，完全是由从节点发起的，不需要我们在主节点做任何事情。


##### 配置文件


在配置文件中加入`replicaof <masterip> <masterport>`，那么当前节点就会作为指定节点的从节点。


```shell
1) 选择位置
cd /usr/local/

2) 创建文件夹
mkdir redis-master_slave 
cd redis-master_slave
mkdir 7001 7002

3) 复制配置文件
cp redis.conf /usr/local/redis-master_slave/7001/
cp redis.conf /usr/local/redis-master_slave/7002/

4) 修改配置文件
	1) 主节点
		1) 指定端口（分别对每个机器的端口进行设置）
	    port 7001
    	2) 允许后台运行
	    daemonize yes
    	3) 绑定主机，当设置为0.0.0.0时表示所有请求都可以接受。
  		bind 0.0.0.0
	2) 从节点
		1) 指定端口（分别对每个机器的端口进行设置）
	    port 7002
    	2) 允许后台运行
	    daemonize yes
    	3) 绑定主机，当设置为0.0.0.0时表示所有请求都可以接受。
  		bind 0.0.0.0
  		4) 指定主节点
		replicaof 192.168.30.136 7001

5) 启动
./redis-server /usr/local/redis-master_slave/7001/redis.conf
./redis-server /usr/local/redis-master_slave/7002/redis.conf

6) 查看是否成功
./redis-cli -p 7001
127.0.0.1:7001> info 
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.30.136,port=7002,state=online,offset=56,lag=0
```


##### 启动命令


在`redis-server`启动命令后加入`--slaveof <masterip> <masterport>`，那么当前节点就会作为指定节点的从节点。


```
./
```


##### 客户端命令


在Redis客户端中执行`slaveof <masterip> <masterport>`命令，那么当前节点就会作为指定节点的从节点。


#### 效果展示


```shell
1) 在从节点中查询一个不存在的key：
127.0.0.1:7002> get name
(nil)

2) 在主节点添加该key
127.0.0.1:7001> set name wzh
OK

3) 在主节点查询该key
127.0.0.1:7001> get name
"wzh"

4) 在从节点查询该key
127.0.0.1:7002> get name
"wzh"
```


### 断开关系


#### 方式


##### 客户端命令


在Redis客户端中执行`slaveof no one`命令，那么当前节点就会放弃作为其他节点的从节点。


```shell
1) 从节点断开关系
127.0.0.1:7002> slaveof no one
OK
```


#### 效果展示


```shell
1) 在主节点添加一个key
127.0.0.1:7001> set name2 wzh2
OK

2) 在主节点查询该key
127.0.0.1:7001> get name2
"wzh2"

3) 在从节点查询该key
127.0.0.1:7002> get name2
(nil)

4) 在从节点查询放弃关系之前的key
127.0.0.1:7002> get name
"wzh"
```


由此得知，从节点断开关系后，不会删除已有的数据，只是不再接受主节点新的数据变化。


### 实现原理


## () 哨兵模式


### 定义


所谓的哨兵模式，不过就是带有自动处理故障转移功能的主从模式。


### 核心


哨兵模式的核心是哨兵机制。哨兵(Sentinel)是用于监控主节点状态的工具，目前已被集成在redis2.4+的版本中。


### 作用


- **监控：**哨兵会定时检查主节点和从节点的运行状态。
- **故障转移：**如果主节点异常，哨兵就会把该主节点中的某一台从节点升为新的主节点，当失效的主节点恢复后沦为新的主节点的从节点。
- **配置提供者：**客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。
- **通知：**哨兵可以将故障转移的结果发送给客户端。



### 缺陷


哨兵的配置略微复杂；性能和高可用性等各方面表现一般；主从切换的瞬间，存在访问瞬断的情况；无法解决现有系统的物理上限问题，物理上限就是随着时间的推移，数据增多，对内存要求变高。


### 快速体验


#### 方式


##### 配置文件方式


```
修改配置文件
	1) 7101
	
	
	2) 7102
	
	
	3) 7103
```


### 启动方式


方式一


```
redis-sentinel sentinel.cof
```


方式二


```
redis-server sentinel.conf --sentinel
```


注意：在启动某个哨兵实例时必须使用配置文件，因为系统将使用此文件来保存当前状态，以便在重启时重新加载。如果未提供配置文件或配置文件路径不可写，哨兵将会拒绝启动。


### 建议


1. 一个健壮的部署至少需要三个哨兵实例。
1. 应将不同的哨兵实例放置到不同的机器中
## () 集群模式
### 定义
所谓的Redis集群就是有很多个Redis节点。


Redis集群是一个由多个主从节点组成的分布式服务器群，它具有复制、高可用和分片特性。Redis集群不需要sentinel哨兵也能完成节点移除和故障转移的功能。需要将每个节点设置成集群模式，这种集群模式没有中心节点，可水平扩展，据官方文档称可以线性扩展到1000节点。Redis集群的性能和高可用性均优于之前版本的哨兵模式，且集群配置非常简单。


服务端通过jedisCluester集群组件去访问。当你set name的请求通过jedisCluester的负载均衡策略被发送到了主从1中，而当你发送get name请求被分到了主从2中，那么Redis内部就会自动帮你重定向到主从1中去读对应的数据。


在Redis3.0以前的版本要实现集群一般是借助哨兵sentinel工具来监控master节点的状态，


### 搭建


redis集群至少要三个master集群，我们这里搭建三个master节点，并且给每个master再搭建一个slave节点，总共6个redis节点，由于节点数较多，这里采用在一台机器上创建6个redis实例，并将这6个redis实例配置成集群模式，所以这里搭建的是伪集群模式，当然真正的分布式集群的配置方法几乎一样。


```shell
1) 选择位置
cd /usr/local

2) 创建文件夹
mkdir redis-cluster 
cd redis-cluster
mkdir 8001 8002 8003 8004 8005 8006

3) 复制配置文件
cp redis.conf /usr/local/redis-cluster/8001

4) 修改配置文件
vim redis-conf
    1) 指定端口（分别对每个机器的端口进行设置）
    prot 8001
    2) 允许后台运行
    daemonize yes
    3) 绑定主机（方便Redis集群定位机器，如果不绑定可能出现循环查找集群节点的情况）
    bind 192.168.30.135
    4) 指定数据文件存放位置（如果不指定会丢数据）
    dir /usr/local/redis-cluster/8001/
    5) 开启集群模式
    cluster-enabled yes
    6) 指定配置文件（这里的800x要和port对应上）
    cluster-config-file nodes-8001.conf
    7) 指定心跳超时时间
    cluster-node-timeout 5000
    8) 开启日志模式
    appendonly yes
:x 
 
5) 复制配置文件
cp /usr/local/redis-cluster/8001/redis.conf /usr/local/redis-cluster/8002/
cp /usr/local/redis-cluster/8001/redis.conf /usr/local/redis-cluster/8003/
cp /usr/local/redis-cluster/8001/redis.conf /usr/local/redis-cluster/8004/
cp /usr/local/redis-cluster/8001/redis.conf /usr/local/redis-cluster/8005/
cp /usr/local/redis-cluster/8001/redis.conf /usr/local/redis-cluster/8006/

6) 批量替换配置文件中的端口号
	1) 8002
    vim /usr/local/redis-cluster/8002/redis.conf
    :%s/8001/8002/g
    :x
    2) 8003
    vim /usr/local/redis-cluster/8003/redis.conf
    :%s/8001/8003/g
    :x
    3) 8004
    vim /usr/local/redis-cluster/8004/redis.conf
    :%s/8001/8004/g
    :x
    4) 8005
    vim /usr/local/redis-cluster/8005/redis.conf
    :%s/8001/8005/g
    :x
    5) 8006
	vim /usr/local/redis-cluster/8006/redis.conf
    :%s/8001/8006/g
    :x
    
7) 启动服务
/usr/local/redis-4.0.11/src/redis-server /usr/local/redis-cluster/8001/redis.conf
/usr/local/redis-4.0.11/src/redis-server /usr/local/redis-cluster/8002/redis.conf
/usr/local/redis-4.0.11/src/redis-server /usr/local/redis-cluster/8003/redis.conf
/usr/local/redis-4.0.11/src/redis-server /usr/local/redis-cluster/8004/redis.conf
/usr/local/redis-4.0.11/src/redis-server /usr/local/redis-cluster/8005/redis.conf
/usr/local/redis-4.0.11/src/redis-server /usr/local/redis-cluster/8006/redis.conf

8) 检查是否启动成功
ps -ef|grep redis

9) 安装 ruby（由于Redis集群需要使用Ruby命令）
yum install ruby

10) 安装 rubygems 组件
yum install rubygems

11) 安装redis 的接口
gem install redis --version 3.3.5

12) 创建集群
/usr/local/redis-4.0.11/src/redis-trib.rb create --replicas 1 192.168.30.135:8001 192.168.30.135:8002 192.168.30.135:8003 192.168.30.135:8004 192.168.30.135:8005 192.168.30.135:8006
	create --replicas后面的1表示的是master与slave的比值，即master/slave的值
	
	1) 返回结果：
    Using 3 masters:
    192.168.30.135:8001
    192.168.30.135:8002
    192.168.30.135:8003
    Adding replica 192.168.30.135:8005 to 192.168.30.135:8001
    Adding replica 192.168.30.135:8006 to 192.168.30.135:8002
    Adding replica 192.168.30.135:8004 to 192.168.30.135:8003
    由此得出，先从左到右的顺序先选出master，剩下的节点作为slave在保证比值的情况下随机分配给master

13) 连接客户端
/usr/local/redis-4.0.11/src/redis-cli -c -h 192.168.30.135 -p 8001
-c 表示集群模式 -h 指定ip地址 -p 指定端口号

14) 查看信息
	1) 查看集群信息 
	cluster info 
	2) 查看节点列表
	cluster nodes
	
15) 添加数据
192.168.30.135:8001> set name1 wzh1
-> Redirected to slot [12933] located at 192.168.30.135:8003
OK
192.168.30.135:8003> set name2 wzh2
-> Redirected to slot [742] located at 192.168.30.135:8001
OK
192.168.30.135:8001> set name3 wzh3
OK
192.168.30.135:8001> set name4 wzh4
-> Redirected to slot [8736] located at 192.168.30.135:8002
OK
192.168.30.135:8002> set name5 wzh5
-> Redirected to slot [12801] located at 192.168.30.135:8003
OK
由此得知，为了保证集群尽量不出现数据倾斜的问题，其内部做了一些负载均衡的策略。

16) 关闭集群（服务挨个进行关闭）
/usr/local/redis-4.0.11/src/redis-cli -c -h 192.168.30.135 -p 800* shutdown

当集群出现无法启动时，删除临时的数据文件，再次重新启动每一个Redis服务，然后重新构造集群环境
```


# 布隆过滤器


### 定义


bloomFilter可以理解为Java中的ArrayList，都是专门存东西的。


布隆过滤器本质是一个位数组。位数组就是数组的每个元素都只占用 1 bit ，每个元素只能是 0 或者 1。当位数组长度为一百亿时，占用空间1.16G。


布隆过滤器存储的是数据指纹，不存储数据本身，这在实际上进行了压缩空间。因此它只能添加或者判断是否存在，而不能修改或删除。如果只是少部分垃圾，不需要重建布隆过滤器。


当往布隆过滤器中插入数据时，该数据会被不同的哈希函数计算出不同的哈希函数值，这些哈希函数值就是位数组中的下标，当把这些下标位置的元素改为1时，那么就表示该数据在布隆过滤器中已被标识。


当要判断一个值是否在布隆过滤器中时，首先会对元素进行哈希计算，得到全部下标后判断对应位置的元素是否为1，只有全部为1才有可能存在，只要有一个不为1就绝对不存在。


误判是由哈希碰撞导致的。误判概率跟数组长度与哈希函数的个数有关。哈希函数的个数以及位数组的长度是由预计插入量以及误判率来决定的。误判率设置的越低，性能开销就越大。当某个值被误判，那么该值就会一直被误判。


哈希算法把数据（可以是任意类型的）算成数字，然后将算出来的数字%（模，取余）位数组的长度，就会得到一个下标。


一般根据更新数据的频率，写定时任务，来维护布隆过滤器。


### 优缺点


缺点：


```
1.存在误判，返回的结果是概率性的。

2.值无法修改与删除。
```


优点：


```
1.相比于传统的 List、Set、Map 等数据结构，它更高效且占用空间更少。
```


在Java中由guava提供的布隆过滤器，受Java中的int类型最大值影响，所以位数组最大是21亿，它是占用的JVM的内存，而且数据不会被持久化。如果使用Redis，因受Redis中的String类型最大容量影响，所以它提供的位数组长度最大是42亿，它占用的是Redis的内存，数据支持持久化。如果要分布式部署的话，不建议用Java的，因为每个JVM的空间不共享，每次数据的更新都要给所有的更新，内容基本是重复的，这就造成了空间的大量浪费。


字符串类型的底层保存的是二进制数据，Redis底层采用位数组来保存二进制数据。


bitmaps（位图）是String类型扩展的数据结构。它就是用位数组保存的二进制值。使用setbit，长度越界时，它会自动在原二进制数据后面进行补位，直到满足越界位数。


# Redis单线程与多线程的选择


Redis官方表示Redis的瓶颈不是cpu的运行速度，也不是机器的内存大小，而是网络I/O。因为单线程实现很简单所以采用的单线程而不是因为单线程性能高而使用单线程。如果你非要跟我说原子操作的单线程确实比有资源竞争的多线程快，那么我只能说大家直接写单线程后端好了，避免上下文切换开销。其实对比发现采用多线程的memcached比采用单线程的redis更快。因为在Redis6.0 引入了多线程 IO，以至于性能提升数倍。单线程模型的Redis之所以快就是因为内存数据库没有文件IO以及非阻塞IO带来的高性能网络通信。多线程可以发挥多核优势，除非热点极端集中，大多数情况下多线程优势明显。


rdb 持久化和 aof 重写是 fork 的子进程做的。全量同步 sync 应该也是


Redis6.0 的多线程部分只是用来处理网络I/O和协议解析，执行命令仍然是单线程。Memcached 则是从 IO 处理到数据访问都是多线程实现。所以说性能瓶颈在网络IO读写。


redis6的多线程只是针对客户端这一块，通过系统调用写操作，将客户端的输入输出缓冲中的数据通过多线程IO与客户端交互。这部分通常能够占到CPU负载的50%，将这部分通过其他线程进行处理，核心流程还是单线程。如果真正想发挥多核性能，还是得用集群。


redis做持久化可采用rdb+aof 双重快照，这样理论上能保证数据100%完整性，但是redis性能会受影响，建议不要用redis做持久化，redis快就是因为使用的内存数据，在架构中定义为缓存。持久化使用mongo


# 常用的使用场景


缓存，一些频繁被访问的数据，经常被访问的数据如果放在关系型数据库，每次查询的开销都会很大，而放在redis中，因为redis 是放在内存中的可以很高效的访问。因此在提升服务器性能方面非常有效。


排行榜，在使用传统的关系型数据库（mysql oracle 等）来做这个事儿，非常的麻烦，而利用Redis的SortSet(有序集合)数据结构能够简单的搞定；


计算器/限速器，利用Redis中原子性的自增操作，我们可以统计类似用户点赞数、用户访问数等，这类操作如果用MySQL，频繁的读写会带来相当大的压力；限速器比较典型的使用场景是限制某个用户访问某个API的频率，常用的有抢购时，防止用户疯狂点击带来不必要的压力；


好友关系，利用集合的一些命令，比如求交集、并集、差集等。可以方便搞定一些共同好友、共同爱好之类的功能；


简单消息队列，除了Redis自身的发布/订阅模式，我们也可以利用List来实现一个队列机制，比如：到货通知、邮件发送之类的需求，不需要高可靠，但是会带来非常大的DB压力，完全可以用List来完成异步解耦；


Session共享，默认Session是保存在服务器的文件中，如果是集群服务，同一个用户过来可能落在不同机器上，这就会导致用户频繁登陆；采用Redis保存Session后，无论用户落在那台机器上都能够获取到对应的Session信息。


Redis的其他应用场景：附近的人、车、餐馆；摇一摇；抢红包；搜索自动补全


# 减库存


减库存用递减计数或者是分布式锁或者lua脚本


# 一致性hash算法


解决集群动态扩展问题。


Redis集群架构






