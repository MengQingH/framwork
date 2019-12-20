RocketMQ是一款**分布式、队列模型**的消息中间件，是阿里巴巴集团自主研发的专业消息中间件，借鉴参考了JMS规范的MQ实现，更参考了优秀的开源消息中间件KAFKA,实现了业务削峰、分布式事务的框架。

## 消息收发模型
<br><img src=img/RocketMQ.png><br>

## 基本概念
* Message:消息，消息队列中**信息传递的载体**。
* Message ID:消息的全局唯一标识，由 MQ 系统自动生成，唯一标识某条消息。
* Message Key:消息的业务标识，由消息生产者（Producer）设置，唯一标识某个业务逻辑。

* Topic:消息主题，**一级消息类型，通过 Topic 对消息进行分类**。
* Tag:消息标签，**二级消息类型，用来进一步区分某个 Topic 下的消息分类**。

* Producer:消息生产者，也称为消息发布者，负责**生产并发送消息**。
* Producer ID：一类 Producer 的标识，这类 Producer 通常生产并发送一类消息，且发送逻辑一致。
* Producer Group：一类Producer的集合，同一个Group内的Producer发送同一类消息，且发送逻辑一致。

* Consumer：消息消费者，也称为消息订阅者，负责**接收并消费消息**。
* Push Consumer：Consumer的一种，通常是应用向Consumer注册一个Listener接口，一旦Consumer收到消息，立刻回调Listener接口的方法。
* Pull Consumer：Consumer的一种，通常由应用主动调用Consumer的获取消息方法从Broker获取消息，主动权由应用控制。
* Consumer ID：一类 Consumer 的标识，这类 Consumer 通常接收并消费一类消息，且消费逻辑一致。
* Consumer Group：一类Consumer的集合，同一个Group内的Consumer消费同一类消息，且消费逻辑一致。

## 组件
分别是nameserver、broker、producer和consumer。
* **nameserver**: **存储当前集群所有Brokers信息、Topic跟Broker的对应关系**。

* **Broker**: 集群最核心模块，主要负责**Topic消息存储**、**消费者的消费位点管理**（消费进度）。
* **Producer**: **消息生产者**，每个生产者都有一个ID(编号)，多个生产者实例可以共用同一个ID。同一个ID下所有实例组成一个生产者集群。
* **Consumer**: **消息消费者**，每个订阅者也有一个ID(编号)，多个消费者实例可以共用同一个ID。同一个ID下所有实例组成一个消费者集群。

## 工作流程
<br><img src=img/RocketMQ流程.png><br>

1. 启动Nameserver，Nameserver起来后监听端口，等待Broker、Produer、Consumer连上来，相当于一个路由控制中心。

2. Broker启动，跟所有的Nameserver保持长连接，定时发送心跳包。心跳包中包含当前Broker信息(IP+端口等)以及存储所有topic信息。注册成功后，Nameserver集群中就有Topic跟Broker的映射关系。
3. 收发消息前，先创建topic，创建topic时需要指定该topic要存储在哪些Broker上。也可以在发送消息时自动创建Topic。
4. Producer发送消息，启动时先跟Namesrv集群中的其中一台建立长连接，并从Nameserver中获取当前发送的Topic存在哪些Broker上，然后跟对应的Broker建立长连接，直接向Broker发消息。
5. Consumer跟Producer类似。跟其中一台Nameserver建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。

## 模块特性
### Nameserver：
1. 多个Namesrv之间相互没有通信，单台Namesrv宕机不影响其他Namesrv与集群；即使整个Namesrv集群宕机，已经正常工作的Producer，Consumer，Broker仍然能正常工作，但新起的Producer, Consumer，Broker就无法工作。
2. Nameserver压力不会太大，平时主要开销是在维持心跳和提供Topic-Broker的关系数据。但有一点需要注意，Broker向Namesr发心跳时，会带上当前自己所负责的所有Topic信息，如果Topic个数太多（万级别），会导致一次心跳中，就Topic的数据就几十M，网络情况差的话，网络传输失败，心跳失败，导致Namesrv误认为Broker心跳失败。

### Broker
1. 高并发读写服务：Broker的高并发读写主要是依靠以下两点：
    * 消息顺序写：所有Topic数据同时只会写一个文件，一个文件满1G，再写新文件，真正的**顺序写盘**，使得发消息TPS大幅提高。
    * 消息随机读：RocketMQ尽可能让读命中系统pagecache，因为操作系统访问pagecache时，即使只访问1K的消息，系统也会提前预读出更多的数据，在下次读时就可能命中pagecache，减少IO操作。
2. 负载均衡和动态伸缩
    * 负载均衡：Broker上存Topic信息，Topic由多个队列组成，队列会分散在多个Broker上，而Producer的发送机制保证**消息尽量分布到所有的消息队列**中。
    * 动态伸缩能力（非顺序消息）：Broker的伸缩性体现在两个维度：Topic, Broker。
        * **Topic维度**：假如一个Topic的消息量特别大，但集群水位压力还是很低，就可以扩大该Topic的队列数，Topic的队列数跟发送、消费速度成正比。
        * **Broker维度**：如果集群水位很高了，需要扩容，直接加机器部署Broker就可以。Broker起来后想Namesrv注册，Producer、Consumer通过Namesrv发现新Broker，立即跟该Broker直连，收发消息。
