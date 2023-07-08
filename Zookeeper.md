# Zookeeper

## 基本介绍

### 框架特征

Zookeeper 是 Apache Hadoop 项目子项目，为分布式框架提供协调服务，是一个树形目录服务

Zookeeper 是基于观察者模式设计的分布式服务管理框架，负责存储和管理共享数据，接受观察者的注册监控，一旦这些数据的状态发生变化，Zookeeper 会通知观察者

* Zookeeper 是一个领导者（Leader），多个跟随者（Follower）组成的集群
* 集群中只要有半数以上节点存活就能正常服务，所以 Zookeeper 适合部署奇数台服务器
* **全局数据一致**，每个 Server 保存一份相同的数据副本，Client 无论连接到哪个 Server，数据都是一致
* 更新的请求顺序执行，来自同一个 Client 的请求按其发送顺序依次执行
* **数据更新原子性**，一次数据更新要么成功，要么失败
* 实时性，在一定的时间范围内，Client 能读到最新数据
* 心跳检测，会定时向各个服务提供者发送一个请求（实际上建立的是一个 Socket 长连接）

![](https://img.jiapeng.store/img/202307081708419.png)



参考视频：https://www.bilibili.com/video/BV1to4y1C7gw





***



### 应用场景

Zookeeper 提供的主要功能包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡、分布式锁等

* 在分布式环境中，经常对应用/服务进行统一命名，便于识别，例如域名相对于 IP 地址更容易被接收

  ```sh
  /service/www.baidu.com 		# 节点路径
  192.168.1.1  192.168.1.2	# 节点值
  ```

  如果在节点中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求，可以实现负载均衡

  ```sh
  192.168.1.1  10	# 次数
  192.168.1.1  15
  ```

* 配置文件同步可以通过 Zookeeper 实现，将配置信息写入某个 ZNode，其他客户端监视该节点，当节点数据被修改，通知各个客户端服务器

* 集群环境中，需要实时掌握每个集群节点的状态，可以将这些信息放入 ZNode，通过监控通知的机制实现

* 实现客户端实时观察服务器上下线的变化，通过心跳检测实现





***





## 基本操作

### 安装搭建

安装步骤：

* 安装 JDK

* 拷贝 apache-zookeeper-3.5.7-bin.tar.gz 安装包到 Linux 系统下，并解压到指定目录

* conf 目录下的配置文件重命名：

  ```
  mv zoo_sample.cfg zoo.cfg
  ```

* 修改配置文件：

  ```sh
  vim zoo.cfg
  # 修改内容
  dataDir=/home/seazean/SoftWare/zookeeper-3.5.7/zkData 
  ```

* 在对应目录创建 zkData 文件夹：

  ```sh
  mkdir zkData
  ```

Zookeeper 中的配置文件 zoo.cfg 中参数含义解读： 

* tickTime = 2000：通信心跳时间，**Zookeeper 服务器与客户端心跳**时间，单位毫秒
* initLimit = 10：Leader 与 Follower 初始通信时限，初始连接时能容忍的最多心跳次数
* syncLimit = 5：Leader 与 Follower 同步通信时限，LF 通信时间超过 `syncLimit * tickTime`，Leader 认为 Follwer 下线
* dataDir：保存 Zookeeper 中的数据目录，默认是 tmp目录，容易被 Linux 系统定期删除，所以建议修改
* clientPort = 2181：客户端连接端口，通常不做修改



***



### 操作命令

#### 服务端

Linux 命令：

* 启动 ZooKeeper 服务：`./zkServer.sh start`

* 查看 ZooKeeper 服务：`./zkServer.sh status`

* 停止 ZooKeeper 服务：`./zkServer.sh stop`

* 重启 ZooKeeper 服务：`./zkServer.sh restart `

* 查看进程是否启动：`jps`





***



#### 客户端

Linux 命令：

* 连接 ZooKeeper 服务端：

  ```sh
  ./zkCli.sh					# 直接启动
  ./zkCli.sh –server ip:port	# 指定 host 启动
  ```

客户端命令：

* 基础操作：

  ```sh
  quit						# 停止连接
  help						# 查看命令帮助
  ```

* 创建命令：**`/` 代表根目录**

  ```sh
  create /path value			# 创建节点，value 可选
  create -e /path value		# 创建临时节点
  create -s /path value		# 创建顺序节点
  create -es /path value  	# 创建临时顺序节点，比如node10000012 删除12后也会继续从13开始，只会增加
  ```

* 查询命令：

  ```sh
  ls /path					# 显示指定目录下子节点
  ls –s /path					# 查询节点详细信息
  ls –w /path					# 监听子节点数量的变化
  stat /path					# 查看节点状态
  get –s /path				# 查询节点详细信息
  get –w /path				# 监听节点数据的变化
  ```

  ```sh
  # 属性，分为当前节点的属性和子节点属性
  czxid: 节点被创建的事务ID, 是ZooKeeper中所有修改总的次序，每次修改都有唯一的 zxid，谁小谁先发生
  ctime: 被创建的时间戳
  mzxid: 最后一次被更新的事务ID 
  mtime: 最后修改的时间戳
  pzxid: 子节点列表最后一次被更新的事务ID
  cversion: 子节点的变化号，修改次数
  dataversion: 节点的数据变化号，数据的变化次数
  aclversion: 节点的访问控制列表变化号
  ephemeralOwner: 用于临时节点，代表节点拥有者的 session id，如果为持久节点则为0 
  dataLength: 节点存储的数据的长度 
  numChildren: 当前节点的子节点数量
  ```

* 删除命令：

  ```sh
  delete /path				# 删除节点
  deleteall /path				# 递归删除节点
  ```



***



### 数据结构

ZooKeeper 是一个树形目录服务，类似 Unix 的文件系统，每一个节点都被称为 ZNode，每个 ZNode 默认存储 1MB 的数据，节点上会保存数据和节点信息，每个 ZNode 都可以通过其路径唯一标识

节点可以分为四大类：

* PERSISTENT：持久化节点 
* EPHEMERAL：临时节点，客户端和服务器端**断开连接**后，创建的节点删除
* PERSISTENT_SEQUENTIAL：持久化顺序节点，创建 znode 时设置顺序标识，节点名称后会附加一个值，**顺序号是一个单调递增的计数器**，由父节点维护
* EPHEMERAL_SEQUENTIAL：临时顺序节点

注意：在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序

![](https://img.jiapeng.store/img/202307081708420.png)



***



### 代码实现

添加 Maven 依赖：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.7</version>
</dependency>
```

实现代码：

```java
public static void main(String[] args) {
    // 参数一：连接地址
    // 参数二：会话超时时间
    // 参数三：监听器
    ZooKeeper zkClient = new ZooKeeper("192.168.3.128:2181", 20000, new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            System.out.println("监听处理函数");
        }
    });
}
```







***





## 集群介绍

### 相关概念

Zookeepe 集群三个角色：

* Leader 领导者：处理客户端**事务请求**，负责集群内部各服务器的调度

* Follower 跟随者：处理客户端非事务请求，转发事务请求给 Leader 服务器，参与 Leader 选举投票

* Observer 观察者：观察集群的最新状态的变化，并将这些状态进行同步；处理非事务性请求，事务性请求会转发给 Leader 服务器进行处理；不会参与任何形式的投票。只提供非事务性的服务，通常用于在不影响集群事务处理能力的前提下，提升集群的非事务处理能力（提高集群读的能力，但是也降低了集群选主的复杂程度）


相关属性：

* SID：服务器 ID，用来唯一标识一台集群中的机器，和 myid 一致
* ZXID：事务 ID，用来标识一次服务器状态的变更，在某一时刻集群中每台机器的 ZXID 值不一定完全一致，这和 ZooKeeper 服务器对于客户端更新请求的处理逻辑有关

* Epoch：每个 Leader 任期的代号，同一轮选举投票过程中的该值是相同的，投完一次票就增加

选举机制：半数机制，超过半数的投票就通过

* 第一次启动选举规则：投票过半数时，服务器 ID 大的胜出

* 第二次启动选举规则：
  * EPOCH 大的直接胜出
  * EPOCH 相同，事务 ID 大的胜出（事务 ID 越大，数据越新）
  * 事务 ID 相同，服务器 ID 大的胜出





***



### 初次选举

选举过程：

* 服务器 1 启动，发起一次选举，服务器 1 投自己一票，票数不超过半数，选举无法完成，服务器 1 状态保持为 LOOKING
* 服务器 2 启动，再发起一次选举，服务器 1 和 2 分别投自己一票并**交换选票信息**，此时服务器 1 会发现服务器 2 的 SID 比自己投票推举的（服务器 1）大，更改选票为推举服务器 2。投票结果为服务器 1 票数 0 票，服务器 2 票数 2 票，票数不超过半数，选举无法完成，服务器 1、2 状态保持 LOOKING
* 服务器 3 启动，发起一次选举，此时服务器 1 和 2 都会更改选票为服务器 3，投票结果为服务器 3 票数 3 票，此时服务器 3 的票数已经超过半数，服务器 3 当选 Leader，服务器 1、2 更改状态为 FOLLOWING，服务器 3 更改状态为 LEADING
* 服务器 4 启动，发起一次选举，此时服务器 1、2、3 已经不是 LOOKING 状态，不会更改选票信息，交换选票信息结果后服务器 3 为 3 票，服务器 4 为 1 票，此时服务器 4 更改选票信息为服务器 3，并更改状态为 FOLLOWING
* 服务器 5 启动，同 4 一样

![](https://img.jiapeng.store/img/202307081708421.png)



***



### 再次选举

ZooKeeper 集群中的一台服务器出现以下情况之一时，就会开始进入 Leader 选举：

* 服务器初始化启动
* 服务器运行期间无法和 Leader 保持连接

当一台服务器进入 Leader 选举流程时，当前集群可能会处于以下两种状态：

* 集群中本来就已经存在一个 Leader，服务器试图去选举 Leader 时会被告知当前服务器的 Leader 信息，对于该服务器来说，只需要和 Leader 服务器建立连接，并进行状态同步即可

* 集群中确实不存在 Leader，假设服务器 3 和 5 出现故障，开始进行 Leader 选举，SID 为 1、2、4 的机器投票情况

  ```sh
  (EPOCH，ZXID，SID): (1, 8, 1), (1, 8, 2), (1, 7, 4)
  ```

  根据选举规则，服务器 2 胜出



***



### 数据写入

写操作就是事务请求，写入请求直接发送给 Leader 节点：Leader 会先将数据写入自身，同时通知其他 Follower 写入，**当集群中有半数以上节点写入完成**，Leader 节点就会响应客户端数据写入完成

<img src="https://img.jiapeng.store/img/202307081708422.png" style="zoom: 50%;" />

写入请求直接发送给 Follower 节点：Follower 没有写入权限，会将写请求转发给 Leader，Leader 将数据写入自身，通知其他 Follower 写入，当集群中有半数以上节点写入完成，Leader 会通知 Follower 写入完成，**由 Follower 响应客户端数据写入完成**

<img src="https://img.jiapeng.store/img/202307081708423.png" style="zoom:50%;" />





****





## 底层协议

### Paxos

Paxos 算法：基于消息传递且具有高度容错特性的一致性算法

优点：快速正确的在一个分布式系统中对某个数据值达成一致，并且保证不论发生任何异常，都不会破坏整个系统的一致性

缺陷：在网络复杂的情况下，可能很久无法收敛，甚至陷入活锁的情况



***



### ZAB

#### 算法介绍

ZAB 协议借鉴了 Paxos 算法，是为 Zookeeper 设计的支持崩溃恢复的原子广播协议，基于该协议 Zookeeper 设计为只有一台客户端（Leader）负责处理外部的写事务请求，然后 Leader 将数据同步到其他 Follower 节点

Zab 协议包括两种基本的模式：消息广播、崩溃恢复



***



#### 消息广播

ZAB 协议针对事务请求的处理过程类似于一个**两阶段提交**过程：广播事务阶段、广播提交操作

* 客户端发起写操作请求，Leader 服务器将请求转化为事务 Proposal 提案，同时为 Proposal 分配一个全局的 ID，即 ZXID
* Leader 服务器为每个 Follower 分配一个单独的队列，将广播的 Proposal **依次放到队列**中去，根据 FIFO 策略进行消息发送
* Follower 接收到 Proposal 后，将其以事务日志的方式写入本地磁盘中，写入成功后向 Leader 反馈一个 ACK 响应消息
* Leader 接收到超过半数以上 Follower 的 ACK 响应消息后，即认为消息发送成功，可以发送 Commit 消息
* Leader 向所有 Follower 广播 commit 消息，同时自身也会完成事务提交，Follower 接收到 Commit 后，将上一条事务提交

<img src="https://img.jiapeng.store/img/202307081708424.png" style="zoom:67%;" />

两阶段提交模型可能因为 Leader 宕机带来数据不一致：

* Leader 发起一个事务 Proposal 后就宕机，Follower 都没有 Proposal
* Leader 收到半数 ACK 宕机，没来得及向 Follower 发送 Commit



***



#### 崩溃恢复

Leader 服务器出现崩溃或者由于网络原因导致 Leader 服务器失去了与**过半 Follower的联系**，那么就会进入崩溃恢复模式，崩溃恢复主要包括两部分：Leader 选举和数据恢复

Zab 协议崩溃恢复要求满足以下两个要求：

* 已经被 Leader 提交的提案 Proposal，必须最终被所有的 Follower 服务器正确提交
* 丢弃已经被 Leader 提出的，但是没有被提交的 Proposal

Zab 协议需要保证选举出来的 Leader 需要满足以下条件：

* 新选举的 Leader 不能包含未提交的 Proposal，即新 Leader 必须都是已经提交了 Proposal 的 Follower 节点
* 新选举的 Leader 节点含有**最大的 ZXID**，可以避免 Leader 服务器检查 Proposal 的提交和丢弃工作

<img src="https://img.jiapeng.store/img/202307081708425.png" style="zoom: 67%;" />

数据恢复阶段：

* 完成 Leader 选举后，在正式开始工作之前（接收事务请求提出新的 Proposal），Leader 服务器会首先确认事务日志中的所有 Proposal 是否已经被集群中过半的服务器 Commit
* Leader 服务器需要确保所有的 Follower 服务器能够接收到每一条事务的 Proposal，并且能将所有已经提交的事务 Proposal 应用到内存数据中，所以只有当 Follower 将所有尚未同步的事务 Proposal 都**从 Leader 服务器上同步**，并且应用到内存数据后，Leader 才会把该 Follower 加入到真正可用的 Follower 列表中



****



#### 异常处理

Zab 的事务编号 zxid 设计：

* zxid 是一个 64 位的数字，低 32 位是一个简单的单增计数器，针对客户端每一个事务请求，Leader 在产生新的 Proposal 事务时，都会对该计数器加 1，而高 32 位则代表了 Leader 周期的 epoch 编号
* epoch 为当前集群所处的代或者周期，每次 Leader 变更后都会在 epoch 的基础上加 1，Follower 只服从 epoch 最高的 Leader 命令，所以旧的 Leader 崩溃恢复之后，其他 Follower 就不会继续追随
* 每次选举产生一个新的 Leader，就会从新 Leader 服务器上取出本地事务日志中最大编号 Proposal 的 zxid，从 zxid 中解析得到对应的 epoch 编号，然后再对其加 1 后作为新的 epoch 值，并将低 32 位数字归零，由 0 开始重新生成 zxid

Zab 协议通过 epoch 编号来区分 Leader 变化周期，能够有效避免不同的 Leader 错误的使用了相同的 zxid 编号提出了不一样的 Proposal 的异常情况

Zab 数据同步过程：**数据同步阶段要以 Leader 服务器为准**

* 一个包含了上个 Leader 周期中尚未提交过的事务 Proposal 的服务器启动时，这台机器加入集群中会以 Follower 角色连上 Leader
* Leader 会根据自己服务器上最后提交的 Proposal 和 Follower 服务器的 Proposal 进行比对，让 Follower 进行一个**回退或者前进操作**，到一个已经被集群中过半机器 Commit 的最新 Proposal（源码解析部分详解）





***



### CAP

CAP 理论指的是在一个分布式系统中，Consistency（一致性）、Availability（可用性）、Partition Tolerance（分区容错性）不能同时成立，ZooKeeper 保证的是 CP

* ZooKeeper 不能保证每次服务请求的可用性，在极端环境下可能会丢弃一些请求，消费者程序需要重新请求才能获得结果
* 进行 Leader 选举时**集群都是不可用**

CAP 三个基本需求，因为 P 是必须的，因此分布式系统选择就在 CP 或者 AP 中：

* 一致性：指数据在多个副本之间是否能够保持数据一致的特性，当一个系统在数据一致的状态下执行更新操作后，也能保证系统的数据仍然处于一致的状态
* 可用性：指系统提供的服务必须一直处于可用的状态，即使集群中一部分节点故障，对于用户的每一个操作请求总是能够在有限的时间内返回结果
* 分区容错性：分布式系统在遇到任何网络分区故障时，仍然能够保证对外提供服务，不会宕机，除非是整个网络环境都发生了故障







***





## 监听机制

### 实现原理

ZooKeeper 中引入了 Watcher 机制来实现了发布/订阅功能，客户端注册监听目录节点，在特定事件触发时，ZooKeeper 会通知所有关注该事件的客户端，保证 ZooKeeper 保存的任何的数据的任何改变都能快速的响应到监听应用程序

监听命令：**只能生效一次**，接收一次通知，再次监听需要重新注册

```sh
ls –w /path					# 监听【子节点数量】的变化
get –w /path				# 监听【节点数据】的变化
```

工作流程：

* 在主线程中创建 Zookeeper 客户端，这时就会创建**两个线程**，一个负责网络连接通信（connet），一个负责监听（listener）
* 通过 connect 线程将注册的监听事件发送给 Zookeeper
* 在 Zookeeper 的注册监听器列表中将注册的**监听事件添加到列表**中
* Zookeeper 监听到有数据或路径变化，将消息发送给 listener 线程
* listener 线程内部调用 process() 方法

Curator 框架引入了 Cache 来实现对 ZooKeeper 服务端事件的监听，三种 Watcher：

* NodeCache：只是监听某一个特定的节点
* PathChildrenCache：监控一个 ZNode 的子节点
* TreeCache：可以监控整个树上的所有节点，类似于 PathChildrenCache 和 NodeCache 的组合





***



### 监听案例

#### 整体架构

客户端实时监听服务器动态上下线

<img src="https://img.jiapeng.store/img/202307081708426.png" style="zoom:50%;" />



***



#### 代码实现

客户端：先启动客户端进行监听

```java
public class DistributeClient {
    private String connectString = "192.168.3.128:2181";
    private int sessionTimeout = 20000;
    private ZooKeeper zk;

