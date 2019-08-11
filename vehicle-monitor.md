##### 自我介绍

- 你好，我叫舒冲冲，本科和研究生都就读于山东大学，主要技术栈是java，平常一直在linux平台上开发。另外也熟悉c/c++和go语言。最近的一个项目是一个汽车运输公司委托我们帮他们做一个满足交通部jt808, jt1078和苏标ADAS协议的一个平台来送去交通部过检。项目主要包括jt808服务器，jt809服务器，jt1078服务器和web端四个主要部分。我主要负责jt808服务器，jt1078服务器和部分相关的后台前端的开发。后面说以下项目难点和待完成的部分，然后说明自己学到的东西。

##### 语言并不重要，而且现在java和c++在语法和功能上已经非常相似了

##### 项目难点 :

- 对自定义协议的处理 : 多个车辆终端的多线程处理，特别是终端发送图片过来的时候，因为图片比较大，进行了分片，而 netty 默认是一个线程处理一个特定的连接，所以必须在处理过程中开辟新的线程来处理图片
- 对G726音频的编解码 : G726这种格式十分少见，在查询了介绍 RTP 的文档 RFC3551 后才了解到它分为40K, 32K, 24K, 16K四个等级采样位数分别为５, 4, 3, 2，采样率都是8000，且每秒20个数据包。首先需要将其转化为 PCM 格式，然后再转化为 AAC 格式。但最为困难的是将它和 h.264 格式的视屏画面进行音画同步。由于一开始项目使用 java 开发，因此使用的是javacv 中的 FFmpeg 来处理。但是由于java 中的 FFmpeg 不好控制，而且设备发送过来的每帧的时间戳也并不准确，因此必须分析帧的类型 : I帧、P帧、B帧和 PTS 帧(用于度量解码后的视频帧什么时候被显示出来)、DTS 帧(主要是标识读入内存中的 bit 流在什么时候开始送入解码器中进行解码) 来确定时间输出音频帧，找到了解析G726的 c 程序，现在因为这个项目中的音频不是很重要，所以目前还是在使用 java 的服务器，后面会改造成 c++ 的。
- 实现网页端和车辆终端的语音对讲 : 使用的是 WebRTC 提供的接口来获取网页端的音频，然后利用Recordjs来保存音频数据，通过 websocket 的二进制模式来传输音频数据到服务端经过编解码送到终端。而终端的音频数据同样通过编解码后，利用 websocket 传输到 web 端播放，目前 bug 比较多，后面会改进
- 文件服务器的信令端口和文件传输端口被设置成一个端口，无法完美的分辨出到底是那种数据

##### 通过项目学习到了什么

- 熟悉了网络应用的开发
- 熟悉了音视频编解码方面开发的知识
- 熟悉了多线程编程的知识
- 熟悉了多个服务间的信息传输，RPC方面的知识
- 熟悉了服务的容器化部署方面的知识
- 熟悉了技术的选型和调整，和对多种常用的应用和框架的了解

##### 为什么使用 mongodb 

- 架构简单
- 没有复杂的连接
- 深度查询能力 MongoDB支持动态查询

- 支持内嵌对象和数组对象
- 后面可能会用 HBase 存储历史数据
- 不需要转化/映射应用对象到数据库对象
- 使用内存作为存储工作区,以便更快的存取数据
- 因为 `query` 简单了，少了许多消耗资源的 `join` 操作，速度自然会上去
- 非常适合实时的插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性。
- 易于扩展

##### mongoDB

- MongoDB用c++编写的

- 非关系型数据库的显著特点是不使用SQL作为查询语言，数据存储不需要特定的表格模式
- 分片是将数据水平切分到不同的物理节点。当应用数据越来越大的时候，数据量也会越来越大。当数据量增长时，单台机器有可能无法存储数据或可接受的读取写入吞吐量。利用分片技术可以添加更多的机器来应对数据量增加以及读写操作的要求

