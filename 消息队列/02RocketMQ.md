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

* Consumer：消息消费者，也称为消息订阅者，负责**接收并消费消息**。
* Consumer ID：一类 Consumer 的标识，这类 Consumer 通常接收并消费一类消息，且消费逻辑一致。

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



