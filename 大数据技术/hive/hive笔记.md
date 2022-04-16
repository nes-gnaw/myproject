# hive笔记

### hive介绍

**Hive** 是 **数据统计工具** ；基于 Hadoop 的 **数据仓库工具**，可以将**结构化的数据文件映射为一张表**，并提供**类 SQL** 查询功能。



**hive组成：**

（1）处理的**数据**存储在 **HDFS** 

（2）分析数据**底层的实现**是 **MapReduce** 

（3）**执行程序**运行在 Yarn 上

（4）可通过JDBC/ODBC协议访问hive、WEB浏览器访问等

（5）hive的元数据，包括表名、表所属的数据库、、列/分区字段、 表的类型等，一般使用mysql储存。



### hive使用

1）Hive 官网地址 http://hive.apache.org/

2）文档查看地址 https://cwiki.apache.org/confluence/display/Hive/GettingStarted

3）下载地址 http://archive.apache.org/dist/hive/

4）github 地址 https://github.com/apache/hive



**hive安装**

```shell
tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/

mv apache-hive-3.1.2-bin hive

sudo vim /etc/profile.d/my_env.sh	#修改环境变量
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin

#解决jar冲突
mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar hive/lib/log4j-slf4j-impl-2.10.0.bak

bin/schematool -dbType derby -initSchema	#初始化

bin/hive	#运行hive

#执行hive操作
hive> show databases;
hive> show tables;
hive> create table test(id int);
hive> insert into test values(1);
hive> select * from test;

安装mysql

#查询并卸载自带mysql
rpm -qa|grep mariadb
sudo rpm -e --nodeps mariadb-libs

tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar

#依次执行
sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm

#解决安装报错问题
yum install -y libaio

#删除预安装目录
cd /var/lib/mysql
sudo rm -rf ./*

#初始化数据库
sudo mysqld --initialize --user=mysql
#查看临时密码
sudo cat /var/log/mysqld.log
#启动mysql
sudo systemctl start mysqld
#设置密码
set password = password("新密码");
#修改root用户允许任意ip连接，并刷新配置
update mysql.user set host='%' where user='root';
flush privileges;

#将MySQL的JDBC驱动拷贝到Hive的lib目录下
cp /opt/software/mysql-connector-java-5.1.37.jar $HIVE_HOME/lib

#新建hive配置文件
vim $HIVE_HOME/conf/hive-site.xml

<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- jdbc 连接的 URL -->
 <property>
   <name>javax.jdo.option.ConnectionURL</name>
   <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
 </property>
 
 <!-- jdbc 连接的 Driver-->
 <property>
   <name>javax.jdo.option.ConnectionDriverName</name>
   <value>com.mysql.jdbc.Driver</value>
 </property>
 
 <!-- jdbc 连接的 username-->
 <property>
   <name>javax.jdo.option.ConnectionUserName</name>
   <value>root</value>
 </property>
 
 <!-- jdbc 连接的 password -->
 <property>
   <name>javax.jdo.option.ConnectionPassword</name>
   <value>000000</value>
 </property>

 <!-- Hive 元数据存储版本的验证 -->
 <property>
   <name>hive.metastore.schema.verification</name>
   <value>false</value>
 </property>
 <!--元数据存储授权-->
 <property>
   <name>hive.metastore.event.db.notification.api.auth</name>
   <value>false</value>
 </property>
 
 <!-- Hive 默认在 HDFS 的工作目录 -->
 <property>
   <name>hive.metastore.warehouse.dir</name>
   <value>/user/hive/warehouse</value>
 </property>
</configuration>

#登陆mysql
mysql -uroot -proot
#新建 Hive 元数据库
create database metastore;
quit;

#初始化 Hive 元数据库
schematool -initSchema -dbType mysql -verbose

#使用元数据服务的方式访问 Hive
#修改hive-site.xml 文件中
<!-- 指定存储元数据要连接的地址 -->
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://hadoop102:9083</value>
</property>

hive --service metastore

#使用 JDBC 方式访问 Hive，修改hive-site.xml

<!-- 指定 hiveserver2 连接的 host -->
<property>
  <name>hive.server2.thrift.bind.host</name>
  <value>hadoop102</value>
</property>
<!-- 指定 hiveserver2 连接的端口号 -->
<property>
  <name>hive.server2.thrift.port</name>
  <value>10000</value>
</property>

#启动 hiveserver2
bin/hive --service hiveserver2
#启动 beeline 客户端
bin/beeline -u jdbc:hive2://hadoop102:10000 -n admin


```



**编写hive启动脚本**

```shell
vim $HIVE_HOME/bin/hiveservices.sh

#!/bin/bash
HIVE_LOG_DIR=$HIVE_HOME/logs
if [ ! -d $HIVE_LOG_DIR ]
then
mkdir -p $HIVE_LOG_DIR
fi
#检查进程是否运行正常，参数 1 为进程名，参数 2 为进程端口
function check_process()
{
 pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print 
$2}')
 ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -
d '/' -f 1)
 echo $pid
 [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
}
function hive_start()
{
 metapid=$(check_process HiveMetastore 9083)
 cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 
&"
 [ -z "$metapid" ] && eval $cmd || echo "Metastroe 服务已启动"
 server2pid=$(check_process HiveServer2 10000)
 cmd="nohup hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
 [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2 服务已启动"
}
function hive_stop()
{
metapid=$(check_process HiveMetastore 9083)
 [ "$metapid" ] && kill $metapid || echo "Metastore 服务未启动"
 server2pid=$(check_process HiveServer2 10000)
 [ "$server2pid" ] && kill $server2pid || echo "HiveServer2 服务未启动"
}
case $1 in
"start")
 hive_start
 ;;
"stop")
 hive_stop
 ;;
"restart")
 hive_stop
 sleep 2
 hive_start
 ;;
"status")
 check_process HiveMetastore 9083 >/dev/null && echo "Metastore 服务运行
正常" || echo "Metastore 服务运行异常"
 check_process HiveServer2 10000 >/dev/null && echo "HiveServer2 服务运
行正常" || echo "HiveServer2 服务运行异常"
 ;;
*)
 echo Invalid Args!
 echo 'Usage: '$(basename $0)' start|stop|restart|status'
 ;;
esac

#添加权限
chmod +x $HIVE_HOME/bin/hiveservices.sh
#启动
hiveservices.sh start
```