    public static void main(String[] args) throws Exception {
        DistributeClient client = new DistributeClient();
        
        // 1 获取zk连接
        client.getConnect();

        // 2 监听/servers下面子节点的增加和删除
        client.getServerList();

        // 3 业务逻辑
        client.business();
    }

    private void business() throws InterruptedException {
        Thread.sleep(Long.MAX_VALUE);
    }

    private void getServerList() throws KeeperException, InterruptedException {
        ArrayList<String> servers = new ArrayList<>();
        // 获取所有子节点，true 代表触发监听操作
        List<String> children = zk.getChildren("/servers", true);

        for (String child : children) {
            // 获取子节点的数据
            byte[] data = zk.getData("/servers/" + child, false, null);
            servers.add(new String(data));
        }
        System.out.println(servers);
    }

    private void getConnect() throws IOException {
        zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                getServerList();
            }
        });
    }
}
```

服务端：启动时需要 Program arguments

```java
public class DistributeServer {
    private String connectString = "192.168.3.128:2181";
    private int sessionTimeout = 20000;
    private ZooKeeper zk;

    public static void main(String[] args) throws Exception {
        DistributeServer server = new DistributeServer();

        // 1 获取 zookeeper 连接
        server.getConnect();

        // 2  注册服务器到 zk 集群，注意参数
        server.register(args[0]);

        // 3 启动业务逻辑
        server.business();
    }

