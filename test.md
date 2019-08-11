## 

为了维护hashcode方法的一般约定，即相等的对象必须有相等的hashcode，在重写 equels() 方法后必须重写hashcode() 方法

做字节码增强都需要使用到框架，比如ASM，CGLIB，ByteBuddy，Javassist。不过可以直接运用位运算操作 byte[]，不需要任何框架，例如 JDK 的反射 (method.invoke()) 的实现，就真的是用位操作拼装了一个类

**1，如何控制多线程执行顺序**

- 通过 join 方法去保证多线程的顺序性， join调用 Object 的 wait 方法去等待子线程的执行
- Excutors.newSingleThreadExcutor(); FIFO 队列执行 submit 的线程

**2，volatile 和 synchronize 和 atomic 的区别**

- JMM (Java Memory Model) 本身并不真实存在，他描述的是一组规则或规范定义了各个变量的访问方式
  - 线程之间如何通信
    - 共享内存
    - 消息传递
  - 线程之间如何同步
    - 在共享内存的并发模型中，同步是显式做的: synchronized
    - 在消息传递的并发模型中，由于消息的发送必须在接收之前，所以同步是隐式的
- volatile 利用 JMM 提供的类似 Intel 提供的 MESI 协议来提供可见性和有序性 (缓存连贯性协议 MESI (Modified、Exclusive、Shared、Invalid) 解决可见性问题)，加入volatile关键字时会多出一个lock前缀指令，lock前缀指令实际上相当于一个内存屏障
- atomic 利用 CAS (Compare And Swap) 来实现原子性操作，(expect，update) 期望值不符则更新后再操作，利用 unsafe 类提供的各种操作来保证原子性，CAS 实际上是利用处理器提供的CMPXCHG指令实现的，而处理器执行CMPXCHG指令是一个原子性操作。CAS 缺点 :
  - 循环开销太大
  - 只能保证一个共享变量的原子性
  - 引出 ABA 问题 :
    - 一个线程将 A 改为 B，然后又将 B 改为 A，另一个线程用 CAS 的话并不知道动过
    - 原子引用 AtomicReference
    - 时间戳原子引用解决 ABA 问题 (即增加一个版本号) 即 AtomicStampedReference
- synchronized 利用 monitorenter 和 monitorexit 来进行线程同步

**3，lock 和 synchronized 的区别**

- synchronized 基于 JVM，lock 是 Java API

- lock 可以主动去释放锁而 synchronized 则必须等待代码块执行完或执行线程出现异常

- ReentrantLock 和 synchronized 就是典型的可重入锁(递归锁)

  - 线程可进入任何一个他已经拥有的锁所同步的代码块
  - 作用是避免死锁

- lock 可以选择公平锁或非公平锁

  - ReentrantLock 默认为非公平锁
  - 非公平锁优点在于吞吐量比公平锁高
  - 对于 synchronized 而言也是非公平锁

- lock 可以有读写锁

  - ReentrantReadWriteLock
- 读锁(共享锁)，读，多个线程可以读
  - 写锁(排它锁)，写，一个线程可以写
- lock 绑定多个条件 Condition 分组唤醒需要唤醒的线程，可以精确唤醒

**4，分布式锁的实现原理**

- 利用数据库，建立一个 lock 表，表上的一个加上唯一性约束，只有一个模块可以插入数据，即插入数据的模块获得锁其他的模块报插入错误，完成操作后删除数据(非重入(可记录进程 Id 解决)，删除失败问题)
- zookeeper 利用写入节点来实现操作的 FIFO 队列，利用其失效删除的特点解决删除失败问题
- redis setnx 命令 只在 key 不存在时设置值，并返回 0 和 1 来代表失败成功

**5，mysql 中的 binlog**

- 记录 mysql 的数据更新或者潜在更新，主从复制就是依靠 binlog
- statement 基于 sql 语句
- row 基于数据
- mixed 混合模式

**6，cookie 和 session 的联系和区别**

- 共同完成了浏览器会话状态的保存
  - cookie 和 session 都保存了jsessionId
- 把用户信息保存在 session 中
- 如果是内存 cookie，客户端只要关闭浏览器 session 就失效了
- 可以将 jsessionId 放到 cookie 中，也可以通过重写 URL 将 jsessionId 拼接在 url 中

**7，AOP 底层实现原理**

- 静态代理 : 代理其它的类完成其他类的工作，分工合作让被代理类专注于他的主要工作
- 动态代理 : 
  - Proxy 类的代码量被固定下来，不会因为业务的逐渐庞大而庞大(相比于静态代理)
  - 可以实现 AOP 编程，实际上静态代理也可以实现，总的来说，AOP 可以算作代理模式的一个典型应用
  - 解耦，通过参数就可以判断真实类，不需要事先实例化，更加灵活多变
- JDK Proxy JDK 自带动态代理实现，必须实现InvocationHandler接口
- cglib 是 spring 使用的 AOP 方式

**8，跨域问题的解决方案**

- 设置 document.domain (一级域名相同的情况之下)
- HTML 标签中有 src 属性，只支持 GET 请求，允许跨域
- script 标签 JSONP 格式 eval 
- iframe 之间交互 window.postMessage 方法 (字符串 255 个)
- 服务器后台 : CORS (安全沙箱) Access-Control-Allow-Origin: *

**9，集合类线程不安全问题 ArrayList**

- 所有集合类都是报 concurrentModificationException 异常

- Vector<> 方法加 synchronized 
- Collections.synchronizedList(list) 来将 list 改为线程安全的类 
- CopyOnWriteArrayList : 写的时候复制一份写完后废弃原来的 list， 写的时候加锁，CopyOnWriteArraySet 和 ConcurrentHashMap 是 set 和 map 的解决方案
- HashSet 底层是 HashMap，只不过 value 是为 PRESENT 的 Object 常量

**10，阻塞队列 BlockQueue**

- 当阻塞队列是空时，从队列中获取元素的操作将会被阻塞

- 当阻塞队列是满时，往队列里添加元素的操作将会被阻塞

- 简化了线程的控制，和 juc 包里的其他类一样简化多线程

| 方法类型 | 抛出异常 | 特殊值 | 阻塞 | 超时 |
| ----|----|----|----|----|
| 插入 | add(e) | offer(e) | put(e) | offer(e, time, unit) |
| 移除 | remove(e) | poll(e) | take(e) | poll(e, time, unit) |
| 检查 | element(e) | peek(e) | 无 | 无 |

生产者消费者模型：

1，synchronize, wait(), notify() (随机选一个唤醒), notifyAll()

2，ReentrantLock, Condition, await(), signal()，Before waiting on the condition the lock must be held by the current thread

3， LockSupport : park(), unpark()

4， BlockingQueue, put(), take() 

LinkedTransferQueue是一个由链表数据结构构成的无界阻塞队列，由于该队列实现了TransferQueue接口，与其他阻塞队列相比主要有以下不同的方法：

- **transfer(E e)** 如果当前有线程（消费者）正在调用take()方法或者可延时的poll()方法进行消费数据时，生产者线程可以调用transfer方法将数据传递给消费者线程。如果当前没有消费者线程的话，生产者线程就会将数据插入到队尾，直到有消费者能够进行消费才能退出
- **tryTransfer(E e)** tryTransfer方法如果当前有消费者线程（调用take方法或者具有超时特性的poll方法）正在消费数据的话，该方法可以将数据立即传送给消费者线程，如果当前没有消费者线程消费数据的话，就立即返回`false`。因此，与transfer方法相比，transfer方法是必须等到有消费者线程消费数据时，生产者线程才能够返回。而tryTransfer方法能够立即返回结果退出
- **tryTransfer(E e,long timeout,imeUnit unit)**
  与transfer基本功能一样，只是增加了超时特性，如果数据才规定的超时时间内没有消费者进行消费的话，就返回`false`

