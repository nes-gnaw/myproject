# Yarn笔记

### Yarn介绍

Yarn 是一个资源调度平台，负责为运算程序提供服务器运算资源，一个分布式的操作系统平台。



**组件**

**ResourceManager**

（1）处理客户端请求 

（2）监控NodeManager

（3）启动或监控ApplicationMaster

（4）资源的分配与调度

**NodeManager**

（1）管理单个节点上的资源 

（2）处理来自ResourceManager的命令 

（3）处理来自ApplicationMaster的命令

**Container**

Container是资源抽象，它封装某个节点上的多维度资源，如内存、CPU、磁 盘、 网络等。

**ApplicationMaster**

（1）为应用程序申请资源并分配给内部的任务

（2）任务的监控与容错



**调度器**

FIFO、容量、公平（默认：容量）

FIFO 调度器（First In First Out）：单队列，根据提交作业的先后顺序，先来先服务。

容量调度器（Capacity Scheduler）：每个队列可配置一定的资源量。

公平调度器（Fair Scheduler）：同队列所有任务共享资源，在时间尺度上获得公平的资源。



### Yarn使用

```shell
yarn application -list	#查看所有application
yarn application -list -appStates FINISHED	#状态筛选
所有状态：ALL、NEW、NEW_SAVING、SUBMITTED、ACCEPTED、RUNNING、FINISHED、FAILED、KILLED

yarn application -kill application_1612577921195_0001	#Kill掉Application
yarn logs -applicationId application_1612577921195_0001	#查询 Application日志

yarn logs -applicationId application_1612577921195_0001 -containerId container_1612577921195_0001_01_000001		#查询 Container 日志

yarn applicationattempt -list application_1612577921195_0001	#列出所有 Application 尝试的列表
yarn applicationattempt -status appattempt_1612577921195_0001_000001	#打印 ApplicationAttemp状态

yarn container -list appattempt_1612577921195_0001_000001	#列出所有 Container
yarn container -status container_1612577921195_0001_01_000001	#打印 Container 状态

yarn node -list -all	#列出所有节点
yarn rmadmin -refreshQueues		#加载队列配置
yarn queue -status default		#打印队列信息
```



### Yarn深究



Yarn 任务执行页面:

http://hadoop103:8088/cluster/apps