    private void business() throws InterruptedException {
        Thread.sleep(Long.MAX_VALUE);
    }

    private void register(String hostname) throws KeeperException, InterruptedException {
        // OPEN_ACL_UNSAFE: ACL 开放
        // EPHEMERAL_SEQUENTIAL: 临时顺序节点
        String create = zk.create("/servers/" + hostname, hostname.getBytes(),
                                  ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println(hostname + " is online");
    }

    private void getConnect() throws IOException {
        zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
            }
        });
    }
}
```





***





## 分布式锁

### 实现原理

分布式锁可以实现在分布式系统中多个进程有序的访问该临界资源，多个进程之间不会相互干扰

核心思想：当客户端要获取锁，则创建节点，使用完锁，则删除该节点

1. 客户端获取锁时，在 /locks 节点下创建**临时顺序**节点
   * 使用临时节点是为了防止当服务器或客户端宕机以后节点无法删除（持久节点），导致锁无法释放
   * 使用顺序节点是为了系统自动编号排序，找最小的节点，防止客户端饥饿现象，保证公平
2. 获取 /locks 目录的所有子节点，判断自己的**子节点序号是否最小**，成立则客户端获取到锁，使用完锁后将该节点删除

3. 反之客户端需要找到比自己小的节点，**对其注册事件监听器，监听删除事件**
4. 客户端的 Watcher 收到删除事件通知，就会重新判断当前节点是否是子节点中序号最小，如果是则获取到了锁， 如果不是则重复以上步骤继续获取到比自己小的一个节点并注册监听

![](https://img.jiapeng.store/img/202307081708427.png)



***



### Curator

Curator 实现分布式锁 API，在 Curator 中有五种锁方案：

- InterProcessSemaphoreMutex：分布式排它锁（非可重入锁）

- InterProcessMutex：分布式可重入排它锁

- InterProcessReadWriteLock：分布式读写锁

- InterProcessMultiLock：将多个锁作为单个实体管理的容器

- InterProcessSemaphoreV2：共享信号量

```java
public class CuratorLock {
    