3. 高可靠 高可用
    * 高可用：集群部署时一般都作为主备，备机实时从主机同步消息，如果其中一个主机宕机，备机提供消息服务，但不提供写服务。
    * 高可靠：所有发往broker的消息，有同步刷盘和异步刷盘机制；同步刷盘时，消息写入物理文件才会返回成功，异步刷盘时，只有机器宕机，才会产生消息丢失，broker挂掉可能会发生，但是机器宕机崩溃是很少发生的，除非突然断电。
4. Baoker和Namesrv的心跳机制：单个Broker跟所有Namesrv保持心跳请求，心跳间隔为30秒，心跳请求中包括当前Broker所有的Topic信息。Namesrv会反查Broer的心跳信息，如果某个Broker在2分钟之内都没有心跳，则认为该Broker下线，调整Topic跟Broker的对应关系。但此时Namesrv不会主动通知Producer、Consumer有Broker宕机。

### 消费者
1. 消费者启动时需要指定Namesr地址，与其中一个Namesrv建立长连接。消费者每隔30秒从nameserver中获取所有的topic最新队列情况，这意味着某个broker如果宕机，客户端最多要30秒才能感知，连接建立后，从Namesrv中获取当前消费Topic所涉及的Broker，直连Broker。
2. Consumer跟Broker时长连接，会每隔30秒发心跳信息到Broker。Broker端每10s检查一次当前存货的Consumer，若发现某个Consumer2分钟内没有心跳，就断开与该Consumer的连接，并且向该消费组的其他实例发送通知，触发该消费者集群的负载均衡。
3. 消费端的负载均衡：就是集群消费模式下，同一个ID的所有消费者实例平均消费该Topic的所有队列。消费者有两种消费模式：集群消费、广播消费。
    * 广播消费：每隔消费者消费Topic下所有的队列。
    * 集群消费：一个topic可以由同一个ID下所有的消费者分担消费。具体例子：假如TopicA有6个队列，某个消费者ID起了2个消费者实例，那么每个消费者负责消费3个队列。如果再增加一个消费者ID相同消费者实例，即当前共有3个消费者同时消费6个队列，那每个消费者负责2个队列的消费。

### 生产者
1. Producer启动时，也需要指定Namesrv的地址，从Namesrv集群中选一台建立长连接。如果该Namesrv宕机，会自动连其他Namesrv。直到有可用的Namesrv为止。
2. Producer每30秒从Namesrv获取Topic跟Broker的映射关系，更新到本地内存中。再跟Topic涉及的所有Broker建立长连接，每隔30秒发一次心跳。在Broker端也会每10秒扫描一次当前注册的Producer，如果发现某个Producer超过2分钟都没有发心跳，则断开连接。
3. 生产者端的负载均衡：生产者发送时，会自动轮询当前所有可发送的broker，一条消息发送成功，下次换另外一个broker发送，以达到消息平均落到所有的broker上。这里需要注意一点：假如某个Broker宕机，意味生产者最长需要30秒才能感知到。在这期间会向宕机的Broker发送消息。当一条消息发送到某个Broker失败后，会往该broker自动再重发2次，假如还是发送失败，则抛出发送失败异常。业务捕获异常，重新发送即可。客户端里会自动轮询另外一个Broker重新发送，这个对于用户是透明的。

## 消费方式
* 广播方式：一条消息被**多个Consumer消费**，即使这些Consumer属于同一个Consumer Group，消息也会被Consumer Group中的每一个Consumer都消费一次。可以认为，在广播消费情况下，Consumer Group的划分无意义。在CORBA Notification规范中，消费方式都属于广播消费。在JMS规范中，相当于Pub/Sub模型。

* 集群消费：一个Consumer Group中的**所有Consumer平均分摊消费消息**(组内负载均衡)，例如某个Topic有9条消息，发往一个Consumer Group，该Consumer Group有3个实例(可能是3个进程，或3台机器)，那么每个实例只消费其中的3条消息。CORBA Notification规范中没有此消费方式。在JMS规范中，类似于P2P模型，但是RocketMQ的集群消费功能大于等于JMS的P2P消费。因为集群消费模式下，RocketMQ单个Consumer Group内的消费类似于P2P，但是一个Topic/Queue可以被多个Consumer Group消费。

* 顺序消费：消息**消费的顺序要和发送的顺序保持一致**。在RocketMQ中，该顺序主要指局部顺序，即一类消费为满足顺序性，必须Producer单线程发送，且发送到同一个队列，这样Consumer就可以按照Producer的发送顺序来消费消息。

* 普通顺序消费：顺序消费的一种，无论正常、异常情况下，都能保证消息的顺序。但是一旦宕机，Broker重启，由于队列总数发生变化，哈希取模后定位的队列会变化，导致短暂的消息顺序不一致。如果业务能够容忍在集群异常情况下(如某个Broker宕机或重启)消息出现短暂的乱序，那么使用普通顺序消费比较合适。
* 严格顺序消息：顺序消费的一种，无论正常、异常情况下，都能保证消息的顺序，但是牺牲了分布式的Failover特性，即Broker集群中只要有一个节点不可用，则整个集群都不可用，这大大降低了服务的可用性。如果服务器部署为同步双写模式，此缺陷可通过slave自动切换成master避免，不过仍然可能存在几分钟的服务不可用。目前已知的应用只要数据库的binlog同步强依赖严格顺序消息，其他应用大部分都可以容忍短暂乱序，推荐使用普通顺序消费模式。