DelayQueue是一个存放实现Delayed接口的数据的无界阻塞队列，只有当数据对象的延时时间达到时才能插入到队列进行存储，所谓数据延时期满时，则是通过Delayed接口的`getDelay(TimeUnit.NANOSECONDS)`来进行判定，如果该方法返回的是小于等于0则说明该数据元素的延时期已满

##### 11，什么是惊群，如何有效避免惊群

​		惊群效应（thundering herd）是指多进程（多线程）在同时阻塞等待同一个事件的时候（休眠状态），如果等待的这个事件发生，那么他就会唤醒等待的所有进程（或者线程），但是最终却只能有一个进程（线程）获得这个时间的“控制权”，对该事件进行处理，而其他进程（线程）获取“控制权”失败，只能重新进入休眠状态，这种现象和性能浪费就叫做惊群效应

- linux 解决方案之 Accept : Linux 2.6 版本之前，监听同一个 socket 的进程会挂在同一个等待队列上，当请求到来时，会唤醒所有等待的进程。Linux 2.6 版本之后，通过引入一个标记位 WQ_FLAG_EXCLUSIVE，解决掉了 accept 惊群效应
- linux 解决方案之 Epoll : 
- 在使用 select、poll、epoll、kqueue 等 IO 复用时，多进程（线程）处理链接更加复杂。
  在讨论 epoll 的惊群效应时候，需要分为两种情况：
  - epoll_create 在 fork 之前创建
  - epoll_create 在 fork 之后创建
- nginx 解决方案之锁的设计 : Nginx 中使用的锁是自己来实现的，这里锁的实现分为两种情况，一种是支持原子操作的情况，也就是由 NGX_HAVE_ATOMIC_OPS 这个宏来进行控制的，一种是不支持原子操作，这是使用文件锁来实现。
- nginx 解决方案之惊群效应 : 变量分析，是否使用锁，获取锁来解决惊群。设置一把全局accpet锁，每个进程先去竞争这把锁，拿到锁的进程才向epoll中注册listen_fd事件

##### 12，Epoll

- epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次

- epoll 操作过程

  - int epoll_create(int size);创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大，参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议
  - int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
    - epfd：是epoll_create()的返回值。
    - op：表示op操作，用三个宏来表示：添加EPOLL_CTL_ADD，删除EPOLL_CTL_DEL，修改EPOLL_CTL_MOD。分别添加、删除和修改对fd的监听事件。
    - fd：是需要监听的fd（文件描述符）
    - epoll_event：是告诉内核需要监听什么事
  - int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout); 等待epfd上的io事件，最多返回maxevents个事件

- epoll 工作模式epoll对文件描述符的操作有两种模式：**LT（level trigger）**和 **ET（edge trigger）**。LT模式是默认模式，LT模式与ET模式的区别如下：

  - **LT模式**：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，`应用程序可以不立即处理该事件`。下次调用epoll_wait时，会再次响应应用程序并通知此事件
  - **ET模式**：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，`应用程序必须立即处理该事件`。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件

  ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。在 select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而**epoll事先通过epoll_ctl()来注册一 个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait() 时便得到通知**。(`此处去掉了遍历文件描述符，而是通过监听回调的的机制`。这正是epoll的魅力所在。)

##### 13，CGroup

​		CGroup 是 Control Groups 的缩写，是 Linux 内核提供的一种可以限制、记录、隔离进程组 (process groups) 所使用的物理资源 (如 cpu memory i/o 等等) 的机制 。是将任意进程进行分组化管理的 linux 内核功能。CGroup 本身是提供将进程进行分组化管理的基础结构。I/O 或内存的分配控制等通过 CGroup 来实现的具体资源管理功能称为 CGroup 子系统或控制器。目前越来越火的轻量级容器 Docker 就使用了 cgroups 提供的资源限制能力来完成cpu，内存等部分的资源控制。CGroup 提供了一个 CGroup 虚拟文件系统，作为进行分组管理和各子系统设置的用户接口。Linux内核有一个很强大的模块叫 VFS (Virtual File System)。 VFS 能够把具体文件系统的细节隐藏起来，给用户态进程提供一个统一的文件系统 API 接口。 cgroups 也是通过 VFS 把功能暴露给用户态的，cgroups 与 VFS 之间的衔接部分称之为 cgroups 文件系统。要使用 CGroup，必须挂载 CGroup 文件系统。这时通过挂载选项指定使用哪个子系统。可以估计 Docker 在实现不同的 Container 之间资源隔离和控制的时候，是可以创建比较复杂的 cgroups 节点和配置文件来完成的。然后对于同一个 Container 中的进程，可以把这些进程 PID 添加到同一组 cgroups 子节点中已达到对这些进程进行同样的资源限制。

**CGroup 子系统** :

1. cpu 子系统，主要限制进程的 cpu 使用率。

2. cpuacct 子系统，可以统计 cgroups 中的进程的 cpu 使用报告。

3. cpuset 子系统，可以为 cgroups 中的进程分配单独的 cpu 节点或者内存节点。

4. memory 子系统，可以限制进程的 memory 使用量。

5. blkio 子系统，可以限制进程的块设备 io。

6. devices 子系统，可以控制进程能够访问某些设备。

7. net_cls 子系统，可以标记 cgroups 中进程的网络数据包，然后可以使用 tc 模块（traffic control）对数据包进行控制。

8. freezer 子系统，可以挂起或者恢复 cgroups 中的进程。

9. ns 子系统，可以使不同 cgroups 下面的进程使用不同的 namespace。

这里面每一个子系统都需要与内核的其他模块配合来完成资源的控制，比如对 cpu 资源的限制是通过进程调度模块根据 cpu 子系统的配置来完成的；对内存资源的限制则是内存模块根据 memory 子系统的配置来完成的，而对网络数据包的控制则需要 Traffic Control 子系统来配合完成。

**CGroup 相关概念** :

- 任务（task）。在 cgroups 中，任务就是系统的一个进程
- 控制族群（control group）。控制族群就是一组按照某种标准划分的进程。Cgroups 中的资源控制都是以控制族群为单位实现。一个进程可以加入到某个控制族群，也从一个进程组迁移到另一个控制族群。一个进程组的进程可以使用 cgroups 以控制族群为单位分配的资源，同时受到 cgroups 以控制族群为单位设定的限制
- 层级（hierarchy）。控制族群可以组织成 hierarchical 的形式，既一颗控制族群树。控制族群树上的子节点控制族群是父节点控制族群的孩子，继承父控制族群的特定的属性
- 子系统（subsystem）。一个子系统就是一个资源控制器，比如 cpu 子系统就是控制 cpu 时间分配的一个控制器。子系统必须附加（attach）到一个层级上才能起作用，一个子系统附加到某个层级以后，这个层级上的所有控制族群都受到这个子系统的控制
##### 14，分布式事务

- 2PC : 两阶段提交 (Two-Phase Commit)，通过引入协调者来协调参与者的行为，并最终决定这些参与者是否要真正执行事务
  - 准备阶段 : 协调者询问参与者事务是否执行成功，参与者发回事务执行结果
  - 提交阶段 : 如果事务在每个参与者上都执行成功，事务协调者发送通知让参与者提交事务; 否则，协调者发送通知让参与者回滚事务。需要注意的是，在准备阶段，参与者执行了事务，但是还未提交。只有在提交阶段接收到协调者发来的通知后，才进行提交或者回滚
  - 存在的问题 :
    - 同步阻塞 : 所有都在等待其他参与者响应的时候都处于同步阻塞状态，无法进行其他操作
    - 单点问题 : 协调者在 2PC 中起到非常大的作用，发生故障会造成很大影响。特别是在二阶段发生故障，所有参与者会一直等待，无法完成其他操作
    - 数据不一致 : 在阶段二，如果协调者只发送了部分 Commit 消息，此时网络发生异常，那么只有部分参与者接收到 Commit 消息，也就是说只有部分参与者提交了事务，使得系统数据不一致
    - 太过保守 : 任意一个节点失败就会导致整个事务失败，没有完善的容错机制