    public static CuratorFramework getCuratorFramework() {
        // 重试策略对象
        ExponentialBackoffRetry policy = new ExponentialBackoffRetry(3000, 3);
        // 构建客户端
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString("192.168.3.128:2181")
                .connectionTimeoutMs(2000)	// 连接超时时间
                .sessionTimeoutMs(20000)	// 会话超时时间 单位ms
                .retryPolicy(policy)		// 重试策略
                .build();

        // 启动客户端
        client.start();
        System.out.println("zookeeper 启动成功");
        return client;
    }
    
    public static void main(String[] args) {
        // 创建分布式锁1
        InterProcessMutex lock1 = new InterProcessMutex(getCuratorFramework(), "/locks");

        // 创建分布式锁2
        InterProcessMutex lock2 = new InterProcessMutex(getCuratorFramework(), "/locks");

        new Thread(new Runnable() {
            @Override
            public void run() {
                lock1.acquire();
                System.out.println("线程1 获取到锁");

                Thread.sleep(5 * 1000);

                lock1.release();
                System.out.println("线程1 释放锁");
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                lock2.acquire();
                System.out.println("线程2 获取到锁");

                Thread.sleep(5 * 1000);

                lock2.release();
                System.out.println("线程2 释放锁");

            }
        }).start();
    }
}
```

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>4.3.0</version>
```





