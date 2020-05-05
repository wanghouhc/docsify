## 一.理论解读

### **1.1什么是kafka**

  Kafka最初由Linkedin公司开发，是一个分布式、支持分区的（partition）、多副本的（replica），基于zookeeper协调的分布式消息系统，它的最大的特性就是可以实时的处理大量数据以满足各种需求场景：比如基于hadoop的批处理系统、低延迟的实时系统、storm/Spark流式处理引擎，web/nginx日志、访问日志，消息服务等等，用scala语言编写，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。随着kafka的发展，功能并不局限于消息系统。kafka对消息保存时根据Topic进行归类，发送消息者成为Producer,消息接受者成为Consumer。kafka集群有多个kafka实例组成，每个实例()成为broker。无论是kafka集群，还是producer和consumer都依赖于zookeeper来保证系统可用性集群保存一些meta信息。

### **1.2 Kafka的特性:**

\- 高吞吐量、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group 对partition进行consume操作。

\- 可扩展性：kafka集群支持热扩展

\- 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失

\- 容错性：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）

\- 高并发：支持数千个客户端同时读写

### **1.3 Kafka适用场景**

由于 Kafka存在高容错、高扩展、扩展分布式等特性， Kafka主要应用场景如下：

日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。

消息系统：解耦和生产者和消费者、缓存消息等。

用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。

运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。

流式处理：比如spark streaming和storm

### **1.4 Kafka中的相关概念**

Kafka中发布订阅的对象是topic。可以为每类数据创建一个topic，把向topic发布消息的客户端称作producer，从topic订阅消息的客户端称作consumer。Producers和consumers可以同时从多个topic读写数据。一个kafka集群由一个或多个broker服务器组成，它负责持久化和备份具体的kafka消息。

**Broker(代理者 )**：Kafka集群中的机器 /服务被称为broker， 是一个物理概念。一个Kafka节点就是一个broker，多个broker可以组成一个Kafka集群。注意：一台节点上可以有多个broker，这和HDFS上一个节点是一个namenode或者datanode有所区别。一台机器上的broker数量由server.properties的数量决定。

**Topic：**一类消息，消息存放的目录即主题，例如page view日志、click日志等都可以以topic的形式存在，Kafka集群能够同时负责多个topic的分发。一个broker可以容纳多个topic。

**Partition**：topic物理上的分组，一个topic可以分为多个partition，每个partition是一个有序的队列。在数据的 产生和消费过程中，不需要关注具体存储的Partition在哪个Broker上，只需要指定 Topic 即可，由 Kafka负责将数据和对应的 Partition关联上

**Segment**：partition物理上由多个segment组成，每个Segment存着message信息

