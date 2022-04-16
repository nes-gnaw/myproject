# HDFS笔记

### hdfs介绍

**优点：**hsfs它是分布式的一个文件系统，适合一次写入，多次读出的场景。

**缺点：**不适合低延时数据访问，无法高效的对大量小文件进行存储，仅支持数据append（追加），不支持文件的随机修改。



1）NameNode（nn）：就是Master，一个主管、管理者。 

（1）管理HDFS的名称空间； 

（2）配置副本策略； 

（3）管理数据块（Block）映射信息；

（4）处理客户端读写请求。

 2）DataNode：就是Slave，NameNode 下达命令，DataNode执行实际的操作。

（1）存储实际的数据块； 

（2）执行数据块的读/写操作。

3）Client：就是客户端。 

（1）文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传； 

（2）与NameNode交互，获取文件的位置信息； 

（3）与DataNode交互，读取或者写入数据； 

（4）Client提供一些命令来管理HDFS，比如NameNode格式化； 

（5）Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作；

 4）Secondary NameNode：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。

（1）辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode ； 

（2）在紧急情况下，可辅助恢复NameNode。



**HDFS块的大小设置主要取决于磁盘传输速率。**



### hdfs原理

**hdfs写入执行流程：**

1. 客户端通过**Distributed FileSystem 模块**向 **NameNode** 请求上传文件 ss.avi，**NameNode** 检查目标文件是否已存在，父目录是否存在。

2. **NameNode** 返回是否可以上传。

3. 客户端请求第一个 **Block** 上传到哪几个 DataNode 服务器上。
4. **NameNode** 返回 数 个 **DataNode** 节点，分别为 **dn1、dn2、dn3**。
5. 客户端通过 **FSDataOutputStream 模块**请求 **dn1** 上传数据，**dn1** 收到请求会继续调用 **dn2**，然后 **dn2** 调用 dn3，将这个 **通信管道** 建立完成。
6. dn1、dn2、dn3 **逐级应答** 客户端。
7. 客户端开始往 dn1 上传第一个 Block（先从磁盘读取数据放到一个本地内存缓存）， 以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 **每传一个 packet 会放入一个应答队列等待应答**。
8. 当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。（**重复执行 3-7 步**）。





**hdfs读取执行流程：**

1. 客户端通过 **DistributedFileSystem** 向 **NameNode** 请求下载文件，NameNode 通过**查询元数据**，找到**文件块所在的 DataNode 地址**。
2. 挑选一台 **DataNode**（就近原则，然后随机）服务器，请求读取数据。
3. DataNode 开始**传输数据给客户端**（从磁盘里面读取数据输入流，以 Packet 为单位 来做校验）。
4. 客户端以 Packet 为单位接收，**先在本地缓存，然后写入目标文件**。



**NameNode 工作机制**

第一阶段：NameNode 启动

（1）第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。 

（2）客户端对元数据进行增删改的请求。 

（3）NameNode 记录操作日志，更新滚动日志。 

（4）NameNode 在内存中对元数据进行增删改。



第二阶段：Secondary NameNode 工作

（1）Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode 是否检查结果。 

（2）Secondary NameNode 请求执行 CheckPoint。 

（3）NameNode 滚动正在写的 Edits 日志。 

（4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。 

（5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。 

（6）生成新的镜像文件 fsimage.chkpoint。 

（7）拷贝 fsimage.chkpoint 到 NameNode。 

（8）NameNode 将 fsimage.chkpoint 重新命名成 fsimage。



**Fsimage 和 Edits 解析**

（1）Fsimage文件：HDFS文件系统元数据的一个永久性的检查点，其中包含HDFS文件系统的所有目 录和文件inode的序列化信息。 

（2）Edits文件：存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先 会被记录到Edits文件中。

（3）seen_txid文件保存的是一个数字，就是最后一个edits_的数字。 

（4）每次 NameNode 启动的时候都会将 Fsimage 文件读入内存，加载 Edits 里面的更新操作，保证内存中的元数据信息是最新的、同步的，可以看成 NameNode 启动的时候就将 Fsimage 和 Edits 文件进行了合并。



**DataNode工作机制**

（1）一个数据块在 DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。 

（2）DataNode 启动后向 NameNode 注册，通过后，周期性（6 小时）的向 NameNode 上 报所有的块信息。

（3）DN 向 NN 汇报当前解读信息的时间间隔，默认6小时；DN 扫描自己节点块信息列表的时间，默认 6 小时。