##### 为什么使用 MQ 

- 使得前端不用一直向后台请求位置信息，实现一定的实时性
- 解耦 : 开始使用的是 WebSocket，使得浏览器与 jt808 服务器耦合度太高
- 分摊 jt808 服务器的压力 : 需要向多个浏览器发送位置信息，占用了 jt808 的带宽和处理器时间
- 异步 : 分离数据的获取与处理，易于后面数据的处理，使得 jt808 不用自己存储原始数据。将消息发送到消息队列之后立即返回，之后这个操作会被异步处理
- 如果设备大于一定数量后，jt808 可能会集群部署，用 MQ 可以统一数据的处理
- ~~不用考虑消息被重复消费 (消息被消费时的幂等性)~~ 先根据 timeStamp 查一下，如果数据存在，就不插入了

##### 为什么使用 RabbitMQ 

- 使用简单，成熟的 API 文档，良好的管理界面
- 使用 erlang 编写，性能很好
- 支持持久化，在消费者下线的情况下，生产的消息会丢失
- 支持 Stomp

##### 如何保证消息队列是高可用的

RabbitMQ 集群

##### 如何保证消费的可靠性传输

- 生产者丢数据 : RabbitMQ提供transaction和confirm模式来确保生产者不丢消息

  transaction机制就是说，发送消息前，开启事物(channel.txSelect())，然后发送消息，如果发送过程中出现什么异常，事物就会回滚(channel.txRollback())，如果发送成功则提交事物(channel.txCommit())

- 消息队列丢数据 : 处理消息队列丢数据的情况，一般是开启持久化磁盘的配置

  - 将queue的持久化标识durable设置为true,则代表是一个持久的队列
  - 发送消息的时候将 deliveryMode = 2

- 消费者丢数据 : 消费者丢数据一般是因为采用了自动确认消息模式。这种模式下，消费者会自动确认收到信息。这时rahbitMQ会立即将消息删除，这种情况下如果消费者出现异常而没能处理该消息，就会丢失该消息
  至于解决方案，采用手动确认消息即可 

##### 为什么使用缓存

- 缓存性能高
- 缓存雪崩 : 缓存失效，流量击垮数据库
  - 事前：redis 高可用，主从+哨兵，redis cluster，避免全盘崩溃
  - 事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 被打死
  - 事后：redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据
- 服务降级 : 一个非核心的功能异常最终导致了整个系统的不可用为了了避免这种小功能搞垮大系统的情况发生
  - 降级按照是否自动化可分为：自动开关降级和人工开关降级
    - 超时降级 : 当访问的数据库/http服务/远程调用响应慢或者长时间响应慢，且该服务不是核心服务的话可以在超时后自动降级
    - 统计失败次数降级 : 有时候依赖一些不稳定的API，比如调用外部机票服务，当失败调用次数达到一定阀值自动降级；然后通过异步线程去探测服务是否恢复了，则取消降级
    - 故障降级 : 比如要调用的远程服务挂掉了（网络故障、DNS故障、http服务返回错误的状态码、rpc服务抛出异常），则可以直接降级
    - 限流降级 : 当我们去秒杀或者抢购一些限购商品时，此时可能会因为访问量太大而导致系统崩溃，此时开发者会使用限流来进行限制访问量，当达到限流阀值，后续请求会被降级
  - 降级按照功能可分为：读服务降级、写服务降级
    - 动态化降级为静态化：比如平时网站可以走动态化渲染商品详情页，但是到了大促来临之际可以将其切换为静态化来减少对核心资源的占用，而且可以提升性能
    - 写服务在大多数场景下是不可降级的，不过可以通过一些迂回战术来解决问题。比如将同步操作转换为异步操作，或者限制写的量/比例
  - 降级按照处于的系统层次可分为：多级降级