***





## 源码解析

### 服务端

服务端程序的入口 QuorumPeerMain

```java
public static void main(String[] args) {
    QuorumPeerMain main = new QuorumPeerMain();
    main.initializeAndRun(args);
}
```

initializeAndRun 的工作：

* 解析启动参数

* 提交周期任务，定时删除过期的快照

* 初始化通信模型，默认是 NIO 通信

  ```java
  // QuorumPeerMain#runFromConfig
  public void runFromConfig(QuorumPeerConfig config) {
      // 通信信组件初始化，默认是 NIO 通信
      ServerCnxnFactory cnxnFactory = ServerCnxnFactory.createFactory();
      // 初始化NIO 服务端socket，绑定2181 端口，可以接收客户端请求
      cnxnFactory.configure(config.getClientPortAddress(), config.getMaxClientCnxns(), false);
      // 启动 zk
      quorumPeer.start();
  }
  ```

* 启动 zookeeper

  ```java
  // QuorumPeer#start
  public synchronized void start() {
      if (!getView().containsKey(myid)) {
          throw new RuntimeException("My id " + myid + " not in the peer list");
      }
      // 冷启动数据恢复，将快照中数据恢复到 DataTree
      loadDataBase();
      // 启动通信工厂实例对象
      startServerCnxnFactory();
      try {
          adminServer.start();
      } catch (AdminServerException e) {
          LOG.warn("Problem starting AdminServer", e);
          System.out.println(e);
      }
      // 准备选举环境
      startLeaderElection();
      // 执行选举
      super.start();
  }
  ```

  