- TCC : 解决了其几个缺点: 1.解决了协调者单点，由主业务方发起并完成这个业务活动。业务活动管理器也变成多点，引入集群。 2.同步阻塞:引入超时，超时后进行补偿，并且不会锁定整个资源，将资源转换为业务逻辑形式，粒度变小。 3.数据一致性，有了补偿机制之后，由业务活动管理器控制一致
- 本地消息表 : 方案的核心是将需要分布式处理的任务通过消息日志的方式来异步执行。消息日志可以存储到本地文本、数据库或消息队列，再通过业务规则自动或人工发起重试。**是BASE理论，是最终一致模型，适用于对一致性要求不高的。实现这个模型时需要注意重试的幂等**
- CAP : 分布式系统不可能同时满足一致性（C：Consistency）、可用性（A：Availability）和分区容忍性（P：Partition Tolerance），最多只能同时满足其中两项。在分布式系统中，分区容忍性必不可少，因为需要总是假设网络是不可靠的。因此，CAP 理论实际上是要在可用性和一致性之间做权衡。
- BASE : BASE 是基本可用（Basically Available）、软状态（Soft State）和最终一致性（Eventually Consistent）三个短语的缩写。BASE 理论是对 CAP 中一致性和可用性权衡的结果，它的核心思想是：即使无法做到强一致性，但每个应用都可以根据自身业务特点，采用适当的方式来使系统达到最终一致性
  - 基本可用 : 指分布式系统在出现故障的时候，保证核心可用，允许损失部分可用性
  - 软状态 : 指允许系统中的数据存在中间状态，并认为该中间状态不会影响系统整体可用性，即允许系统不同节点的数据副本之间进行同步的过程存在时延
  - 最终一致性 : 最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能达到一致的状态
  - ACID 要求强一致性，通常运用在传统的数据库系统上。而 BASE 要求最终一致性，通过牺牲强一致性来达到可用性，通常运用在大型分布式系统中
- 一致性
  - 弱一致性 : **所谓强一致性，即复制是同步的，弱一致性，即复制是异步的。**如果在返回“更新成功”并使更新对其他用户可见之前，主库需要等待从库的确认，确保从库已经收到写入操作，那么复制是同步的，即强一致性。如果主库写入成功后，不等待从库的响应，直接返回“更新成功”，则复制是异步的，即弱一致性。强一致性可以保证从库有与主库一致的数据。如果主库突然宕机，我们仍可以保证数据完整。但如果从库宕机或网络阻塞，主库就无法完成写入操作。在实践中，我们通常**使一个从库是同步的，而其他的则是异步的**。如果这个同步的从库出现问题，则使另一个异步从库同步。这可以确保永远有两个节点拥有完整数据：主库和同步从库。 这种配置称为**半同步**。当用户从**异步从库**读取时，**如果此异步从库落后，他可能会看到过时的信息。**这种不一致只是一个**暂时的状态**——如果等待一段时间，从库最终会赶上并与主库保持一致。这称为**最终一致性**
  - 最终一致性
    - DNS
    - Gossip : 被bitcoin使用。Gossip 过程是由种子节点发起，当一个种子节点有状态需要更新到网络中的其他节点时，它会随机的选择周围几个节点散播消息，收到消息的节点也会重复该过程，直至最终网络中所有的节点都收到了消息。这个过程可能需要一定的时间，由于不能保证某个时刻所有节点都收到消息，但是理论上最终所有节点都会收到消息，因此它是一个最终一致性协议。消息传播速度达到了 logN
    - Kademlia : 与前两代协议如 Chord、CAN、Pastry 等相比，Kad以全局唯一id标记对等网络节点，以节点ID异或（XOR）值度量节点之间距离，并通过距离分割子树构建路由表，建立了一种全新的网络拓扑结构。相比于其他算法，更简单，更高效。Kad网络中的每个节点都会被分配唯一的节点ID，一般是160bit的二进制数。节点之间可以计算距离，节点距离以节点ID的XOR值度量
  - 强一致性
    - 同步-主从同步复制
    - Paxos
    - Raft
    - ZAB


##### 15，Paxos, Raft, Zab

- Paxos

  - Basic Paxos，在Basic paxos算法中，分为4种角色 :
    - Client： 系统外部角色，产生议题者 (各部门)
    - Proposer：接收议题请求，向集群提出议题，并在冲突发生时起到冲突调节的作用 (议长)
    - Acceptor：提议的投票者和决策者，只有在形成法定人数（一般是majority多数派）时，提议才会被最终接受 (议会)
    - Learner：最终提议的接收者，backup，对集群的已执行没有什么影响 (记录员)
  - Multi Paxos = Raft/Zab **通过选择一个proposer作为leader降低多个proposer引起冲突的频率**　(国王)
  - Fast Paxos，重新描述了Paxos算法中的几个角色 :
    - Client/Proposer/Learner：负责提案并执行提案
    - Coordinator：Proposer协调者，可为多个，Client通过Coordinator进行提案
    - Leader：在众多的Coordinator中指定一个作为Leader
    - Acceptor：负责对Proposal进行投票表决Multi Paxos = Raft/Zab 通过选择一个proposer作为leader降低多个proposer引起冲突的频率　(国王)
- Raft : leader，follower，candidate

  - 在Raft集群中，有且仅有一个Leader，在Leader运行正常的情况下，一个节点服务器要么就是Leader，要么就是Follower。Follower直到Leader故障了，才有可能变成candidate

  - Leader负责把client的写请求log复制到follower。它会和follower保持心跳。每个follower都有一个timeout时间（一般为150ms~300ms），在接受到心跳的时候，这个timeout时间会被重置。如果follower没有接收到心跳，这些follower会把他们的状态变为candidate，并且开启新的一轮leader election
  - **term逻辑时钟** : Term相当于paxos中的proposerID，相当于一个国家的朝代。term是一段任意的时间序号。每一任Leader都有一个与之前不同的term，当Leader选举成功之后，一个节点成为了Leader，就会产生一个新的term，并且直到Leader故障，整个集群都会一直在这个term下执行操作。如果leader选举失败了，则会再生成出一个term，再开启一轮leader选举。
  - **Quorums** : 多数派，意思是超过一半的机器存活，则这个机器可用，这个Quorums指的就是集群可用的指标。例如：集群中的节点数为2N，如果有N+1的机器存活，则代表集群可用，可接受请求，写入log，应用到state machine中去，执行操作。如果少于N+1个机器存活，则代表集群可用，可接受请求，可写入log，但不应用到state machine中去，不执行操作。
  - Leader Election : 只有在下列两种情况下才会进行leader election：
    - 在第一次启动raft集群的时候
    - 在一个已存在的Leader故障的时候
  - 选举流程：如果以上两种任何一种发生了，所有的Follower无法再和Leader保持心跳，则它们都会等待一个（选举）timeout，如果其中一个Follower的timeout最先到时，则这个Follower变成candidate开始选举，第一，增加term计数器，第二，给自己投票并向所有其他的节点服务器请求投自己一票。如果一个Follower在接受到投票请求时，接受到两个term相同的投票请求时 (也就是说，产生了两个candidate)，则在多个相同term的投票请求中，这个Follower只能给投给其中一个请求，只能投一票，并且按照先来先服务的原则投票。如果这个candidate收到另外一个节点服务器的消息，并且这个节点服务器的term序号和当前的term序号一样大，甚至更大的话，则这个candidate选举失败，从而它的状态变成Follower，并且接受新的Leader。如果一个candidate获得了**Quorums**选票N+1(2N为集群中节点的数目)，则它变成新的leader。如果多个candidate和多个Follower投完票之后，有多个candidate获得了相同的票数，则会产生split vote，则新的term产生，重新选举。Raft用随机选举timeout迅速地解决split vote问题，这个方法就是对于产生spit vote的candidates各自随机生成一个选举timeout，谁先到时，谁当leader，其他candidate都变为Follower。当一个leader被选举出来之后，就在Follower timeout到时变为candidate之前，发心跳信息给所有Followers。
  - Client发送请求给Leader，其中每个请求都是一条操作指令。Leader接受到client请求之后，把操作指令(Entry)追加到Leader的操作日志中。紧接着对Follower发起AppendEntries请求、尝试让操作指令(Entry)追加到Followers的操作日志中，即落地。如果有Follower不可用，则一直尝试。一旦Leader接受到多数（**Quorums**）Follower的回应，Leader就会进行commit操作，每一台节点服务器会把操作指令交给状态机处理。这样就保证了各节点的状态的一致性。各服务器状态机处理完成之后，Leader将结果返回给Client。
  - **Follower crashes** : 如果一个follower故障了，则不会再接受AppendEntriesandvoterequests，并且Leader会不断尝试与这个节点保持心跳。如果这个节点恢复了，则会接受Leader的最新的log，并且将log应用到state machine中去，执行log中的操作
  - **Leader crashes** : 则会进行Leader election。如果碰到Leader故障的情况，集群中所有节点的日志可能不一致。old leader的一些操作日志没有通过集群完全复制。new leader将通过强制Followers复制自己的log来处理不一致的情况
  - Raft要求具备唯一Leader，并把一致性问题具体化为保持日志副本的一致性，以此实现相较Paxos而言更容易理解、更容易实现的目标。Raft是state machine system，Zab是primary-backup system