**hive库使用**

```sql
#创建一个数据库
create database if not exists db_hive;
#创建一个数据库，指定数据库在 HDFS 上存放的位置
create database db_hive2 location '/db_hive2.db';

#显示数据库
show databases;
show databases like 'db_hive*';
#显示数据库信息
desc database db_hive;
#显示数据库详细信息
desc database extended db_hive;
#切换当前数据库
use db_hive;
#修改数据库
alter database db_hive set dbproperties('createtime'='20220830');

#删除空数据库
drop database if exists db_hive2;
#强制删除
drop database db_hive cascade;
```



**hive表使用**

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY (col_name, col_name, ...)
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
[ROW FORMAT row_format]
[STORED AS file_format]
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]

以下统称为管理表或内部表
#创建表
create table if not exists student(
  id int, name string
)
row format delimited fields terminated by '\t'	#确定表的具体列的数据
stored as textfile	#指定存储文件类型（文本）
location '/user/hive/warehouse/student';	#指定表在 HDFS 上的存储位置

#查询结果创建表
create table if not exists student2 as select id, name from student;
#复制表结构创建表
create table if not exists student3 like student;
#查询表的类型
desc formatted student2;


以下统称为外部表（不会直接删除数据）
#创建外部表
create external table if not exists dept(
  deptno int,
  dname string,
  loc int
)
row format delimited fields terminated by '\t';


#内外转换
alter table student2 set tblproperties('EXTERNAL'='TRUE');
TRUE 代表是外部

#更改表名
ALTER TABLE table_name RENAME TO new_table_name

#删除表
drop table dept;
```



**数据导入**

```sql
#加载本地文件到 hive
load data local inpath '/opt/module/hive/datas/student.txt' into table default.student;

#加载 HDFS 文件到 hive 中
1.上传文件到 HDFS
dfs -put /opt/module/hive/data/student.txt /user/admin/hive;
2.加载 HDFS 上数据
load data inpath '/user/admin/hive/student.txt' into table default.student;

#加载数据覆盖表中已有的数据
load data inpath '/user/admin/hive/student.txt' overwrite into table default.student;

#手动录入数据
insert into table student_par values(1,'wangwu'),(2,'zhaoliu');
#通过查询结果覆盖式录入
insert overwrite table student_par select id, name from student where month='201709';
#创建表直接插入查询结果
create table if not exists student3 as select id, name from student;
#创建表直接指定数据位置
create external table if not exists student5(
 	id int, name string
)
row format delimited fields terminated by '\t'
location '/student;
```



**数据导出**

```sql
#将查询的结果导出到本地
insert overwrite local directory '/opt/module/hive/data/export/student' select * from student;

#格式化导出到本地
insert overwrite local directory '/opt/module/hive/data/export/student1' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' select * from student;

#将查询的结果导出到 HDFS 上
insert overwrite directory '/user/atguigu/student2' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' select * from student;

# Hadoop 命令导出到本地
dfs -get /user/hive/warehouse/student/student.txt /opt/module/data/export/student3.txt;

#清除表中数据，只能删除管理表，不能删除外部表中数据
truncate table student;

```



**hive分区、分桶**

```sql
分区；类似于mysql中的索引，采用另加一行的方式进行记录数据，
	 在查询表数据时，会额外拼加一列分区字段数据。
	 当在表中增加好分区后，添加数据时可直接使用。

#增加分区
alter table dept_partition add partition (day='20200404');

#删除分区
alter table dept_partition drop partition (day='20200406');

#查看分区表
show partitions dept_partition;
#查看分区表结构
desc formatted dept_partition;

分桶：类似mysql数据库的分表操作，将hive映射的数据文件进行数量控制。
存放方式：Hive 的分桶采用对分桶字段的值进行哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中。
```



**hive函数**

```sql
#查看系统自带的函数
show functions;
#显示函数用法
desc function upper；
#详细用法
desc function extended upper;

函数
NVL(value,default_value)	#空字段赋值
CASE WHEN THEN ELSE END		#过滤
CONCAT()					#行转列
COLLECT_SET(col)			#只接受基本类型，外加滤重
EXPLODE(col)				#列转行
OVER()						#开窗函数
开窗拓展
CURRENT ROW：当前行
n PRECEDING：往前 n 行数据
n FOLLOWING：往后 n 行数据

UNBOUNDED：起点
UNBOUNDED PRECEDING 表示从前面的起点
UNBOUNDED FOLLOWING 表示到后面的终点

LAG(col,n,default_val)：往前第 n 行数据
LEAD(col,n, default_val)：往后第 n 行数据
NTILE(n)：把有序窗口的行分发到指定数据的组中，编号从 1 开始。
RANK()						#排序相同时会重复，总数不会变
DENSE_RANK() 				#排序相同时会重复，总数会减少
ROW_NUMBER() 				#会根据顺序计算
```



**本地模式**

```shell
减少本地执行时间
set hive.exec.mode.local.auto=true; //开启本地 mr

```

