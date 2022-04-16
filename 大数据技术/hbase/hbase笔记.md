# hbase笔记

### hbase介绍

分布式、可扩展、支持海量数据存储的 NoSQL 数据库。底层物理存储结构（K-V）。



**hbase组成**

1. Name Space：数据库的 DatabBase。
2. Region：数据库的表。包括 hbase 和 default，hbase 中存放的是 HBase 内置的表， default 表是用户默认使用的命名空间。
3. Column：列族；建表时，只需指明列族。
4. Row key：类似于id，查询数据时只能根据 RowKey 进行检索。
5. Time Stamp：标识数据的不同版本。



**架构组成**

1. region server：Region 的管理者，负责对region内数据的操作。
2. master：所有 Region Server 的管理者，负责对region 的操作。
3. zookeeper：实现Master 的高可用、RegionServer 的监控、集群配置的维护等。
4. HDFS：底层数据存储。



### hbase使用



**Hbase安装**

```shell
1.zookeeper部署
2.hadoop部署
3.hbase安装

tar -zxvf hbase-1.3.1-bin.tar.gz -C /opt/module
#环境变量
export JAVA_HOME=/opt/module/jdk1.6.0_144
export HBASE_MANAGES_ZK=false

#hbase-site.xml 修改内容：
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop102:9000/HBase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <!-- 0.98 后的新变动，之前版本没有.port,默认端口为 60000 -->
  <property>
    <name>hbase.master.port</name>
    <value>16000</value>
  </property>
  <property> 
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop102,hadoop103,hadoop104</value>
  </property>
  <property> 
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/module/zookeeper-3.4.10/zkData</value>
  </property>
</configuration>

#regionservers配置
hadoop102
hadoop103
hadoop104

#软连接hadoop配置文件到hbase
ln -s /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml /opt/module/hbase/conf/core-site.xml
ln -s /opt/module/hadoop-2.7.2/etc/hadoop/hdfs-site.xml /opt/module/hbase/conf/hdfs-site.xml

#集群分发

#启动方式一
bin/hbase-daemon.sh start master
bin/hbase-daemon.sh start regionserver
#启动方式二
bin/start-hbase.sh / bin/stop-hbase.sh

```



**hbase使用**

```sql
进入 HBase 客户端命令行
bin/hbase shell
查看表列表
list

create 'student','info'

put 'student','1001','info:sex','male'
put 'student','1001','info:age','18'
put 'student','1002','info:name','Janna'
put 'student','1002','info:sex','female'
put 'student','1002','info:age','20'

#查看数据
scan 'student'
scan 'student',{STARTROW => '1001', STOPROW => '1001'}
scan 'student',{STARTROW => '1001'}

#查看表结构
describe 'student'

#修改
put 'student','1001','info:name','Nick'
#查看指定行
get 'student','1001'
#统计行数
count 'student'
#删除某 rowkey 的全部数据
deleteall 'student','1001'
#删除某 rowkey 的某一列数据
delete 'student','1002','info:sex'
#清空表数据
truncate 'student'
#删除表，先让表disable 状态
disable 'student'
drop 'student'
#将 info 列族中的数据存放 3 个版本
alter 'student',{NAME=>'info',VERSIONS=>3}
#查询某key的三个版本数据
get 'student','1001',{COLUMN=>'info:name',VERSIONS=>3}
```



### hbase原理



**hbase写流程**

1）Client 先访问 zookeeper，获取meta 表位于哪个 Region Server。 

2）访问对应 Region Server，获取 meta 表，根据读请求的 rowkey， 查询 Region Server 中的 Region 中。并将该 table 的 region 信息以及 meta 表的位置信息缓存在客户端的 meta cache，方便下次访问。 

3）与目标 Region Server 进行通讯； 

4）将数据顺序写入（追加）到 WAL； 

5）将数据写入对应的 MemStore，数据会在 MemStore 进行排序； 

6）向客户端发送 ack； 

7）等达到 MemStore 的刷写时机后，将数据刷写到 HFile。



**MemStore 刷写时机**

1. 当某个 memstroe 的大小达到了 hbase.hregion.memstore.flush.size（默认值 128M），其所在 region 的所有 memstore 都会刷写。当 memstore 的大小达到了定义值时，会阻止继续写入数据。
2. 到达自动刷写的时间，也会触发flush。hbase.regionserver.optionalcacheflushinterval（默认 1 小时）
3. 当 WAL 文件数量超过 hbase.regionserver.max.logs，会刷写（已经废弃， 现无需设置，最大值为 32）



**hbase读流程**

1）Client 先访问 zookeeper，获取 meta 表位于的 Region Server。 

2）访问对应 Region Server，获取 meta 表，根据请求 rowkey， 查询数据位于 Region Server 中的 Region 。并将该 table 的 region 信息以及 meta 表的位置信息缓存在客户端的 meta cache，方便下次。 

3）与目标 Region Server 进行通讯； 

4）分别在 Block Cache（读缓存），MemStore 和 Store File（HFile）中查询数据，并将查到的数据合并。所有数据是指同一条数据的不同版本、不同类型。 

5） 将查询到的数据块缓存到 Block Cache。 

6）合并后的最终结果返回给客户端。



Compaction （合并）分为两种

1. Minor Compaction：会将临近的若干个较小 HFile 合并成一个较大 HFile，不清理过期和删除的数据。
2. Major Compaction：会将一个 Store 下所有 HFile 合并成一个大 HFile，会清理掉过期 和删除的数据。



Region Split（拆分）默认情况，每个 Table 只有一个 Region，随着数据的写入，Region 会自动进行拆分。



**HBase 与 Hive 集成**



**HBase 优化**

高可用：HBase 支持对 HMaster 的高可用配置。

预分区：以将数据所要 投放的分区提前大致的规划好。

RowKey 设计：设计 RowKey  ，让数据均匀的分布于所有的 region 中。

内存优化：不建议分配非常大的堆内存。

基础优化：允许在 HDFS 的文件中追加内容、优化 DataNode 允许的最大文件打开数、优化延迟高的数据操作的等待时间、优化数据的写入效率、设置 RPC 监听数量、优化 HStore 文件大小、优化 HBase 客户端缓存、指定 scan.next 扫描 HBase 所获取的行数、flush、compact、split 机制。