***



### 选举机制

#### 环境准备

QuorumPeer#startLeaderElection 初始化选举环境：

```java
synchronized public void startLeaderElection() {
    try {
        // Looking 状态，需要选举
        if (getPeerState() == ServerState.LOOKING) {
            // 选票组件: myid (serverid), zxid, epoch
            // 开始选票时，serverid 是自己，【先投自己】
            currentVote = new Vote(myid, getLastLoggedZxid(), getCurrentEpoch());
        }
    }
    if (electionType == 0) {
        try {
            udpSocket = new DatagramSocket(getQuorumAddress().getPort());
            // 响应投票结果线程
            responder = new ResponderThread();
            responder.start();
        } catch (SocketException e) {
            throw new RuntimeException(e);
        }
    }
    // 创建选举算法实例
    this.electionAlg = createElectionAlgorithm(electionType);
}
```

```java
// zk总的发送和接收队列准备好
protected Election createElectionAlgorithm(int electionAlgorithm){
    // 负责选举过程中的所有网络通信，创建各种队列和集合
    QuorumCnxManager qcm = createCnxnManager();
    QuorumCnxManager.Listener listener = qcm.listener;
    if(listener != null){
        // 启动监听线程, 调用 client = ss.accept()阻塞，等待处理请求
        listener.start();
        // 准备好发送和接收队列准备
        FastLeaderElection fle = new FastLeaderElection(this, qcm);
        // 启动选举线程，【WorkerSender 和 WorkerReceiver】
        fle.start();
        le = fle;
    }
}
```



