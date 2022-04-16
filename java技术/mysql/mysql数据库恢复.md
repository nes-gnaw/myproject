mysql数据库恢复



MySQL数据库服务端会存储各时间段的操作日志，log-bin二进制日志。



开启方式：在my.ini的[mysqld]选项下：添加代码：log-bin=E:/mysql_log_bin，记录内容：主要是记录所有的更改数据的语句，可使用mysqlbinlog命令恢复数据。



mysqlbinlog工具

mysqlbinlog是一个查看mysql二进制日志的工具，可以把mysql上面的所有操作记录从日志里导出，这个工具默认的安装路径为： `/usr/local/mysql/bin/mysqlbinlog` 。

可以通过 `find / -name "mysqlbinlog"` 命令查找mysqlbinlog的工具路径。



解析日志命令：

```shell
/usr/bin/mysqlbinlog -v --base64-output=decode-rows  mysql-bin.000977   --start-datetime='2022-01-24 06:00:00' --stop-datetime='2022-01-24 08:00:00' > binlog000977.sql

mysqlbinlog路径 -v --base64-output=decode-rows  bin-log文件路径   --start-datetime='开始时间' --stop-datetime='截止时间' > 生成sql文件

```



筛选命令：

```shell
1. 去除‘UPDATE’所在行的行尾‘,’符号
sed -i -r 's/(UPDATE.*),/\1/g' 0958_v1.sql

2. @1替换为id、@2替换name...
sed -i 's/@1/id/g;s/@2/name/g;s/@3/sex/g;s/@4/address/g' recover.sql

3. ‘@75=’替换为‘class_adviser_id=’
sed -i 's/@75=/class_adviser_id=/g' 0958.sql

4.删除包含"xxx"的行
sed -i '/xxx/d' filename

5.删除第N行
sed -i 'Nd' filename

6.删除第N~M行
sed -i 'N,Md' filename # file的[N,M]行都被删除

7.删除shell变量表示的行号（配合for等语句使用）
sed -i "${var1},${var2}d" filename # 这里引号必须为双引号

8.删除最后一行
sed -i '$d' filename
```





notepad++正则表达式：

```shell

在行尾添加指定字符
$

在行头添加指定字符
^

查询AA开头BB结尾的段落
\AA(.*?)\BB

查询CC开头的行
^CC.*$

匹配新行：筛选时，结果中是否包括换行
```

