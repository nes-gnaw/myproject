# hadoop笔记

### 组成介绍

hadoop1.x版本 ： MapReduce + HDFS + Common

hadoop2.x版本 ： MapReduce + Yarn + HDFS + Common



**HDFS**：储存hadoop操作数据的文件系统。

​	NameNode：储存文件元数据，文件名、属性、目录结构等，块列表和块所在的DataNode。

​	DataNode：储存文件块数据，块数据的校验和。

​	Secondary NameNode(2nn)：每隔一段时间对NameNode元数据备份。



**Yarn** ：hadoop的资源管理器。

ResourceManager（RM）：整个集群资源（内存、CPU等）的老大。

NodeManager（NM）：单个节点服务器资源老大。

ApplicationMaster（AM）：单个任务运行的老大。

Container：容器，相当一台独立的服务器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等。



**MapReduce** ：数据分析计算的过程。

map：并行处理数据逻辑。

reduce：对map的结果进行汇总。



三者关系：



### 安装使用

1. 搭建主虚拟机 hadoop100，该虚拟机仅供后续克隆使用，不直接在内部操作系统功能。

   ```shell
   搭建虚拟机
   1.安装所求插件：
   yum install -y epel-release	#提供额外的软件包，相当于是一个软件仓库
   yum install -y net-tools	#net-tool：工具包集合，包含 ifconfig 等命令
   yum install -y vim	#vim：编辑器
   
   2.关闭防火墙：
   systemctl stop firewalld	#关闭防火墙
   systemctl disable firewalld.service	#关闭防火墙开机自启
   
   3.创建自定义用户，并修改用户的密码
   useradd admin
   passwd admin
   
   4.配置自定义用户具有 root 权限
   vim /etc/sudoers
   
   5.在%wheel 这行下面添加一行
   ## Allow root to run any commands anywhere
   root ALL=(ALL) ALL
   ## Allows people in group wheel to run all commands
   %wheel	ALL=(ALL) ALL
   admin	ALL=(ALL) NOPASSWD:ALL
   
   6.修改静态IP
   vim /etc/sysconfig/network-scripts/ifcfg-ens33
   改成
   DEVICE=ens33
   TYPE=Ethernet
   ONBOOT=yes
   BOOTPROTO=static
   NAME="ens33"
   IPADDR=192.168.10.100
   PREFIX=24
   GATEWAY=192.168.10.2
   DNS1=192.168.10.2
   
   7.对VMware和外机电脑同样进行相关设置，修改 windows 的主机映射文件（hosts 文件）
    C:\Windows\System32\drivers\etc
   
   8.修改主机名称
   vim /etc/hostname
   hadoop100
   
   9.配置克隆机主机映射hosts文件
   vim /etc/hosts
   192.168.10.100 hadoop100
   192.168.10.101 hadoop101
   192.168.10.102 hadoop102
   192.168.10.103 hadoop103
   192.168.10.104 hadoop104
   192.168.10.105 hadoop105
   192.168.10.106 hadoop106
   192.168.10.107 hadoop107
   192.168.10.108 hadoop108
   
   10.在/opt 目录下创建 module、software 文件夹
   mkdir /opt/module
   mkdir /opt/software
   
   11.修改 module、software 文件夹的所有者和所属组
   chown admin:admin /opt/module
   chown admin:admin /opt/software
   
   12.卸载虚拟机自带的JDK，最小化安装不需要执行这一步。
   rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps
   
   ➢ rpm -qa：查询所安装的所有 rpm 软件包
   ➢ grep -i：忽略大小写
   ➢ xargs -n1：表示每次只传递一个参数
   ➢ rpm -e –nodeps：强制卸载软件
   
   13.重启
   reboot
   关机
   init 0
   ```

2. 克隆hadoop102，hadoop103，hadoop104虚拟机，为搭建集群做准备。