- Zab : Leading，Following，Election

  - **Epoch逻辑时钟** : Epoch相当于paxos中的proposerID，Raft中的term，相当于一个国家，朝代纪元。
  - **Quorums：**多数派，集群中超过半数的节点集合。
  - 节点中的持久化信息：
    - **history :** a log of transaction proposals accepted; 历史提议日志文件
    - **acceptedEpoch :** the epoch number of the last NEWEPOCH message accepted; 集群中的最近最新Epoch
    - **currentEpoch :** the epoch number of the last NEWLEADER message accepted; 集群中的最近最新Leader的Epoch
    - **lastZxid :** zxid of the last proposal in the history log; 历史提议日志文件的最后一个提议的zxid，在 ZAB 协议的事务编号 Zxid 设计中，**Zxid**是一个 64 位的数字，低 32 位是一个简单的单调递增的计数器，针对客户端每一个事务请求，计数器加 1; 高 32 位则代表 Leader 周期 epoch 的编号，每个当选产生一个新的 Leader 服务器，就会从这个 Leader 服务器上取出其本地日志中最大事务的ZXID，并从中读取 epoch 值，然后加 1，以此作为新的 epoch，并将低 32 位从 0 开始计数
  - Zab协议分为四个阶段 :
      - Phase 0: Leader election（选举阶段，Leader不存在）节点在一开始都处于选举阶段，只要有一个节点得到超半数Quorums节点的票数的支持，它就可以当选prospective  leader。只有到达 Phase 3 prospective leader 才会成为established  leader(EL)。
      - Phase 1: Discovery（发现阶段，Leader不存在）在这个阶段，PL收集Follower发来的acceptedEpoch(或者)，并确定了PL的Epoch和Zxid最大，则会生成一个NEWEPOCH分发给Follower，Follower确认无误后返回ACK给PL。这个阶段的主要目的是PL生NEWEPOCH，同时更新Followers的acceptedEpoch，并寻找最新的historylog，赋值给PL的history。**这个阶段的根本：发现最新的history log**
      - Phase 2: Synchronization（同步阶段，Leader不存在）**这个一阶段的主要目的是同步PL的historylog副本**
      - Phase 3: Broadcast（广播阶段，Leader存在）到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。**这个一阶段的主要目的是接受请求，进行消息广播**
  - 协议的 Java 版本实现跟上面的定义有些不同，选举阶段使用的是 Fast Leader Election（FLE），它包含了 Phase 1 的发现职责。因为**FLE 会选举拥有最新提议历史的节点作为 leader，这样就省去了发现最新提议的步骤**。实际的实现将 Phase 1 和 Phase 2 合并为 Recovery Phase（恢复阶段）。所以，ZAB 的实现只有三个阶段
- zab用的是epoch和count的组合来唯一表示一个值, 而raft用的是term和index.
- zab的follower在投票给一个leader之前必须和leader的日志达成一致,而raft的follower则简单地说是谁的term高就投票给谁.
- raft协议的心跳是从leader到follower, 而zab协议则相反.
- raft协议数据只有单向地从leader到follower(成为leader的条件之一就是拥有最新的log), 而zab协议在discovery阶段, 一个prospective leader需要将自己的log更新为quorum里面最新的log,然后才好在synchronization阶段将quorum里的其他机器的log都同步到一致.

##### 16，etcd 基于Raft

从 etcd 的架构图中我们可以看到，etcd 主要分为四个部分 :

- HTTP Server： 用于处理用户发送的 API 请求以及其它 etcd 节点的同步与心跳信息请求。
- Store：用于处理 etcd 支持的各类功能的事务，包括数据索引、节点状态变更、监控与反馈、事件处理与执行等等，是 etcd 对用户提供的大多数 API 功能的具体实现。
- Raft：Raft 强一致性算法的具体实现，是 etcd 的核心。
- WAL：Write Ahead Log（预写式日志），是 etcd 的数据存储方式。除了在内存中存有所有数据的状态以及节点的索引以外，etcd 就通过 WAL 进行持久化存储。WAL 中，所有的数据提交前都会事先记录日志。Snapshot 是为了防止数据过多而进行的状态快照；Entry 表示存储的具体日志内容。

通常，一个用户的请求发送过来，会经由 HTTP Server 转发给 Store 进行具体的事务处理，如果涉及到节点的修改，则交给 Raft 模块进行状态的变更、日志的记录，然后再同步给别的 etcd 节点以确认数据提交，最后进行数据的提交，再次同步。

##### 17，java CPU 高负载排查

- top -c 将系统资源使用情况实时显示出来（-c 参数可以完整显示命令）
- 输入大写 P 将应用按 CPU 使用率排序
- 利用 top -Hp pid 然后输入 p 可以按 CPU 使用率将线程排序，存储 pid
- jstack pid > pid.log 生成日志文件
- 如果是内存gc，用 jstat -gcutil pid 200 50 将内存使用，gc 回收状况打印出来
- 内存分析，通过命令 jmap -dump:live,format=b,file=dump.hprof pid 导出内存快照
- 最后利用 Eclipse Memory Analyzer 分析问题

##### 18，线程池 ThreadPoolExecutor 实现原理

- **降低资源消耗**，**提升系统响应速度**，**提高线程的可管理性**
- 线程池的工作原理 :

  - 先判断线程池中**核心线程池**所有的线程是否都在执行任务。如果不是，则新创建一个线程执行刚提交的任务，否则，核心线程池中所有的线程都在执行任务，则进入第2步
  - 判断当前**阻塞队列**是否已满，如果未满，则将提交的任务放置在阻塞队列中；否则，则进入第3步
  - 判断**线程池中所有的线程**是否都在执行任务，如果没有，则创建一个新的线程来执行任务，否则，则交给饱和策略进行处理 (Invokes the rejected execution handler for the given command)
