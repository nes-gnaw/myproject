# kafka笔记

### kafka介绍

Kafka 是一个**分布式**的基于**发布/订阅模式**的**消息队列**。



**消息队列优点：**

解耦、可恢复性、缓冲、灵活性 & 峰值处理能力、异步通信。



**消息队列模式：**

点对点模式：一对一

发布/订阅模式：一对多





**kafka组成：**

1）Producer ：消息生产者

2）Consumer ：消息消费者

3）Consumer Group ：消费者组，由多个 consumer 组成。

4）Broker ：一台 kafka 服务器就是一个 broker。

5）Topic ：可以理解为一个队列。topic 可以分布到多个 broker上。

6）Partition：一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列。

7）Replica：副本，保证该节点上的 partition 数据不丢失。

8）leader：每个分区多个副本的“主”。

9）follower：每个分区多个副本中的“从”。



### kafka使用

 jar 包下载 http://kafka.apache.org/downloads.html



```shell
tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/
mv kafka_2.11-0.11.0.0/ kafka
mkdir logs	#在/opt/module/kafka 目录下创建 logs 文件夹

vim config/server.properties#	修改配置文件

#broker的全局唯一编号，不能重复
broker.id=0
#删除 topic 功能使用
delete.topic.enable=true

#kafka 运行日志存放的路径
log.dirs=/opt/module/kafka/logs

#配置连接 Zookeeper 集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181


#配置环境变量
sudo vim /etc/profile

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin

source /etc/profile

#分发配置,并修改环境变量

#在集群机器上分别启动
#启动
bin/kafka-server-start.sh -daemon config/server.properties
#关闭
bin/kafka-server-stop.sh stop

#集群启动脚本
for i in hadoop102 hadoop103 hadoop104
do
  echo "========== $i ==========" 
  ssh $i '/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties'
done

#查看当前服务器中的所有 topic
bin/kafka-topics.sh --zookeeper hadoop102:2181 --list

#创建 topic
bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 3 --partitions 1 --topic first

#删除 topic
bin/kafka-topics.sh --zookeeper hadoop102:2181 --delete --topic first

#发送消息
bin/kafka-console-producer.sh --broker-list hadoop102:9092 --topic first

#消费消息
bin/kafka-console-consumer.sh \ --zookeeper hadoop102:2181 --topic first
bin/kafka-console-consumer.sh \ --bootstrap-server hadoop102:9092 --from-beginning --topic first
--from-beginning：会把所有的数据读取出来。

#查看某个 Topic 的详情
bin/kafka-topics.sh --zookeeper hadoop102:2181 --describe --topic first

#修改分区数
bin/kafka-topics.sh --zookeeper hadoop102:2181 --alter --topic first --partitions 6

```



### kafka原理



一个topic分为多个partition，一个partition分为多个segment，一个segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，文件夹的命名规则为：topic名称+分区序号。



Kafka 采取了分片和索引机制。



**Kafka 生产者的分区**，方便在集群中扩展，可以提高并发。

**分区原则：**

（1）指明partition，将指明的值作为 partiton 值； 

（2）没有指明 partition 值但有 key ，将 key 的 hash 值与 topic 的 partition  数进行取余得到 partition 值； 

（3）既没有 partition 值又没有 key 值，第一次调用时随机生成一个整数（后面每次调用自增），将值与 topic 可用的 partition 总数取余得到 partition  值， round-robin 算法。



**数据可靠性保证：**

partition 收到 producer 发送的数据后，需要向 producer 发送 ack，收到 ack，就会进行下一轮的发送，否则重新发送。



**副本数据同步策略：**

1. 半数以上follower同步完成，发送 ack，延迟低，选举新的 leader 时，容忍 n 台 节点的故障，需要 2n+1个副 本。

2. 全部完成同步，发送 ack 选举新的 leader 时，容忍 n 台 节点的故障，需要 n+1 个副 本，延迟高。

Kafka 选择了第二种方案。



**ISR：**

**介绍：**Leader 维护了一个动态的 ISR，和 leader 保持同步的 follower 集 合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。

**原理：**如果 follower 长时间未同步数据 ， 该 follower 将被踢出 ISR ， 该时间阈值由replica.lag.time.max.ms 参数设定。

**失败：**Leader 发生故障之后，就会从 ISR 中选举新的 leader。



**ack 应答机制**

​	对数据的可靠性要求不是很高，能够容忍少量丢失， 没必要等 ISR 中的 follower 全部接收成功。可以设置ack机制。

**0级别：**不等待 broker 的 ack，最低的延迟，一接收到还没有写入磁盘就返回，当 broker 故障时有可能**丢失数据**；

**1级别：**等待ack，leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，将会**丢失数据**；

**-1（all）级别：**等待ack，leader 和 follower 全部落盘成功后才返回 ack。但是如果在 follower 同步完成后，发送 ack 之前，leader 发生故障，会造成**数据重复**





**LEO和HW**

LEO：每个副本最大的 offset； 

HW：消费者能见到的最大的 offset，ISR 队列中排名次最小的 LEO。



（1）**follower 故障** follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后，follower 会读取本地磁盘 记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。 等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重新加入 ISR 了。 

（2）**leader 故障** leader 发生故障之后，会从 ISR 中选出一个新的 leader，为保证多个副本的数据一致性，其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 leader 同步数据。





**kafka分区分配策略**

RoundRobin（轮询）

Range（范围）



**offset 的维护**

1. 0.9 版本之前，consumer 默认将 offset 保存在 Zookeeper 中，

2. 0.9 版本开始， consumer 默认将 offset 保存在 Kafka 一个内置的 topic 中，该 topic 为__consumer_offsets。