3. 在hadoop102 内设置所需环境、软件安装包等。并测试是否可以正常使用。

   ```shell
   1.在VMware端对Hadoop100进行克隆操作。
   2.修改主机名、ip等，并重启。
   3.导入jdk、hadoop压缩包，解压。
   tar -zxvf ... /opt/module/	#解压
   mv ...	...	#重命名
   
   配置环境变量，新建/etc/profile.d/my_env.sh 文件。
   sudo vim /etc/profile.d/my_env.sh
   
   #JAVA_HOME
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   export PATH=$PATH:$JAVA_HOME/bin
   #HADOOP_HOME
   export HADOOP_HOME=/opt/module/hadoop-3.1.3
   export PATH=$PATH:$HADOOP_HOME/bin
   export PATH=$PATH:$HADOOP_HOME/sbin
   
   使之生效
   source /etc/profile
   
   测试
   java -version
   hadoop version
   ```

4. 本地测试hadoop。

   ```shell
   创建在 hadoop-3.1.3 文件下面创建一个 wcinput 文件夹
   mkdir wcinput
   vim word.txt
   
   hadoop yarn
   hadoop mapreduce
   atguigu
   atguigu
   
   hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount wcinput wcoutput
   
   查看结果
   cat wcoutput/part-r-00000
   ```

5. 书写集群分发脚本，将其软件分发给其他虚拟机。

   ```shell
   1.scp基本拷贝分发
   scp -r /opt/module/jdk1.8.0_212 admin@hadoop103:/opt/module		#在102分发103
   scp -r admin@hadoop102:/opt/module/hadoop-3.1.3 /opt/module/	#在103拉取102
   scp -r admin@hadoop102:/opt/module/* admin@hadoop104:/opt/module #在103把102分发至104
   
   2.rsync远程同步工具分发
   rsync和scp区别：rsync比scp速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。
   
   rsync -av hadoop-3.1.3/ admin@hadoop103:/opt/module/hadoop-3.1.3/	#在102同步103
   
   3.rsync脚本
   在/home/admin/bin下创建
   vim xsync
   
   #!/bin/bash
   
   #1. 判断参数个数
   if [ $# -lt 1 ]
   then
     echo Not Enough Arguement!
     exit;
   fi
   
   #2. 遍历集群所有机器
   for host in hadoop102 hadoop103 hadoop104
   do
     echo ==================== $host ====================
     #3. 遍历所有目录，挨个发送
     for file in $@
     do
       #4. 判断文件是否存在
       if [ -e $file ]
         then
           #5. 获取父目录
           pdir=$(cd -P $(dirname $file); pwd)
           
           #6. 获取当前文件的名称
           fname=$(basename $file)
           ssh $host "mkdir -p $pdir"
           rsync -av $pdir/$fname $host:$pdir
         else
           echo $file does not exists!
       fi
     done
   done
   
   chmod +x xsync	#修改脚本 xsync 具有执行权限
   sudo cp xsync /bin/	#将脚本复制到/bin 中，以便全局调用
   
   集群SSH无密登录配置，分别在每台机器操作
   cd /home/admin/.ssh
   ssh-keygen -t rsa	#生成公钥和私钥，三次回车
   ssh-copy-id hadoop102
   ssh-copy-id hadoop103
   ssh-copy-id hadoop104
   
   ```

5. 集群的配置和启动