- 线程池的创建 : 创建线程池主要是**ThreadPoolExecutor**类来完成，ThreadPoolExecutor的有许多重载的构造方法，ThreadPoolExecutor 参数最多的的构造方法的参数 :

  - corePoolSize：表示核心线程池的大小。当提交一个任务时，如果当前核心线程池的线程个数没有达到corePoolSize，则会创建新的线程来执行所提交的任务，**即使当前核心线程池有空闲的线程**。如果当前核心线程池的线程个数已经达到了corePoolSize，则不再重新创建线程。如果调用了`prestartCoreThread()`或者 `prestartAllCoreThreads()`，线程池创建的时候所有的核心线程都会被创建并且启动
  - maximumPoolSize：表示线程池能创建线程的最大个数。如果当阻塞队列已满时，并且当前线程池线程个数没有超过maximumPoolSize的话，就会创建新的线程来执行任务
  - keepAliveTime：空闲线程存活时间。如果当前线程池的线程个数已经超过了corePoolSize，并且线程空闲时间超过了keepAliveTime的话，就会将这些空闲线程销毁，这样可以尽可能降低系统资源消耗
  - unit：时间单位。为keepAliveTime指定时间单位
  - workQueue：阻塞队列。用于保存任务的阻塞队列，可以使用**ArrayBlockingQueue (有界队列), LinkedBlockingQueue (无界队列), SynchronousQueue (直接切换，这个队列比较特殊，它是一个没有数据缓冲的BlockingQueue (队列只能存储一个元素)，生产者线程对其的插入操作put必须等待消费者的移除操作take，反过来也一样，消费者移除数据操作必须等待生产者的插入), PriorityBlockingQueue**
  - threadFactory：创建线程的工程类。可以通过指定线程工厂为每个创建出来的线程设置更有意义的名字，如果出现并发问题，也方便查找问题原因
  - handler：饱和策略。当线程池的阻塞队列已满和指定的线程都已经开启，说明当前线程池已经处于饱和状态了，那么就需要采用一种策略来处理这种情况。采用的策略有这几种： 
    1. AbortPolicy： 直接拒绝所提交的任务，并抛出 **RejectedExecutionException** 异常
    2. CallerRunsPolicy：只用调用者所在的线程来执行任务
    3. DiscardPolicy：不处理直接丢弃掉任务
    4. DiscardOldestPolicy：丢弃掉阻塞队列中存放时间最久的任务，执行当前任务
- execute方法执行逻辑有这样几种情况：
  - 如果当前运行的线程少于corePoolSize，则会创建新的线程来执行新的任务
  - 如果运行的线程个数等于或者大于corePoolSize，则会将提交的任务存放到阻塞队列workQueue中
  - 如果当前workQueue队列已满的话，则会创建新的线程来执行任务
  - 如果线程个数已经超过了maximumPoolSize，则会使用饱和策略RejectedExecutionHandler来进行处理
- 线程池的监控 :
  - **getTaskCount**：线程池已经执行的和未执行的任务总数
  - **getCompletedTaskCount**：线程池已完成的任务数量，该值小于等于taskCount
  - **getLargestPoolSize**：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过，也就是达到了maximumPoolSize
  - **getPoolSize**：线程池当前的线程数量
  - **getActiveCount**：当前线程池中正在执行任务的线程数量
- ScheduledThreadPoolExecutor继承ThreadPoolExecutor来重用线程池的功能，它的实现方式如下 :
  - 将任务封装成ScheduledFutureTask对象，ScheduledFutureTask基于相对时间，不受系统时间的改变所影响
  - ScheduledFutureTask实现了`java.lang.Comparable`接口和`java.util.concurrent.Delayed`接口，所以有两个重要的方法：compareTo和getDelay。compareTo方法用于比较任务之间的优先级关系，如果距离下次执行的时间间隔较短，则优先级高；getDelay方法用于返回距离下次任务执行时间的时间间隔
  - ScheduledThreadPoolExecutor定义了一个DelayedWorkQueue，它是一个有序队列，会通过每个任务按照距离下次执行时间间隔的大小来排序
  - ScheduledFutureTask继承自FutureTask，可以通过返回Future对象来获取执行的结果
- DelayedWorkQueue是一个基于堆的数据结构，类似于DelayQueue和PriorityQueue。在执行定时任务的时候，每个任务的执行时间都不同，所以DelayedWorkQueue的工作就是按照执行时间的升序来排列，执行时间距离当前时间越近的任务在队列的前面，定时任务执行时需要取出最近要执行的任务，所以任务在队列中每次出队时一定要是当前队列中执行时间最靠前的，所以自然要使用优先级队列。
- The main pool control state , ctl : is an atomic integer packing two conceptual fields

  - workerCount : indicating the effective number of threads (后 29 位)
  - runState : indicating whether running, shutting down etc (前 3 位)
    - RUNNING : Accept new tasks and process queued tasks
    - SHUTDOWN : Don't accept new tasks, but process queued tasks
    - STOP : Don't accept new tasks, don't process queued tasks, and interrupt in-progress tasks
    - TIDYING : All tasks have terminated, workerCount is zero, the thread transitioning to state TIDYING will run the terminated() hook method
    - TERMINATED : terminated() has completed
    - Threads waiting in awaitTermination() will return when the state reaches TERMINATED

##### 19，Excutors

- newFixedThreadPool : corePoolSize n, maximumPoolSize n, keepAlivedTime 0, LinkedBlockingQueue

- newWorkStealingPool : ForkJoinPool (java8 新增，使用目前机器上可用处理器数作为他的并行级别)

- newSingleThreadExcutor : corePoolSize 1, maximumPoolSize 1, keepAlivedTime 0, LinkedBlockingQueue

- newCachedThreadPool : corePoolSize 0, maximumPoolSize Integer.MAX_VALUE, keepAlivedTime 60, SynchronousQueue

-  newSingleThreadScheduledExecutor : ScheduledThreadPoolExecutor

- newScheduledThreadPool : ScheduledThreadPoolExecutor

- 线程池线程数的合理配置

  - CPU 密集型 : 服务器 CPU 核心数

  - IO 密集型 : 

    - 一般 CPU 核心数 * 2
    - CPU 核心数 / (1 - 阻塞系数 (0.8 / 0.9))
- java 死锁
  

  - jps 定位进程号
  - jstack 找到死锁查看

##### 20，java IO

- IO: 一旦 IO 发起，则进程一直等待直到操作完成
- NIO : 同步非阻塞，调用后如果资源可用则进行操作，并立即返回
- IO多路复用 : 有一个线程专门同步着询问有那个IO操作准备好的，实现方式为selector，poll，epoll (都是通过系统软中断来实现，epoll_wait是阻塞调用，eppll只是一个通知的机制,但是取数据这个动作本身,还是要程序自己主动去调用recv之类的函数的)
- 信号驱动IO SIGIO
- 以上四种全部是同步IO (包括epoll) 因为他们都被阻塞到了recvfrom linux系统调用上 (只要没有设置read/recvfrom 的 IO_NONBLOCK 就为阻塞 IO)，above are all synchronous because the actual I/O operation (recvfrom) blocks the process
- AIO : 异步非阻塞，底层实现方式为linux AIO，windows iocp， jdk7主要增加了三个新的异步通道:
  - AsynchronousFileChannel : 用于文件异步读写
  - AsynchronousSocketChannel : 客户端异步socket
  - AsynchronousServerSocketChannel : 服务器异步socket

##### 21，Reactor 和 Proactor

两种I/O多路复用模式：Reactor和Proactor

一般地,I/O多路复用机制都依赖于一个事件**多路分离器(Event Demultiplexer)**。分离器对象可将来自事件源的I/O事件分离出来，并分发到对应的**read/write事件处理器(Event Handler)**。开发人员预先注册需要处理的事件及其事件处理器（或回调函数），事件分离器负责将请求事件传递给事件处理器