**Message (消息 )**：传递的数据对象，主要由四部分构成(offset(偏移量 )、key、value、 timestamp(插入时间 )

消息和数据不是一个概念，消息是对数据的封装，其中的value部分是可以看成是数据。

**Producer** : 生产message发送到topic

**Consumer** : 订阅topic消费message, consumer作为一个线程来消费

**Consumer Group**：一个Consumer Group包含多个consumer, 这个是预先在配置文件中配置好的。各个consumer（consumer 线程）可以组成一个组（Consumer group ），<u>partition中的每个message只能被组（Consumer group ） 中的一个consumer（consumer 线程 ）消费，如果一个message可以被多个consumer（consumer 线程 ） 消费的话，那么这些consumer必须在不同的组。</u>Kafka不支持一个partition中的message由两个或两个以上的consumer thread来处理，即便是来自不同的consumer group的也不行。它不能像AMQ那样可以多个BET作为consumer去处理message，这是因为多个BET去消费一个Queue中的数据的时候，由于要保证不能多个线程拿同一条message，所以就需要行级别悲观锁（for update）,这就导致了consume的性能下降，吞吐量不够。而<u>kafka为了保证吞吐量，只允许一个consumer线程去访问一个partition。如果觉得效率不高的时候，可以加partition的数量来横向扩展（同一个topic的数据被分放到多个partion中都是一样的，就相当于为这些数据多修了路来将数据分散运输），</u>那么再加新的consumer thread去消费。这样没有锁竞争，充分发挥了横向的扩展性，吞吐量极高。这也就形成了分布式消费的概念。

![](\assets\20200505164819.png)

### **1.4 Kafka的拓扑图：**

客户端和服务端通过TCP协议通信。Kafka提供了Java客户端，并且对多种语言都提供了支持

#### 1.4.1 整体架构图

![](\assets\20200505162916.png)

![](\assets\20200505164411.png)

####  1.4.2 Topics 和Logs       

一个topic是对一组消息的归纳。对每个topic，Kafka 对它的日志进行了分区，如下图所示

####       ![](\assets\20200505163358.png)                      

​       每个分区都由一系列有序的、不可变的消息组成，这些消息被连续的追加到分区中。分区中的每个消息都有一个连续的序列号叫做offset,用来在分区中唯一的标识这个消息。


在一个可配置的时间段内，Kafka集群保留所有发布的消息，不管这些消息有没有被消费。比如，如果消息的保存策略被设置为2天，那么在一个消息被发布的两天时间内，它都是可以被消费的。之后它将被丢弃以释放空间。Kafka的性能是和数据量无关的常量级的，所以保留太多的数据并不是问题。


实际上每个consumer唯一需要维护的数据是消息在日志中的位置，也就是offset。这个offset由consumer来维护：一般情况下随着consumer不断的读取消息，这offset的值不断增加，但其实consumer可以以任意的顺序读取消息，比如它可以将offset设置成为一个旧的值来重读之前的消息。

以上特点的结合，使Kafka consumers非常的轻量级：它们可以在不对集群和其他consumer造成影响的情况下读取消息。你可以使用命令行来"tail"消息而不会对其他正在消费消息的consumer造成影响。

将日志分区可以达到以下目的：首先这使得每个日志的数量不会太大，可以在单个服务上保存。另外每个分区可以单独发布和消费，为并发操作topic提供了一种可能。

#### 1.4.3 分布式                                                         

每个分区在Kafka集群的若干服务中都有副本，副本数量是可以配置的。副本使Kafka具备了容错能力。

每个分区都由一个服务器作为“leader”，零或若干服务器作为“followers”,  leader负责处理消息的读和写，followers则去复制leader。如果leader down了，followers中的一台则会被选为新leader。集群中的每个服务都会同时扮演两个角色：作为它所持有的一部分分区的leader，同时作为其他分区的followers，这样集群就会据有较好的负载均衡。

**Producers**
Producer将消息发布到它指定的topic中, 并负责决定发布到哪个分区。通常简单的由负载均衡机制随机选择分区，但也可以通过特定的分区函数选择分区。使用的更多的是第二种。

**Consumers**
发布消息通常有两种模式：队列模式（queuing）和发布-订阅模式(publish-subscribe)。队列模式中，consumers可以同时从服务端读取消息，每个消息只会被其中一个consumer读到；发布-订阅模式中消息被广播到所有的consumer中。多个consumer可以组成一个consumer 组，共同竞争一个topic，topic中的消息将被分发到组中的一个成员中。同一组中的consumer可以在不同的程序中，也可以在不同的机器上。如果所有的consumer都在一个组中，这就成为了传统的队列模式，在各consumer中实现负载均衡。如果所有的consumer都不在不同的组中，这就成为了发布-订阅模式，所有的消息都被分发到所有的consumer中。更常见的是，每个topic都有若干数量的consumer组，每个组都是一个逻辑上的“订阅者”，为了容错和更好的稳定性，每个组由若干consumer组成。这其实就是一个发布-订阅模式，只不过订阅者是个组而不是单个consumer。
![](\assets\20200505164228.png)

##   二.使用入门

### 1.下载源代码并解压

```
> tar -xzf kafka_2.12-2.5.0.tgz
> cd kafka_2.12-2.5.0
```

### 2.启动zookeeper服务器

  Kafka使用[ZooKeeper，](https://zookeeper.apache.org/)因此如果您还没有，请首先启动ZooKeeper服务器。您可以使用kafka随附的便利脚本来获取快速且肮脏的单节点ZooKeeper实例。

#### 2.1 开启外置zookeeper 

#####     2.1.1 单个zookeeper

此模式主要用于**开发人员本地环境下测试代码**

###### 1. 解压 Zookeeper 并进入其根目录

```bash
tar xzf zookeeper-3.4.9.tar.gz -C /usr/local/
cd /usr/local/zookeeper-3.4.9
```

###### 2. 创建配置文件 conf/zoo.cfg

```bash
cp conf/zoo_sample.cfg conf/zoo.cfg
```

###### 3. 修改内容如下:

```bash
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/var/lib/zookeeper/data
dataLogDir=/var/lib/zookeeper/logs
clientPort=2181
```

- tickTime: 是 zookeeper 的最小时间单元的长度(以毫秒为单位), 它被用来设置心跳检测和会话最小超时时间(tickTime 的两倍)
- initLimit: 初始化连接时能容忍的最长 tickTime 个数
- syncLimit: follower 用于同步的最长 tickTime 个数
- dataDir: 服务器存储 **数据快照** 的目录
- dataLogDir: 服务器存储 **事务日志** 的目录
- clientPort: 用于 client 连接的 server 的端口

其中需要注意的是`dataDir`和`dataLogDir`, 分别是 zookeeper 运行时的数据目录和日志目录, 要保证 **这两个目录已创建** 且 **运行 zookeeper 的用户拥有这两个目录的所有权**

###### 4.测试

- 启动/关闭 Zookeeper:

```bash
bin/zkServer.sh start
bin/zkServer.sh stop
```

- 查看 Zookeeper 状态:

```bash
bin/zkServer.sh status
```

显示 `mode: standalone`, 单机模式.

- 使用 java 客户端连接 ZooKeeper

```bash
./bin/zkCli.sh -server 127.0.0.1:2181
```

然后就可以使用各种命令了, 跟文件操作命令很类似, 输入 help 可以看到所有命令.

##### 2.1.2 集群模式

此模式是 **生产环境中实际使用的模式**

因为 zookeeper 保证 2n + 1 台机器最大允许 n 台机器挂掉, 所以配置集群模式最好是奇数台机器: 3, 5, 7...

最少 3 台构成集群

###### 1 hosts 映射(可选)

```bash
echo "192.168.1.1 zoo1" >> /etc/hosts
echo "192.168.1.2 zoo2" >> /etc/hosts
echo "192.168.1.3 zoo3" >> /etc/hosts
```

###### 2 修改 zookeeper-3.4.9/conf/zoo.cfg 文件

```bash
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/var/lib/zookeeper/data
dataLogDir=/var/lib/zookeeper/log
clientPort=2181
server.1=192.168.1.1:2888:3888
server.2=192.168.1.2:2888:3888
server.3=192.168.1.3:2888:3888
```

与单机模式的不同就是最后三条: `server.X=host:portA:portB`

```bash
server.1=192.168.1.1:2888:3888
server.2=192.168.1.2:2888:3888
server.3=192.168.1.3:2888:3888
```

或

```bash
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```

X 为标识为 X 的机器, host 为其 hostname 或 IP, portA 用于这台机器与集群中的 Leader 机器通信, portB 用于 server 选举 leader.

**PS**: 要配单机伪分布式的话, 可以修改这里为（就是localhost相同）

```bash
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
```

然后每个 zookeeper 实例的 dataDir 和 dataLogDir 配置为不同的即可

###### 3 myid 文件

在标示为 X 的机器上, 将 X 写入 ${dataDir}/myid 文件, 如: 在 192.168.1.2 机器上的 /var/lib/zookeeper/data 目录下建立文件 myid, 写入 2

```bash
echo "2" > /var/lib/zookeeper/data/myid
```

###### 4 开放端口(每台服务器配置)

CentOS 7 使用 firewalld 代替了原来的 iptables, 基本使用如下

```bash
systemctl start firewalld                                      # 启动防火墙
firewall-cmd --state                                           # 检查防火墙状态
firewall-cmd --zone=public --add-port=2888/tcp --permanent     # 永久开启 2888 端口
firewall-cmd --reload                                          # 重新加载防火墙规则
firewall-cmd --list-all                                        # 列出所有防火墙规则
```

把 Zookeeper 用到的端口开放出来

```bash
firewall-cmd --zone=public --add-port=2181/tcp --permanent     # 永久开启 2181 端口
firewall-cmd --zone=public --add-port=2888/tcp --permanent     # 永久开启 2888 端口
firewall-cmd --zone=public --add-port=3888/tcp --permanent     # 永久开启 3888 端口
firewall-cmd --reload                                          # 重新加载防火墙规则
```

###### 5 测试

- 在 **集群中所有机器上** 启动 zookeeper(尽量同时):

```bash
bin/zkServer.sh start
```

- 查看状态, 应该有一台机器显示`mode: leader`, 其余为`mode: follower`

```bash
bin/zkServer.sh status
```

- 使用 java 客户端连接 ZooKeeper

```bash
./bin/zkCli.sh -server 192.168.1.1:2181
```

然后就可以使用各种命令了, 跟文件操作命令很类似, 输入help可以看到所有命令.

- 关闭 zookeeper:

```bash
./bin/zkServer.sh stop
```

#### 2.2 开启kafka内置的zookeeper

##### 2.2.1 单个zookeeper

```
 bin/zookeeper-server-start.sh config/zookeeper.properties
```





#### 2.3 Zookeeper 常见问题

查看状态时, 应该有一台机器显示`mode: leader`, 其余为`mode: follower`

```bash
bin/zkServer.sh status
```

当显示`Error contacting service. It is probably not running.`时, 可以查看日志

```bash
cat zookeeper.out
```

查看 zookeeper.out 日志可以看到是那些机器连不上, 可能是 **网络, ip, 端口, 配置文件, myid 文件** 的问题.
 正常应该是: 先是一些 java 异常, 这是因为 ZooKeeper 集群启动的时候, 每个结点都试图去连接集群中的其它结点, 先启动的肯定连不上后面还没启动的, 所以上面日志前面部分的异常是可以忽略的, 当集群所有的机器的 zookeeper 都启动起来, 就没有异常了, 并选举出来了 leader.

**PS**: 因为 zkServer.sh 脚本中是用 nohup 命令启动 zookeeper 的, 所以 zookeeper.out 文件是在调用 zkServer.sh 时的路径下, 如:用 `bin/zkServer.sh start` 启动则 zookeeper.out 文件在 `zookeeper-3.4.9/` 下; 用 `zkServer.sh start` 启动则 zookeeper.out 文件在 `zookeeper-3.4.9/bin/` 下.

### 3.启动kafka服务器

在启动zookeeper之后启动kafka，kafka设置的zookeeper路径要与我们使用的路径一致

##### 3.1 使用步骤

###### 3.1.1.修改config/server.properties

```
listeners=PLAINTEXT://your.host.name:9092 
//name就是这台kafka服务器要使用的端口号
```

###### 3.1.2启动kafka的服务

```
bin/kafka-server-start.sh config/server.properties
```

###### 3.1.3.建立主题

```
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

###### 3.1.4.查看主题

```
 bin/kafka-topics.sh --list --bootstrap-server localhost:9092
 或者，
 除了手动创建主题外，还可以将代理配置为在发布不存在的主题时自动创建主题。
```

###### 3.1.5 发送消息

```
Kafka带有一个命令行客户端，它将从文件或标准输入中获取输入，并将其作为消息发送到Kafka集群。默认情况下，每行将作为单独的消息发送。
运行生产者，然后在控制台中键入一些消息以发送到服务器。
 bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test
```

###### 3.1.6 启动消费者

```
Kafka还有一个命令行使用者，它将消息转储到标准输出。
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
如果上面的每个命令都在不同的终端上运行，那么您现在应该能够在生产者终端中键入消息，并看到它们出现在消费者终端中。
所有命令行工具都有其他选项。在不带参数的情况下运行该命令将显示用法信息，并对其进行详细记录。
```

###### 3.1.7 设置多个broker

```
对于Kafka来说，单个代理只是一个大小为1的集群，因此除了启动更多的代理实例之外，没有什么太大的变化。但是，只是为了感受一下，让我们将集群扩展到三个节点（仍然全部在本地计算机上）
> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties

编辑这些新文件并设置以下属性：
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://localhost:9093
    log.dirs=/tmp/kafka-logs-1
 
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://localhost:9094
    log.dirs=/tmp/kafka-logs-2
该broker.id属性是集群中每个节点的唯一且永久的名称。我们只需要覆盖端口和日志目录，这是因为我们都在同一台计算机上运行它们，并且希望所有代理都不要试图在同一端口上注册或覆盖彼此的数据。
```

我们已经有Zookeeper并启动了单个节点，因此我们只需要启动两个新节点：

```
在不同的窗口中启动
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
```

创建一个具有三个复制因子的新主题

```
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

但是现在有了集群，我们如何知道哪个经纪人在做什么？要查看该命令，请运行“描述主题”命令：

```
> bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
    
这是输出的说明。第一行给出了所有分区的摘要，每一行都给出了有关一个分区的信息。由于该主题只有一个分区，因此只有一行。

“领导者”是负责给定分区的所有读取和写入的节点。每个节点将成为分区的随机选择部分的领导者。
“副本”是为该分区复制日志的节点列表，无论它们是引导者还是当前处于活动状态。
“ isr”是“同步”副本的集合。这是副本列表的子集，当前仍处于活动状态并追随领导者。
```

请注意，在我的示例中，节点1是主题唯一分区的领导者。

我们可以在创建的原始主题上运行相同的命令，以查看其位置：

```
> bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
    因此，这里没有任何惊喜-原始主题没有副本，并且位于服务器0上，这是我们创建群集时集群中唯一的服务器。
```

让我们向我们的新主题发布一些消息：

```
> bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

现在让我们使用这些消息：

```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

现在让我们测试一下容错能力。经纪人1扮演领导者的角色，所以让我们杀死它：

```
> ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
> kill -9 7564
```

在Windows上使用：

```
> wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
ProcessId
6016
> taskkill /pid 6016 /f
```

领导层已切换为关注者之一，并且节点1不再位于同步副本集中：

```
> bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```

但是，即使最初进行写操作的领导者已经下线，消息仍然可供使用：

```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

### 4.[使用Kafka Connect导入/导出数据](http://kafka.apache.org/quickstart#quickstart_kafkaconnect)

从控制台读取数据并将其写回到控制台是一个方便的起点，但是您可能要使用其他来源的数据或将数据从Kafka导出到其他系统。对于许多系统，可以使用Kafka Connect导入或导出数据，而无需编写自定义集成代码。

Kafka Connect是Kafka附带的工具，用于将数据导入和导出到Kafka。它是运行*连接器*的可扩展工具，该 *连接器*实现用于与外部系统进行交互的自定义逻辑。在本快速入门中，我们将看到如何使用简单的连接器运行Kafka Connect，该连接器将数据从文件导入到Kafka主题，并将数据从Kafka主题导出到文件。

首先，我们将从创建一些种子数据开始进行测试：

```
> ``echo` `-e ``"foo\nbar"` `> ``test``.txt
```

或在Windows上：

```
> ``echo` `foo> ``test``.txt``> ``echo` `bar>> ``test``.txt
```

接下来，我们将启动两个以*独立*模式运行的连接器，这意味着它们将在单个本地专用进程中运行。我们提供了三个配置文件作为参数。第一个始终是Kafka Connect流程的配置，其中包含通用配置，例如要连接的Kafka代理和数据的序列化格式。其余的配置文件均指定要创建的连接器。这些文件包括唯一的连接器名称，要实例化的连接器类，以及连接器所需的任何其他配置。

```
> bin``/connect-standalone``.sh config``/connect-standalone``.properties config``/connect-file-source``.properties config``/connect-file-sink``.properties
```

这些样本配置文件（随Kafka一起提供）使用您先前启动的默认本地集群配置并创建两个连接器：第一个是源连接器，该连接器从输入文件中读取行并将每个行生成到Kafka主题，第二个是宿连接器。从Kafka主题读取消息，并在输出文件中将它们作为一行显示。

在启动过程中，您将看到许多日志消息，其中包括一些表明正在实例化连接器的消息。Kafka Connect进程启动后，源连接器应开始从`test.txt`主题中读取行并将其生成到主题`connect-test`，而接收器连接器应开始从主题中读取消息`connect-test` 并将其写入文件`test.sink.txt`。我们可以通过检查输出文件的内容来验证数据已通过整个管道传递：

```
> ``more` `test``.sink.txt``foo``bar
```

请注意，数据存储在Kafka主题中`connect-test`，因此我们也可以运行控制台使用者以查看该主题中的数据（或使用自定义使用者代码进行处理）：

```
> bin``/kafka-console-consumer``.sh --bootstrap-server localhost:9092 --topic connect-``test` `--from-beginning``{``"schema"``:{``"type"``:``"string"``,``"optional"``:``false``},``"payload"``:``"foo"``}``{``"schema"``:{``"type"``:``"string"``,``"optional"``:``false``},``"payload"``:``"bar"``}``...
```

连接器继续处理数据，因此我们可以将数据添加到文件中，并查看它在管道中的移动情况：

```
> ``echo` `Another line>> ``test``.txt
```

您应该看到该行出现在控制台使用者输出和接收器文件中。

### [5.使用Kafka Streams处理数据](http://kafka.apache.org/quickstart#quickstart_kafkastreams)

Kafka Streams是用于构建关键任务实时应用程序和微服务的客户端库，其中输入和/或输出数据存储在Kafka集群中。Kafka Streams结合了在客户端编写和部署标准Java和Scala应用程序的简便性以及Kafka服务器端集群技术的优势，使这些应用程序具有高度可伸缩性，弹性，容错性，分布式等等。此[快速入门示例](http://kafka.apache.org/25/documentation/streams/quickstart)将演示如何运行此库中编码的流应用程序。

## 三.与spring的整合

[整合案例](https://github.com/wanghouhc/springbootDemo/tree/master/kafkaDemo)

  