```shell
cd /opt/module/hadoop-3.1.3/etc/hadoop

vim core-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- 指定 NameNode 的地址 -->
 <property>
 	<name>fs.defaultFS</name>
 	<value>hdfs://hadoop102:8020</value>
 </property>
 <!-- 指定 hadoop 数据的存储目录 -->
 <property>
 	<name>hadoop.tmp.dir</name>
 	<value>/opt/module/hadoop-3.1.3/data</value>
 </property>
 <!-- 配置 HDFS 网页登录使用的静态用户为 atguigu -->
 <property>
 	<name>hadoop.http.staticuser.user</name>
 	<value>admin</value>
 </property>
</configuration>

vim hdfs-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- nn web 端访问地址-->
 <property>
 	<name>dfs.namenode.http-address</name>
 	<value>hadoop102:9870</value>
 </property>
 <!-- 2nn web 端访问地址-->
 <property>
 	<name>dfs.namenode.secondary.http-address</name>
 	<value>hadoop104:9868</value>
 </property>
</configuration>

vim yarn-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- 指定MR走shuffle -->
 <property>
 	<name>yarn.nodemanager.aux-services</name>
 	<value>mapreduce_shuffle</value>
 </property>
 <!-- 指定 ResourceManager 的地址-->
 <property>
 	<name>yarn.resourcemanager.hostname</name>
 	<value>hadoop103</value>
 </property>
 <!-- 环境变量的继承 -->
 <property>
 	<name>yarn.nodemanager.env-whitelist</name>
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
 </property>
</configuration>

vim mapred-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!-- 指定 MapReduce 程序运行在 Yarn 上 -->
 <property>
 	<name>mapreduce.framework.name</name>
 	<value>yarn</value>
 </property>
</configuration>

向Hadoop103、104分发配置

vim /opt/module/hadoop-3.1.3/etc/hadoop/workers	#配置集群 workers
hadoop102
hadoop103
hadoop104

在hadoop102节点格式化 NameNode
hdfs namenode -format

hadoop102启动/关闭 HDFS
sbin/start-dfs.sh / sbin/stop-dfs.sh
hadoop103启动/关闭 YARN
sbin/start-yarn.sh / sbin/stop-yarn.sh


hadoop fs -mkdir /input	#创建文件夹
hadoop fs -put $HADOOP_HOME/wcinput/word.txt /input	#上传文件
hadoop fs -get /word.txt ./	#下载
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output	#执行wordcount程序

配置历史服务器
vim mapred-site.xml
<!-- 历史服务器端地址 -->
<property>
  <name>mapreduce.jobhistory.address</name>
  <value>hadoop102:10020</value>
</property>
<!-- 历史服务器 web 端地址 -->
<property>
  <name>mapreduce.jobhistory.webapp.address</name>
  <value>hadoop102:19888</value>
</property>

hadoop102启动历史服务器
mapred --daemon start historyserver

jsp	#查看进程

启动脚本
admin/bin# vim myhadoop.sh

#!/bin/bash

if [ $# -lt 1 ]
then
  echo "No Args Input..."
  exit ;
fi

case $1 in
"start")
  echo " =================== 启动 hadoop 集群 ==================="
  echo " --------------- 启动 hdfs ---------------"
  ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
  echo " --------------- 启动 yarn ---------------"
  ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
  echo " --------------- 启动 historyserver ---------------"
  ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
  echo " =================== 关闭 hadoop 集群 ==================="
  echo " --------------- 关闭 historyserver ---------------"
  ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
  echo " --------------- 关闭 yarn ---------------"
  ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
  echo " --------------- 关闭 hdfs ---------------"
  ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
  echo "Input Args Error..."
;;
esac

chmod +x myhadoop.sh	#分配权限

进程查看脚本
vim jpsall
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
  echo =============== $host ===============
  ssh $host jps 
done

chmod +x jpsall	#分配权限
```



Web 端查看 HDFS 的 NameNode：http://hadoop102:9870 

Web 端查看 YARN 的 ResourceManager：http://hadoop103:8088

Web 端查看历史服务器地址： http://hadoop102:19888/jobhistory



常见错误及解决方案：

集群启动后无法执行wordcount

org.apache.hadoop.mapreduce.v2.app.MRAppMaster



hadoop集群运行wordcount脚本失败解决方案：

[(32条消息) Could not find or load main class org.apache.hadoop.mapreduce.v2.app.MRAppMaster_lsl520hah的博客-CSDN博客](https://blog.csdn.net/lsl520hah/article/details/114372120)



```shell
vim mapred-site.xml

<property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>

hadoop classpaths
<configuration>
  <property>
    <name>yarn.application.classpath</name>
    <value>复制的Hadoop classpath信息</value>
  </property>
</configuration>
```



常用端口号s

| 端口名称                   | Hadoop2.x   | Hadoop3.x        |
| -------------------------- | ----------- | ---------------- |
| NameNode 内部通信端口      | 8020 / 9000 | 8020 / 9000/9820 |
| NameNode HTTP UI           | 50070       | 9870             |
| MapReduce 查看执行任务端口 | 8088        | 8088             |
| 历史服务器通信端口         | 19888       | 19888            |



### 代码编写

### 源码讲解