**两个与事件分离器有关的模式是Reactor和Proactor。Reactor模式采用同步IO，而Proactor采用异步IO**

**在Reactor中，**事件分离器负责等待文件描述符或socket为读写操作准备就绪，然后将就绪事件传递给对应的处理器，最后由处理器负责完成实际的读写工作

**而在Proactor模式中，**处理器--或者兼任处理器的事件分离器，只负责发起异步读写操作。IO操作本身由操作系统来完成。传递给操作系统的参数需要包括用户定义的数据缓冲区地址和数据大小，操作系统才能从中得到写出操作所需数据，或写入从socket读到的数据。事件分离器捕获IO操作完成事件，然后将事件传递给对应处理器。比如，在windows上，处理器发起一个异步IO操作，再由事件分离器等待IOCompletion事件。典型的异步模式实现，都建立在操作系统支持异步API的基础之上，我们将这种实现称为“系统级”异步或“真”异步，因为应用程序完全依赖操作系统执行真正的IO工作。

两个模式的相同点，都是对某个IO事件的事件通知(即告诉某个模块，这个IO操作可以进行或已经完成)。在结构上，两者也有相同点：demultiplexor负责提交IO操作(异步)、查询设备是否可操作(同步)，然后当条件满足时，就回调handler；不同点在于，异步情况下(Proactor)，当回调handler时，表示IO操作已经完成；同步情况下(Reactor)，回调handler时，表示IO设备可以进行某个操作(can read or can write)

**reactor应该提醒你外卖到了**  **proactor提醒你饭在桌上**

##### 22，Netty

sync queue : 接收 client 的第一个 sync 保存双方的 SYNC_SEND 和 SYNC_RECV 信息的 server 端队列

accept queue : server 发送 SYNC + ACC 后 client 返回 ACC，双方变为 ESTABLISHED 的连接状态队列

Channel 是 Netty 抽象出来的网络I/O读写相关的接口，为什么不使用 JDK NIO 原生的 Channel 呢：

- JDK 的 SocketChannel 和 ServerSocketChannel 没有统一的 Channel 接口供业务开发者使用，对于用户而言，没有统一的操作视图，使用起来并不方便
- JDK 的 SocketChannel 和 ServerSocketChannel 的主要职责就是网络I/O操作，由于它们是SPI类接口，由具体的虚拟机厂家来提供，所以通过继承SPI功能类来扩展其功能的难度很大；直接实现 ServerSocketChannel 和 SocketChannel 抽象类，其工作量和重新开发一个新的 Channel 功能类是差不多的
- Netty 的 Channel 需要能够跟 Netty 的整体架构融合在一起，例如I/O模型、基于ChannelPipeline 的定制模型，以及基于元数据描述配置化的TCP参数等，这些 JDK 的SocketChannel 和 ServerSocketChannel 都没有提供，需要重新封装
- 自定义的Channel，功能实现更加灵活

##### 23，AQS AbstarctQueuedSynchronized

使用AQS可以实现锁的功能，AQS的功能可以分为两种：独占和共享。对于这两种功能，有一个很常用的类：ReentrantReadWriteLock，其就是通过两个内部类来分别实现了这两种功能，提供了读锁和写锁的功能

AQS内部维护着一个FIFO的队列，该队列就是用来实现线程的并发访问控制。队列中的元素是一个Node类型的节点队列里还有一个`head`节点和一个`tail`节点，分别表示头结点和尾节点，其中头结点不存储Thread，仅保存next结点的引用

Node 的结构 :

- waitStatus：表示节点的状态，其中包含的状态有：
  - *CANCELLED*：值为1，表示当前节点被取消；
  - *SIGNAL*：值为-1，表示当前节点的的后继节点将要或者已经被阻塞，在当前节点释放的时候需要unpark后继节点；
  - *CONDITION*：值为-2，表示当前节点在等待condition，即在condition队列中；
  - *PROPAGATE*：值为-3，表示releaseShared需要被传播给后续节点（仅在共享模式下使用）；
  - *0*：无状态，表示当前节点在队列中等待获取锁。
- *prev*：前继节点；
- *next*：后继节点；
- *nextWaiter*：存储condition队列中的后继节点；
- *thread*：当前线程。

AQS 获取独占锁的实现 :

- acquire 方法
  - 尝试获取独占锁
  - 获取成功则返回，否则执行步骤3
  - addWaiter方法将当前线程封装成Node对象，并添加到队列尾部
  - 自旋获取锁，并判断中断标志位。如果中断标志位为`true`，执行步骤5，否则返回
  - 设置线程中断
- tryAcquire 方法
- addWaiter 方法 : 根据当前线程创建一个 Node 对象，然后添加到队列尾部
- acquireQueued 方法 : 循环的尝试获取锁，直到成功为止，最后返回中断标志位

##### 24，树

- AVL 树 : **AVL树**是最先发明的自平衡二叉查找树，在AVL树中任何节点的两个子树的高度最大差别为1，所以它也被称为**高度平衡树**，增加和删除可能需要通过一次或多次树旋转来重新平衡这个树
- 笛卡尔树 : 一种特定的二叉树，可由数列构造，在范围最值查询、范围top k查询（range top k queries）等问题上有广泛应用
- 2-3树
- 红黑树 (linux 任务调度，linux 内存管理mm-rb) : 是一种自平衡二叉查找树
- Kd-tree
  - k-demension tree
  - 确定划分维度 : 对于一个由n维数据构成的数据集，我们首先寻找方差最大的那个维度，设这个维度是dd，然后找出在维度dd上所有数据项的中位数mm，按mm划分数据集，一分为二，记这两个数据子集为Dl,DrDl,Dr。建立树节点，存储这次划分的情况（记录划分的维度dd以及中位数mm）
  - 选择充当切割标准的数据项 : 对Dl,DrDl,Dr重复进行以上的划分，并且将新生成的树节点设置为上一次划分的左右孩子
  - 递归地进行以上两步，直到不能再划分为止（所谓不能划分是说当前节点中包含的数据项的数量小于了我们事先规定的阈值，不失一般性，我在此篇博客中默认这个阈值是2，也就是说所有叶子节点包含的数据项不会多于2条），不能再划分时，将对应的数据保存至最后的节点中，这些最后的节点也就是叶子节点