（4）心跳是每 3 秒一次，如果超过 10 分钟没有收到某个 DataNode 的心跳， 则认为该节点不可用。 

（5）集群运行中可以安全加入和退出一些机器。



### hdfs操作

```shell
上传
hadoop fs -mkdir /sanguo	#创建/sanguo 文件夹
hadoop fs -moveFromLocal ./shuguo.txt /sanguo	#从本地剪切粘贴到 HDFS
hadoop fs -copyFromLocal weiguo.txt /sanguo		#从本地拷贝文件到 HDFS
hadoop fs -put ./wuguo.txt /sanguo				#-put：等同于 copyFromLocal
hadoop fs -appendToFile liubei.txt /sanguo/shuguo.txt	#追加一个文件到已经存在的文件末尾

下载
hadoop fs -copyToLocal /sanguo/shuguo.txt ./	#从 HDFS 拷贝到本地
hadoop fs -get /sanguo/shuguo.txt ./shuguo2.txt	#-get：等同于 copyToLocal

操作
hadoop fs -ls /sanguo	#显示目录信息
hadoop fs -cat /sanguo/shuguo.txt	#显示文件内容
hadoop fs -chmod 666 /sanguo/shuguo.txt	#修改文件所属权限
hadoop fs -chown admin:admin /sanguo/shuguo.txt	#修改文件所属权限
hadoop fs -cp /sanguo/shuguo.txt /jinguo	#从 HDFS 的一个路径拷贝到 HDFS 的另一个路径
hadoop fs -mv /sanguo/wuguo.txt /jinguo		#在 HDFS 目录中移动文件
hadoop fs -tail /jinguo/shuguo.txt	#显示一个文件的末尾 1kb 的数据
hadoop fs -rm /sanguo/shuguo.txt	#删除文件或文件夹
hadoop fs -rm -r /sanguo		#递归删除目录及目录里面内容
hadoop fs -du -s -h /jinguo		#统计文件夹的大小信息
hadoop fs -du -h /jinguo		#统计文件夹的大小信息
hadoop fs -setrep 10 /jinguo/shuguo.txt		#设置 HDFS 中文件的副本数量
<这里设置的副本数只是记录在 NameNode 的元数据中，是否真的会有这么多副本，还得
看 DataNode 的数量。因为目前只有 3 台设备，最多也就 3 个副本，只有节点数的增加到 10
台时，副本数才能达到 10。>


oiv 查看 Fsimage 文件
hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-3.1.3/fsimage.xml
oev 查看 Edits 文件
hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-3.1.3/edits.xml
```



**合并CheckPoint 时间设置**

```shell
[hdfs-default.xml]

通常情况下，SecondaryNameNode 每隔一小时执行一次。
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>3600s</value>
</property>

一分钟检查一次操作次数，当操作次数达到 1 百万时，SecondaryNameNode 执行一次。
<property>
  <name>dfs.namenode.checkpoint.txns</name>
  <value>1000000</value>
  <description>操作动作次数</description>
</property>
<property>
  <name>dfs.namenode.checkpoint.check.period</name>
  <value>60s</value>
  <description> 1 分钟检查一次操作次数</description>
</property>
```



### hdfs深究



**数据完整性**

（1）当 DataNode 读取 Block 的时候，它会计算 CheckSum。 

（2）如果计算后的 CheckSum，与 Block 创建时值不一样，说明 Block 已经损坏。 

（3）Client 读取其他 DataNode 上的 Block。 

（4）常见的校验算法 crc（32），md5（128），sha1（160） 

（5）DataNode 在其文件创建后周期验证 CheckSum。



**掉线时限参数设置**

1、DataNode进程死亡或者网络故障造成DataNode 无法与NameNode通信。

2、NameNode不会立即把该节点判定为死亡，要经过超时时长。

3、HDFS默认的超时时长为10分钟+30秒。

4、TimeOut，超时时长的计算公式为：

TimeOut = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval。

默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。



超时时长 hdfs-site.xml 配置

< heartbeat.recheck.interval 的单位为毫秒， dfs.heartbeat.interval 的单位为秒。>

```shell
<property>
  <name>dfs.namenode.heartbeat.recheck-interval</name>
  <value>300000</value>
</property>
<property>
  <name>dfs.heartbeat.interval</name>
  <value>3</value>
</property>
```