- 限流组件 : 可以设置每秒的请求，未通过的请求走降级，可以返回一些默认的值，或友情提示，或空白
  - 数据库绝对不会死，限流组件确保了每秒只有多少个请求能通过
  - 只要数据库不死，就是说，对用户来说，2/5 的请求都是可以被处理的
  - 只要有 2/5 的请求可以被处理，就意味着你的系统没死，对用户来说，可能就是点击几次刷不出来页面，但是多点几次，就可以刷出来一次
- 常见的限流算法有：令牌桶、漏桶。计数器也可以进行粗暴限流实现
  - 令牌桶算法 : 令牌桶算法是一个存放固定容量令牌的桶，按照固定速率往桶里添加令牌 Semaphore
  - 漏桶算法 : 漏桶作为计量工具（The Leaky Bucket Algorithm as a Meter）时，可以用于流量整形（Traffic Shaping）和流量控制（TrafficPolicing）
- cache aside pattern : 
  - 读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应
  - 更新的时候，**先更新数据库，然后再删除缓存**/**先删除缓存，然后再更新数据库**
  - 更新数据的时候，根据**数据的唯一标识**，将操作路由之后，发送到一个 jvm 内部队列中。读取数据的时候，如果发现数据不在缓存中，那么将重新读取数据+更新缓存的操作，根据唯一标识路由之后，也发送同一个 jvm 内部队列中

##### 为什么要用 Redis

- 纯内存操作

- 单线程

- 高效的数据结构

  - 特殊的字符串结构 SDS { int len; int free; char buf[] } 二进制安全
  - rehash，渐进式 rehash分多次完成 rehash 操作
  - **跳跃表 zskipList 是有序集合 Zset 的底层结构**
  - 压缩列表 zipList 是 Redis 为节约内存而开发的，是列表键和字典键的底层实现之一
  - String : int、raw，List : zipList、linkedList，Hash : zipList、hashtable，Set : intSet、hashtable，Zset : ziplist、zskiplist

- Redis 会将每一个设置了 expire 的键存储在一个独立的字典中，Redis 默认每秒进行十次过期扫描，过期扫描不会扫描所有过期字典中的 key，而是采用了一种简单的贪心策略

- redis 事务的 CAS : **多客户端同时并发写**一个 key

  - 可以基于 zookeeper 实现分布式锁
  - 写入缓存的数据，都是从 mysql 里查出来的，都得写入 mysql 中，写入 mysql 中的时候必须保存一个时间戳，从 mysql 查出来的时候，时间戳也查出来

- redis cluster 介绍

  - 自动将数据进行分片，每个 master 上放一部分数据
  - 供内置的高可用支持，部分 master 不可用时，还是可以继续工作的
  - redis cluster 节点间采用 gossip 协议进行通信，每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，gossip 协议包含多种消息，包含 `ping`,`pong`,`meet`,`fail` 等等

- 一致性 hash 算法 :

  - 一致性 hash 算法将整个 hash 值空间组织成一个虚拟的圆环，整个空间按顺时针方向组织，下一步将各个 master 节点（使用服务器的 ip 或主机名）进行 hash。这样就能确定每个节点在其哈希环上的位置。将哈希空间 [0, 2n-1] 看成一个哈希环，每个服务器节点都配置到哈希环上。每个数据对象通过哈希取模得到哈希值之后，存放到哈希环中顺时针方向第一个**大于等于**该哈希值的节点上
  - 一致性哈希算法在节点太少时，容易因为节点分布不均匀而造成**缓存热点**的问题。为了解决这种热点问题，一致性 hash 算法引入了虚拟节点机制，即对每一个节点计算多个 hash，每个计算结果位置都放置一个虚拟节点。这样就实现了数据的均匀分布，负载均衡
  - redis cluster 有固定的 `16384` 个 hash slot，对每个 `key` 计算 `CRC16` 值，然后对 `16384`取模，可以获取 key 对应的 hash slot
  - redis cluster 中每个 master 都会持有部分 slot，比如有 3 个 master，那么可能每个 master 持有 5000 多个 hash slot。hash slot 让 node 的增加和移除很简单，增加一个 master，就将其他 master 的 hash slot 移动部分过去，减少一个 master，就将它的 hash slot 移动到其他 master 上去，任何一台机器宕机，另外两个节点，不影响的。因为 key 找的是 hash slot，不是机器

- sentinel，中文名是哨兵。哨兵是 redis 集群机构中非常重要的一个组件，主要有以下功能：

  - 集群监控：负责监控 redis master 和 slave 进程是否正常工作
  - 消息通知：如果某个 redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员
  - 故障转移：如果 master node 挂掉了，会自动转移到 slave node 上
  - 配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址

- redis 过期策略是：**定期删除 + 惰性删除**

- redis 内存淘汰机制有以下几个：

  - noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用
  -  **allkeys-lru**：当内存不足以容纳新写入数据时，在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）
  - allkeys-random：当内存不足以容纳新写入数据时，在**键空间**中，随机移除某个 key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊
  - volatile-lru：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）
  - volatile-random：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，**随机移除**某个 key
  - volatile-ttl：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除

- redis 和 memcached 有啥区别

  - redis 支持复杂的数据结构

  - redis 支持原生集群模式 

- redis 的线程模型 : redis 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 redis 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 socket，根据 socket 上的事件来选择对应的事件处理器进行处理

- 文件事件处理器的结构包含 4 个部分

  - 多个 socket
  - IO 多路复用程序
  - 文件事件分派器
  - 事件处理器（连接应答处理器、命令请求处理器、命令回复处理

- redis 分布式锁和 zk 分布式锁的对比 

  - redis 分布式锁，其实**需要自己不断去尝试获取锁**，比较消耗性能。
  - zk 分布式锁，获取不到锁，注册个监听器即可，不需要不断主动尝试获取锁，性能开销较小
  - redis 获取锁的那个客户端 出现 bug 挂了，那么只能等待超时时间之后才能释放锁，znode 就没了，此时就自动释放锁
  
- bgsave的原理是什么 : fork和cow。fork是指redis通过创建子进程来进行bgsave操作，cow指的是copy on write，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来

- 是否使用过Redis集群，集群的原理是什么

  - Redis Sentinal着眼于高可用，在master宕机时会自动将slave提升为master，继续提供服务。

  - Redis Cluster着眼于扩展性，在单个redis内存不足时，使用Cluster进行分片存储。

##### 为什么使用 nginx

- 增加 nginx-http-flv-module 后作为推流服务器推送视频和音频

- 存储终端传来的图片，方便前端展示

- nginx负载均衡的算法怎么实现的

  - 轮询 (默认) 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除

  - weight 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况

  - ip_hash 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题

  - fair (第三方) 按后端服务器的响应时间来分配请求，响应时间短的优先分配。  

  - url_hash (第三方)

- 什么是 nginx : Nginx是一个web服务器和反向代理服务器，用于`HTTP`、`HTTPS`、`SMTP`、`POP3`和`IMAP`协议
- nginx 的一些特性
  - 反向代理/L7负载均衡器
  - 嵌入式Perl解释器
  - 动态二进制升级
  - 可用于重新编写URL，具有非常好的PCRE支持
- nginx 如何处理 HTTP 请求 : `Nginx`使用反应器模式。主事件循环等待操作系统发出准备事件的信号，这样数据就可以从套接字读取，在该实例中读取到缓冲区并进行处理。单个线程可以提供数万个并发连接，`Master`进程：读取及评估配置和维持，`Worker`进程：处理请求
- 什么是 C10K 问题 : `C10K`问题是指无法同时处理大量客户端(10,000)的网络套接字

##### 为什么使用 protobuf

作为序列化层，用于加速命令在服务器之间的传输

##### 如何学习

在 github 或 programCreek 搜索别人的写法，然后模仿

  