***



#### 选举源码

当 Zookeeper 启动后，首先都是 Looking 状态，通过选举让其中一台服务器成为 Leader

执行 `super.start()` 相当于执行 `QuorumPeer#run()` 方法

```java
public void run() {
    case LOOKING:
        // 进行选举，选举结束返回最终成为 Leader 胜选的那张选票
        setCurrentVote(makeLEStrategy().lookForLeader());
}
```

FastLeaderElection 类：

* lookForLeader：选举

  ```java
  public Vote lookForLeader() {
      // 正常启动中其他服务器都会向我发送一个投票，保存每个服务器的最新合法有效的投票
      HashMap<Long, Vote> recvset = new HashMap<Long, Vote>();
  	// 存储合法选举之外的投票结果
      HashMap<Long, Vote> outofelection = new HashMap<Long, Vote>();
  	// 一次选举的最大等待时间，默认值是0.2s
      int notTimeout = finalizeWait;
  	// 每发起一轮选举，logicalclock++,在没有合法的epoch 数据之前，都使用逻辑时钟代替
      synchronized(this){
          // 更新逻辑时钟，每进行一次选举，都需要更新逻辑时钟
          logicalclock.incrementAndGet();
          // 更新选票(serverid， zxid, epoch）
          updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
      }
      // 广播选票，把自己的选票发给其他服务器
      sendNotifications();
      // 一轮一轮的选举直到选举成功
      while ((self.getPeerState() == ServerState.LOOKING) && (!stop)){ }
  }
  ```

* sendNotifications：广播选票

  ```java
  private void sendNotifications() {
      // 遍历投票参与者，给每台服务器发送选票
      for (long sid : self.getCurrentAndNextConfigVoters()) {
  		// 创建发送选票
          ToSend notmsg = new ToSend(...);
          // 把发送选票放入发送队列
          sendqueue.offer(notmsg);
      }
  }
  ```

FastLeaderElection 中有 WorkerSender 线程：

* `ToSend m = sendqueue.poll(3000, TimeUnit.MILLISECONDS)`：**阻塞获取要发送的选票**