- LSM 树 日志结构合并树 (HBase)
  - 核心思想的核心就是放弃部分读能力，换取写入的最大化能力。它的核心思路其实非常简单，就是假定内存足够大，因此不需要每次有数据更新就必须将数据写入到磁盘中，而可以先将最新的数据驻留在内存中，等到积累到最后多之后，再使用归并排序的方式将内存内的数据合并追加到磁盘队尾(因为所有待排序的树都是有序的，可以通过合并排序的方式快速合并到一起
  - 对磁盘来说，能够最大化的发挥磁盘技术特性的使用方式是:一次性的读取或写入固定大小的一块数据，并尽可能的减少随机寻道这个操作的次数
  - LSM具有批量特性，存储延迟
  - 基于LSM树实现的HBase的写性能比MySQL高了一个数量级，读性能低了一个数量级
  - LSM树原理把一棵大树拆分成N棵小树，它首先写入内存中，随着小树越来越大，内存中的小树会flush到磁盘中，磁盘中的树定期可以做merge操作，合并成一棵大树，以优化读性能
- B 树，B+ 树，B* 树，R 树
  - 二叉查找树结构由于树的深度过大而造成磁盘I/O读写过于频繁，进而导致查询效率低下
  - B树 (mongoDB)：有序数组+平衡多叉树
  - B+树 (mysql)：有序数组链表+平衡多叉树
  - B*树：一棵丰满的B+树
  - R树 (Redis) : 处理高维空间存储问题的数据结构，redis的geospatial可能用的就是R树。R树在数据库等领域做出的功绩是非常显著的。它很好的解决了在高维空间搜索等问题。举个R树在现实领域中能够解决的例子：查找20英里以内所有的餐厅。如果没有R树你会怎么解决？一般情况下我们会把餐厅的坐标(x,y)分为两个字段存放在数据库中，一个字段记录经度，另一个字段记录纬度。这样的话我们就需要遍历所有的餐厅获取其位置信息，然后计算是否满足要求。如果一个地区有100家餐厅的话，我们就要进行100次位置计算操作了，如果应用到谷歌地图这种超大数据库中，这种方法便必定不可行了。R树就很好的解决了这种高维空间搜索问题。它把B树的思想很好的扩展到了多维空间，采用了B树分割空间的思想，并在添加、删除操作时采用合并、分解结点的方法，保证树的平衡性。因此，R树就是一棵用来存储高维数据的平衡树

##### 25，可以作为 GCRoot 的对象

- 虚拟机栈 (栈帧中的局部变量，也叫作局部变量表) 中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中 JNI (Native 方法) 引用的对象

##### 26，JVM 的参数类型　

- jinfo -flag 参数 pid ------ 表示查看 jvm 指定参数信息 jinfo -flags pid
- java -XX:PrintFlagsInitial 打印所有默认 JVM 参数
- java -XX:PrintFlagsFinal 打印所有改过的 JVM 参数
- java -XX:+PrintCommandLineFlags 打印命令行参数

- 标配参数 : java -version, java -help
- X 参数 : 
  - -Xint : 解释执行
  - -Xcomp : 第一次使用就编译成本地代码，编译执行
  - -Xmixed : 混合模式
- XX 参数 : 
  - boolean 类型 : -XX:+ 或者 - 某个属性值，+ 表示开启，- 表示关闭 -XX:+PrintGCDetails
  - KV 设值类型 : -XX:属性key=属性值value         -XX:MetaSpaceSize=20m  -XX:MaxTenuringThreshold=15
  - -Xms == -XX:InitialHeapSize
  - -Xmx == -XX:MaxHeapSize
  - -Xss == -XX:TheadStackSize 设置单个线程栈的大小，一般默认值为 512K~1024K
  - -Xmn 设置年轻代大小
  - -XX:SurvivorRatio 默认8 (eden : from : to = 8 : 1 : 1)
  - -XX:NewRatio 默认2 (老年代 : 新生代 = 2 : 1)

##### 27，java 引用

- 强 : 就算是出现了 OOM 也不会对该对象进行回收
- 软 : 在内存不够时就会被回收，内存够时不会被回收
- 弱 : 在进行一次 GC 后就被回收
- 虚 : 如果一个对象仅持有虚引用，那么他就和没有任何引用一样，必须和引用队列来联合使用

##### 28，SMP, MMP, NUMA

- SMP 对称多处理 : 将多个处理器与一个集中的存储器相连，在 SMP 模式下，所有处理器都可以访问同一个物理存储器，这就意味着 SMP 系统只运行操作系统的一个拷贝。很显然，SMP 的缺点是伸缩性有限，因为在存储器接口达到饱和的时候，增加处理器并不能获得更高的性能

- ＭMP 大规模并行处理 : MMP 模式是一种分布式存储器模式，能够将更多的处理器纳入一个分布式系统的存储器。一个分布式存储器模式具有多个节点，每个节点都有自己的存储器，可以配置位 SMP 模式，也可以是非 SMP 模式，单个的节点相互连接起来就形成了一个总系统。由于没有硬件支持共享内存或高速缓存一致性的问题，所以比较容易实现大量处理器的连接

- NUMA 非统一内存访问 : NUMA 模式也采用了分布式存储器模式。不同的是所有节点中的处理器都可以访问全部的系统物理存储器。然而，每个处理器访问本节点内的存储器所需要的时间，可能比访问某些远程节点的存储器所花的时间要少的多。也就是访问存储器的时间是不一致的，这也是这种模式被称为 ‘NUMA’ 的原因

- AMD和Intel先后将内存控制器移入CPU内部，CPU之间用芯片互联总线连接，分别起名Hyper Transport和QPI，从而带来了 NUMA。NUMA既保持了SMP模式单一操作系统拷贝、简便的应用程序编程模式以及易于管理的特点，又继承了MPP模式的可扩充性，可以有效地扩充系统的规模。这也正是NUMA的优势所在

- 固件在NUMA的生态链中扮演了至关重要的地位，只有固件才能知道具体平台上CPU和内存的亲缘关系。UEFI固件通过ACPI报告给OS NUMA的组成结构，其中最重要的是SRAT（System Resource Affinity Table）和SLIT（System Locality Information Table）表。

  - 其中SRAT包含两个结构
    - Processor Local APIC/SAPIC Affinity Structure：记录某个CPU的信息
    - Memory Affinity Structure：记录内存的信息

  - SLIT表则记录了各个结点之间的距离，在系统中由数组node_distance[ ]记录

##### 29，Service Mesh

服务网格是一个用于处理服务间通信的基础设施层，它负责为构建复杂的云原生应用传递可靠的网络请求

##### 30，java 常见错误

- java.lang.StackOverFlowError
- java.lang.OutOfMemoryError : Java heap space
- java.lang.OutOfMemoryError : GC overhead limit exceeded
- java.lang.OutOfMemoryError : Direct buffer memory
- java.lang.OutOfMemoryError : unable to create new native thread 　　　
  - 在 linux 系统中默认限制非 root 只能产生 1024 个线程 (进程)
- java.lang.OutOfMemoryError : Metaspace

##### 31，java 垃圾回收器

- 新生代的垃圾回收器就决定了老年代使用的垃圾回收器
- Serial : 串行垃圾回收器，一个线程，阻塞用户线程，不适合服务器，-XX:UseSerialGC
- Parallel/ParallelOld : 并行垃圾回收器，多个线程，阻塞用户线程，科学计算、大数据处理等弱交互场景         -XX:UseParallelGC，-XX:UseParallelOldGC
- ParNew/CMS : 并发垃圾回收器，多个线程，同时执行，互联网公司多用，-XX:UseParNewGC,-XX:UseConcMarkSweepGC
- G1 : Eden, survivor 和 Tenured 等内存区域不再是连续的了，而是变成了一个个大小一样的 region，每个 region 可以在新生代和老年代之间切换。启动时可以通过参数 -XX:G1HeapRegionSize=n 可以指定分区大小 (1MB~32MB，且必须是 2 的幂)，默认将整堆划分为 2048 个分区，即最大支持内存 64G。小区域收集形成连续内存区域，和 CMS 一样分为 : 初始标记，并发标记，最终标记，筛选回收。但 G1 没有内存碎片，可以指定垃圾收集时间
- ZGC

##### 32，SSL/TLS协议的基本过程

- 客户端向服务器端索要并验证公钥
- 双方协商生成 对话秘钥
- 双方采用 对话秘钥 进行加密通信
- 握手阶段涉及四次通信
  - 客户端发出请求
    - 支持的协议版本，比如TLS 1.0版
    - 一个客户端生成的随机数，稍后用于生成"对话密钥"
    - 支持的加密方法，比如RSA公钥加密
    - 支持的压缩方法
  - 服务器回应
    -  确认使用的加密通信协议版本，比如TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信
    - 一个服务器生成的随机数，稍后用于生成"对话密钥"
    - 确认使用的加密方法，比如RSA公钥加密
    - 服务器证书
  - 客户端回应
    - 一个随机数。该随机数用服务器公钥加密，防止被窃听
    - 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送
    - 客户端握手结束通知，表示客户端的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供服务器校验
  - 服务器的最后回应
    - 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送
    - 服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供客户端校验

##### 33，常见的反爬虫机制

- 通过 UA 识别爬虫 : 有些爬虫的UA是特殊的，与正常浏览器的不一样，可通过识别特征UA直接封掉爬虫请求
- 设置 IP 访问频率，超出一定频率弹出验证码
- 通过并发识别爬虫 : 有些爬虫的并发是很高的，统计并发最高的IP，加入黑名单
- 蜜罐资源 : 爬虫解析离不开正则匹配，适当在页面添加一些正常浏览器浏览访问不到的资源，一旦有ip访问，过滤下头部是不是搜素引擎的蜘蛛，不是就可以直接封了。比如说隐式链接 

##### 34，Java Instrumentation

JVMTI : JVM Tool Interface 是 Java Instrumentation API 的基础，程序 Debug 功能也是通过这个功能实现的

JVMTI agent 会在 debug 连接时加载到 debugee 的 JVM 中。debuger 通过 JDI (Java Debug Interface) 与 Debugee 通过进程通讯来设置断点、获取调试信息。JDI 还有一项 redefineClass 的方法，可以直接修改一个类的字节码

##### 35，Attach API

- Attach API可不是仅仅为了实现动态加载 agent，attcah API 其实是跨 JVM 进程通讯的工具，能够将某种指令从一个 JVM 发送给另一个 JVM 进程。加载 Agent 只是 Attach API 发送的各种指令中的一种，诸如 jstack 打印线程栈、jps 列出 Java 进程、jmap做内存 dump 等功能，都属于 Attach API 可以发送的指令

- external process 执行 VirtualMachine.attach 时，需要通过操作系统提供的进程通信方法，例如信号、socket，进行握手和通信，具体流程为

  - external process 创建文件 .attach_pidXXX，向目标 JVM 发送 SIGQUIT信号，轮询等待 .java_pidXXX 的创建，在文件 .java_pidXXX 创建后尝试连接 socket
  - JVM 的 Signal Dispatcher 线程收到 SIGQUIT 信号，检查 .attach_pidXXX 文件是否存在，不存在则忽略信号，存在则创建一个新线程 Attach Listener，专门负责接收 Attach 请求指令，创建 .java_pidXXX 文件开始监听 socket


##### 26，JVM如何创建一个线程

- 调用 Java 线程的 start() 方法，通过 jni 方式，调用pthread_create() 创建一个系统内核线程
- JVＭ 创建一个 Thread 对象，指定内核线程的初始运行地址
- 利用 JavaCalls 模块，调用 Java 线程的 run() 方法，开始java级别的线程执行

##### 27，KVM

- KVM 全称是 基于内核的虚拟机（Kernel-based Virtual Machine），它是一个 Linux 的一个内核模块，该内核模块使得 Linux 变成了一个 Hypervisor
- KVM中，虚拟机被实现为常规的linux进程，由标准 Linux 调度程序进行调度; 虚拟机的每个虚拟 CPU 被实现为一个常规的 Linux 进程，这使得 KVM 能够使用 Linux 内核的已有功能
- KVM 本身不执行任何硬件模拟，需要客户空间程序通过 /dev/kvm 接口设置一个客户机虚拟服务器的地址空间，向它提供模拟的 I/O，并将它的视频显示映射回宿主的显示屏。目前这个应用程序是 QEMU
- Linux 上的用户空间、内核空间和虚拟机 :
  - Guest：客户机系统，包括CPU（vCPU）、内存、驱动（Console、网卡、I/O 设备驱动等），被 KVM 置于一种受限制的 CPU 模式下运行。
  - KVM：运行在内核空间，提供CPU 和内存的虚级化，以及**客户机的 I/O 拦截**。Guest 的 I/O 被 KVM 拦截后，交给 QEMU 处理。
  - QEMU：修改过的为 KVM 虚拟机使用的 QEMU 代码，运行在用户空间，提供硬件 I/O 虚拟化，通过 IOCTL /dev/kvm 设备和 KVM 交互
- **KVM 是实现拦截虚拟机的 I/O 请求的原理：**现代 CPU 本身提供了对特殊指令的截获和重定向的硬件支持，甚至新的硬件会提供额外的资源来帮助软件实现对关键硬件资源的虚拟化从而提高性能。以 X86 平台为例，支持虚拟化技术的 CPU  带有特别优化过的指令集来控制虚拟化过程。通过这些指令集，VMM 很容易将客户机置于一种受限制的模式下运行，一旦虚拟机试图访问物理资源，硬件会暂停客户机的运行，将控制权交回给 VMM 处理。VMM 还可以利用硬件的虚级化增强机制，将客户机在受限模式下对一些特定资源的访问，完全由硬件重定向到 VMM 指定的虚拟资源，整个过程不需要暂停客户机的运行和 VMM 的参与。由于虚拟化硬件提供全新的架构，支持操作系统直接在上面运行，无需进行二进制转换，减少了相关的性能开销，极大简化了VMM的设计，使得VMM性能更加强大。从 2005 年开始，Intel 在其处理器产品线中推广 Intel Virtualization Technology 即 IntelVT 技术
- 一个普通的 Linux 内核有两种执行模式：内核模式（Kenerl）和用户模式 （User）。为了支持带有虚拟化功能的 CPU，KVM 向 Linux 内核增加了第三种模式即客户机模式（Guest），该模式对应于 CPU 的 VMX non-root mode，KVM 内核模块作为 User mode 和 Guest mode 之间的桥梁：
  - User mode 中的 QEMU-KVM 会通过 IOCTL 命令来运行虚拟机
  - KVM 内核模块收到该请求后，它先做一些准备工作，比如将 VCPU 上下文加载到 VMCS （virtual machine control structure）等，然后驱动 CPU 进入 VMX non-root 模式，开始执行客户机代码
- 三种模式的分工为 :
  - Guest 模式：执行客户机系统非 I/O 代码，并在需要的时候驱动 CPU 退出该模式
  - Kernel 模式：负责将 CPU 切换到 Guest mode 执行 Guest OS 代码，并在 CPU 退出  Guest mode 时回到 Kenerl 模式
  - User 模式：代表客户机系统执行 I/O 操作

##### 28，keepalive实现虚拟IP + IP偏移

- 虚拟IP : 就是一个未分配给真实主机的IP，也就是说对外提供数据库服务器的主机除了有一个真实IP外还有一个虚IP，使用这两个 IP 中的任意一个都可以连接到这台主机，所有项目中数据库链接一项配置的都是这个虚IP，当服务器发生故障无法对外提供服务时，动态将这个虚拟IP切换到备用主机。这个切换的过程我们称之为**IP漂移**
- 其实现原理主要是靠 TCP/IP 的 ARP 协议。因为 IP 地址只是一个逻辑 地址，在以太网中 MAC 地址才是真正用来进行数据传输的物理地址，每台主机中都有一个 ARP缓存，存储同一个网络内的IP地址与 MAC 地址的对应关系，以太网中的主机发送数据时会先从这个缓存中查询目标 IP 对应的MAC地址，会向这个 MAC 地址发送数据。操作系统会自动维护这个缓存。这就是整个实现的关键
- 我们可以通过 Keepalived 来实现这个过程。 Keepalived 是一个基于 **VRRP 协议（Virtual Router Redundancy Protocol，即虚拟路由冗余协议）**来实现的LVS（负载均衡器）服务高可用方案，可以利用其来避免单点故障
- 一个 LVS 服务会有2台服务器运行 Keepalived，一台为主服务器（MASTER），另一台为备份服务器（BACKUP），但是对外表现为一个虚拟IP，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候，备份服务器就会接管虚拟IP，这时就需要根据 VRRP 的优先级来选举一个 backup 当 master，保证路由器的高可用，继续提供服务，从而保证了高可用性

#####　29，造成死锁的四个必要条件

- 互斥
- 占有且等待
- 不可强行占有
- 循环等待条件