* `process(m)`：处理要发送的选票

  `manager.toSend(m.sid, requestBuffer)`：发送选票

  * `if (this.mySid == sid)`：如果**消息的接收者 sid 是自己**，直接进入自己的 RecvQueue（自己投自己）

  * `else`：如果接收者是其他服务器，创建对应的发送队列或者复用已经存在的发送队列，把消息放入该队列

  * `connectOne(sid)`：建立连接

    * `sock.connect(electionAddr, cnxTO)`：建立与 sid 服务器的连接

    * `initiateConnection(sock, sid)`：初始化连接

      `startConnection(sock, sid)`：创建并启动发送器线程和接收器线程

      * `dout = new DataOutputStream(buf)`：**获取 Socket 输出流**，向服务器发送数据
      * `din = new DataInputStream(new BIS(sock.getInputStream())))`：通过输入流读取对方发送过来的选票
      * `if (sid > self.getId())`：接收者 sid 比我的大，没有资格给对方发送连接请求的，直接关闭自己的客户端
      * `SendWorker sw`：初始化发送器，并启动发送器线程，线程 run 方法
        * `while (running && !shutdown && sock != null)`：连接没有断开就一直运行
        * `ByteBuffer b = pollSendQueue()`：从发送队列 SendQueue 中获取发送消息
        * `lastMessageSent.put(sid, b)`：更新对于 sid 这台服务器的最近一条消息
        * `send(b)`：**执行发送**
      * `RecvWorker rw`：初始化接收器，并启动接收器线程
        * `din.readFully(msgArray, 0, length)`：输入流接收消息
        * `addToRecvQueue(new Message(messagg, sid))`：将消息放入接收消息 recvQueue 队列

FastLeaderElection 中有 WorkerReceiver 线程

* `response = manager.pollRecvQueue()`：从 RecvQueue 中**阻塞获取出选举投票消息**（其他服务器发送过来的）

<img src="https://img.jiapeng.store/img/202307081708428.png" style="zoom: 50%;" />





***



#### 状态同步

选举结束后，每个节点都需要根据角色更新自己的状态，Leader 更新状态为 Leader，其他节点更新状态为 Follower，整体流程：

* Follower 需要让 Leader 知道自己的状态 (sid, epoch, zxid)
* Leader 接收到信息，**根据信息构建新的 epoch**，要返回对应的信息给 Follower，Follower 更新自己的 epoch
* Leader 需要根据 Follower 的状态，确定何种方式的数据同步 DIFF、TRUNC、SNAP，就是要**以 Leader 服务器数据为准**
  * DIFF：Leader 提交的 zxid 比 Follower 的 zxid 大，发送 Proposal 给 Follower 提交执行
  * TRUNC：Follower 的 zxid 比leader 的 zxid 大，Follower 要进行回滚
  * SNAP：Follower 没有任何数据，直接全量同步
* 执行数据同步，当 Leader 接收到超过半数 Follower 的 Ack 之后，进入正常工作状态，集群启动完成

<img src="https://img.jiapeng.store/img/202307081708429.png" style="zoom:50%;" />

核心函数解析：

* Leader 更新状态入口：`Leader.lead()`
  * `zk.loadData()`：恢复数据到内存
  * `cnxAcceptor = new LearnerCnxAcceptor()`：启动通信组件
    * `s = ss.accept()`：等待其他 Follower 节点向 Leader 节点发送同步状态
    * `LearnerHandler fh `：接收到 Follower 的请求，就创建 LearnerHandler 对象
    * `fh.start()`：启动线程，通过 switch-case 语法判断接收的命令，执行相应的操作
* Follower 更新状态入口：`Follower.followerLeader()`
  * `QuorumServer leaderServer = findLeader()`：查找 Leader
  * `connectToLeader(addr, hostname) `：与 Leader 建立连接
  * `long newEpochZxid = registerWithLeader(Leader.FOLLOWERINFO)`：向 Leader 注册





***



#### 主从工作

Leader：主服务的工作流程

![](https://img.jiapeng.store/img/202307081708430.png)

Follower：从服务的工作流程，核心函数为 `Follower#followLeader()`

* `readPacket(qp)`：读取信息

* `processPacket(qp)`：处理信息

  ```java
  protected void processPacket(QuorumPacket qp) throws Exception{
      switch (qp.getType()) {
          case Leader.PING:
              break;
          case Leader.PROPOSAL:
              break;
          case Leader.COMMIT:
              break;
          case Leader.COMMITANDACTIVATE:
              break;
          case Leader.UPTODATE:
              break;
          case Leader.REVALIDATE:
              break;
          case Leader.SYNC:
              break;
          default:
              break;
      }
  }
  ```



***



### 客户端

![](https://img.jiapeng.store/img/202307081708431.png)











