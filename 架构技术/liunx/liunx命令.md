# liunx

liunx查看ip

`ip addr`

`ifconfig`

`ifconfig -a `



**1.Linux服务启动命令**

service tomcat start 或 systemctl start tomcat

**2.Linux服务停止命令**

service tomcat stop 或 systemctl stop tomcat 此时tomcat需要做成服务启动

**3. Linux服务重启命令**

service tomcat restart 或 systemctl restart tomcat



**vi编辑**

Esc 退出编辑模式，输入以下命令：

:wq  保存后退出vi（常用）

:wq! 则为强制储存后退出

:w    保存但不退出（常用）

:w!   若文件属性为『只读』时，强制写入该档案

:q    离开 vi （常用）

:q!   若曾修改过档案，又不想储存，使用 ! 为强制离开不储存档案。

:e!   将档案还原到最原始的状态！

G	跳至最后一行

# root权限篇

1. 执行“sudo passwd -u root”，然后输入当前账户的密码

2. 行“sudo passwd root”，然后输入两次欲设置的root密码

3. 此时，新的root密码就已经设置好了。
   执行“su”后输入新的root密码，就可以获得root权限了。

4. root账户开启成功 ，退出root账户 exit

5. 设置登录面板，使其实现root登录

   进入 /usr/share/lightdm/lightdm.conf.d/

6. 编辑: 50-unity-greeter.config

   添加如下代码,保存退出

   user-session=ubuntu

   greeter-show-manual-login=true

   all-guest=false

7. 重启ubuntu-kulin，成功实现root登录，终端显示



# 防火墙篇

**一.Linux下开启/关闭防火墙命令**

1) 永久性生效，重启后不会复原

```
开启： chkconfig iptables on

关闭： chkconfig iptables off
```

2) 即时生效，重启后复原

```
开启： /etc/init.d/iptables start

关闭： /etc/init.d/iptables stop
```

3)开启相关端口

修改/etc/sysconfig/iptables 文件，添加以下内容：

```
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT

-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
```



**二.UBuntu关闭防火墙**

iptables -A INPUT -i !   PPP0   -j ACCEPT



**三.CentOS Linux 防火墙配置及关闭**

```
/sbin/iptables -I INPUT -p tcp –dport 80 -j ACCEPT

/sbin/iptables -I INPUT -p tcp –dport 22 -j ACCEPT

/etc/rc.d/init.d/iptables save
```

这样重启计算机后,防火墙默认已经开放了80和22端口

这里应该也可以不重启计算机：

```
/etc/init.d/iptables restart
```

查看防火墙信息：

```
/etc/init.d/iptables status
```

关闭防火墙服务：

```
/etc/init.d/iptables stop
```

永久关闭

```
chkconfig --level 35 iptables off
```



**四.centos 7 防火墙**

```
systemctl stop firewalld.service
systemctl disable firewalld.service
```

*centos 7 查看防火墙状态*

firewall-cmd --state





## Linux 下安装 Redis

下载并安装：

```
$ wget http://download.redis.io/releases/redis-2.8.17.tar.gz
$ tar xzf redis-2.8.17.tar.gz
$ cd redis-2.8.17
$ make
```

```
需要安装gcc工具；
安装gcc前执行`$sudo apt-get update`,若不成功再执行`$sudo apt-get clean：
$sudo apt-get clean
$sudo apt-get update
$sudo apt-get build-dep gcc`
按照上面处理就可以使用make命令了。
```

启动redis服务：

```
$ cd src
$ ./redis-server
```

通过启动参数告诉redis使用指定配置文件使用下面命令启动

```
$ cd src
$ ./redis-server ../redis.conf
```

启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了

```
$ cd src
$ ./redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```



## Linux 下安装 zookeeper

下载zookeeper源码包

```
$ wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
$ tar -zxvf zookeeper-3.4.14.tar.gz
```

配置

```
进入conf目录：
$  cd zookeeper-3.3.6/conf/
$  ls
$ configuration.xsl  log4j.properties  zoo_sample.cfg

拷贝zoo_samle.cfg为zoo.cfg：
$  cp zoo_sample.cfg zoo.cfg
$  ls
$ configuration.xsl  log4j.properties  zoo.cfg  zoo_sample.cfg
```

编辑zoo.cfg

```
单机模式:不做集群
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/usr/local/mycrosoftware/zookeeper/zookeeper-3.4.14/data
dataLogDir=/usr/local/mycrosoftware/zookeeper/zookeeper-3.4.14/log
# the port at which the clients will connect
clientPort=2181

集群模式:要做集群，内容如下(dataDir目录和server地址需改成你真实部署机器的信息)
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/zookeeper-3.3.6/data
clientPort=2181
server.0=192.168.0.109:2555:3555  
server.1=192.168.0.110:2555:3555  
server.2=192.168.0.111:2555:3555
```

启动

```
$ ./zkServer.sh start
JMX enabled ``by` `default
Using config: /root/zookeeper-3.3.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

测试

```
$ ./zkCli.sh -server 127.0.0.1:2181
```

查看

```
$ ps -aux | grep 'zookeeper'    	#查看进程
$ netstat -anp|grep 2181            #查看zookeeper的端口号命令
$ bin/zkServer.sh stop              #zookeeper 的停止命令
$ bin/zkServer.sh status            #zookeeper 的状态查看命令
```



##### 查看文件

```
ls         		查看目录中的文件
ls -F     	 	查看目录中的文件
ls -l      		显示文件和目录的详细资料
ls -a         	显示隐藏文件
ls *[0-9]*      显示包含数字的文件名和目录名
ls -lSr |more 	以尺寸大小排列文件和目录
```

#####  

#####  查看系统

```
cat /proc/cpuinfo | grep "physical id" | uniq		#查看CPU个数
cat /proc/cpuinfo | grep "cpu cores" | uniq			#查看CPU核数
cat /proc/cpuinfo | grep 'model name' |uniq			#查看CPU型号
cat /proc/meminfo | grep MemTotal					#查看内存
#查看磁盘空间
fdisk -l 					看到的是物理磁盘大小（包括swap分区的物理大小）
df -h 						看到的是文件系统使用状况（不包括swap分区）
du -sh dir1 				估算目录 ‘dir1’ 已经使用的磁盘空间’ 
du -sk * | sort -rn 		以容量大小为依据依次显示文件和目录的大小 

rpm -q -a –qf ‘%10{SIZE}t%{NAME}n’ | sort -k1,1n 
以大小为依据依次显示已安装的rpm包所使用的空间 (fedora, redhat类系统) 

dpkg-query -W -f=’Installed−Size;10tInstalled−Size;10t{Package}n’ | sort -k1,1n 
以大小为依据显示已安装的deb包所使用的空间 (ubuntu, debian类系统)
```



##### df命令

```
1.格式：
df [选项] [文件]

2．命令功能：
显示指定磁盘文件的可用空间。如果没有文件名被指定，则所有当前被挂载的文件系统的可用空间将被显示。默认情况下，磁盘空间将以 1KB 为单位进行显示，除非环境变量 POSIXLY_CORRECT 被指定，那样将以512字节为单位进行显示。

3．命令参数：
必要参数：
-a 			#全部文件系统列表
-h 			#方便阅读方式显示
-H 			#等于“-h”，但是计算式，1K=1000，而不是1K=1024
-i 			#显示inode信息
-k 			#区块为1024字节
-l 			#只显示本地文件系统
-m 			#区块为1048576字节
--no-sync 	#忽略 sync 命令
-P 			#输出格式为POSIX
--sync 		#在取得磁盘信息前，先执行sync命令
-T 			#文件系统类型
选择参数：
--block-size=<区块大小>  #指定区块大小
-t<文件系统类型> 			#只显示选定文件系统的磁盘信息
-x<文件系统类型> 			#不显示选定文件系统的磁盘信息
--help 					#显示帮助信息
--version 				#显示版本信息
df -h		#这条命令再熟悉不过。以更易读的方式显示目前磁盘空间和使用情况。
df -i		#以inode模式来显示磁盘使用情况。
```



##### 压缩与解压缩

```
tar xvf xxx.tar 						#解压tar格式的文件
tar -tvf xxx.tar 						#查看tar文件中包含的文件
tar cf xxx.tar tool 					#把tool目录打包为xxx.tar文件
tar czf xxx.tar.gz tool 				#把tool目录打包且压缩为xxx.tar.gz文件

-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件
这五个是独立的命令，压缩解压都要用到其中一个。

-z：有gzip属性的
-j：有bz2属性的
-v ：显示所有过程
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
-p ：使用原文件的原来属性（属性不会依据使用者而变）

-Z：有compress属性的
-O：将文件解开到标准输出
-P ：可以使用绝对路径来压缩！
-N ：比后面接的日期(yyyy/mm/dd)还要新的才会被打包进新建的文件中！

tar jcvf /var/bak/xxx.tar.bz2  /var/xxx/ 	#创建xxx.tar.bz2文件，压缩率高 
tar xjf xxx.tar.bz2 		#解压tar.bz2格式
gzip -d ge.tar.gz 			#解压.tar.gz文件为.tar文件
unzip phpbb.zip 			#解压zip文件

bunzip2 file1.bz2 			#解压一个叫做 ‘file1.bz2′的文件
bzip2 file1 				#压缩一个叫做 ‘file1′ 的文件

gunzip file1.gz 			#解压一个叫做 ‘file1.gz’的文件
gzip file1 					#压缩一个叫做 ‘file1′的文件
gzip -9 file1 				#最大程度压缩

rar a file1.rar test_file 					#创建一个叫做 ‘file1.rar’ 的包
rar a file1.rar file1 file2 dir1 			#同时压缩 ‘file1′, ‘file2′ 以及目录 ‘dir1′
rar x file1.rar 			#解压rar包
unrar x file1.rar 			#解压rar包
tar -cvf archive.tar file1 					#创建一个非压缩的 tarball
tar -cvf archive.tar file1 file2 dir1 		#创建一个包含了 ‘file1′, ‘file2′ 以及 ‘dir1′的档案文件
tar -tf xxx.tar 			#显示一个包中的内容
tar -xvf xxx.tar 			#释放一个包
tar -xvf xxx.tar -C /tmp 				#将压缩包释放到 /tmp目录下
tar -cvfj xxx.tar.bz2 dir1 				#创建一个bzip2格式的压缩包
tar -xvfj xxx.tar.bz2 					#解压一个bzip2格式的压缩包
tar -cvfz xxx.tar.gz dir1 				#创建一个gzip格式的压缩包
tar -xvfz xxx.tar.gz 					#解压一个gzip格式的压缩包
zip xxx.zip file1 						#创建一个zip格式的压缩包
zip -r xxx.zip file1 file2 dir1 		#将几个文件和目录同时压缩成一个zip格式的压缩包
unzip xxx.zip 			#解压一个zip格式压缩包
```



##### 系统信息 

```
arch 					显示机器的处理器架构(1) 
uname -m 				显示机器的处理器架构(2) 
uname -r 				显示正在使用的内核版本 
dmidecode -q 			显示硬件系统部件 - (SMBIOS / DMI) 
hdparm -i /dev/hda 		罗列一个磁盘的架构特性 - 不行
hdparm -tT /dev/sda 	在磁盘上执行测试性读取操作 - 无用
cat /proc/cpuinfo 		显示CPU info的信息 
cat /proc/interrupts 	显示中断 
cat /proc/meminfo 		校验内存使用 
cat /proc/swaps 		显示哪些swap被使用 
cat /proc/version		显示内核的版本 
cat /proc/net/dev 		显示网络适配器及统计 
cat /proc/mounts 		显示已加载的文件系统 
lspci -tv 				罗列 PCI 设备 
lsusb -tv 				显示 USB 设备 
date 					显示系统日期 
cal 2007 				显示2007年的日历表 
date 041217002007.00 	设置日期和时间 - 月日时分年.秒 
clock -w 				将时间修改保存到 BIOS
```



##### 关机 (系统的关机、重启以及登出 ) 

```
shutdown -h now 		关闭系统(1) 
init 0 					关闭系统(2) 
telinit 0 				关闭系统(3) 
shutdown -h hours:minutes & 	按预定时间关闭系统 
shutdown -c			 			取消按预定时间关闭系统 
shutdown -r now 		重启(1) 
reboot 					重启(2) 
logout 					注销
```



##### 文件和目录 

```
cd /home 				进入 ‘/ home’ 目录’ 
cd .. 					返回上一级目录 
cd ../.. 				返回上两级目录 
cd 						进入个人的主目录 
cd ~user 				进入个人的主目录 
cd - 					返回上次所在的目录 
pwd 					显示工作路径
 
tree 					显示文件和目录由根目录开始的树形结构(1) 
lstree 					显示文件和目录由根目录开始的树形结构(2) -- 无用 
mkdir dir1 				创建一个叫做 ‘dir1’ 的目录’ 
mkdir dir1 dir2 		同时创建两个目录 
mkdir -p /tmp/dir1/dir2 创建一个目录树

rm -f file1 			删除一个叫做 ‘file1’ 的文件’ 
rmdir dir1 				删除一个叫做 ‘dir1’ 的空目录’ 
rm -rf dir1 			删除一个叫做 ‘dir1’ 的目录并同时删除其内容
rmdir -p dir1			递归删除‘dir1’目录及其内部的空目录
rm -rf dir1 dir2 		同时删除两个目录及它们的内容 

mv dir1 new_dir 		重命名/移动 一个目录

cp file1 file2 			复制一个文件 
cp dir1/* . 			复制一个目录下的所有文件到当前工作目录 
cp -a /tmp/dir1 . 		复制一个目录到当前工作目录 
cp -a dir1 dir2 		复制一个目录并命名

ln -s file1 lnk1 		创建一个指向文件或目录的软链接 
ln file1 lnk1 			创建一个指向文件或目录的硬链接 

touch -t 0712250000 file1 	修改一个文件或目录的时间戳 - (YYMMDDhhmm) 
 
iconv -l 				列出已知的编码 
iconv -f code01 -t code02 file01 -o file02  将file01转至file01,从code01至code02.

-f encoding :把字符从encoding编码开始转换。 
-t encoding :把字符转换到encoding编码。 
-l :列出已知的编码字符集合 
-o file :指定输出文件 
-c :忽略输出的非法字符 
-s :禁止警告信息，但不是错误信息 
--verbose :显示进度信息 
-f和-t所能指定的合法字符在-l选项的命令里面都列出来了。 
```



##### 文件搜索 

```
find mt.cgi     						当前目录查找文件名为mt.cgi的文件
find / -name xxx -print         		全局查找xxx文件
find / -name file1 						从 ‘/’ 开始进入根文件系统搜索文件和目录 
find / -user user1 						搜索属于用户 ‘user1’ 的文件和目录 
find /home/user1 -name *.bin 			在目录 ‘/ home/user1’ 中搜索带有’.bin’ 结尾的文件 
find /home/user1 -type f -atime +100 	搜索在过去100天内未被使用过的执行文件 
find /home/user1 -type f -mtime -10 	搜索在10天内被创建或者修改过的文件 
find / -name *.rpm -exec chmod 755 ‘{}’ \; 		搜索以 ‘.rpm’ 结尾的文件并定义其权限 
find / -xdev -name *.rpm 				搜索以 ‘.rpm’ 结尾的文件，忽略光驱、捷盘等可移动设备 
locate *.ps 							寻找以 ‘.ps’ 结尾的文件 - 先运行 ‘updatedb’ 命令 
whereis halt 							显示一个二进制文件、源码或man的位置 
which halt 								显示一个二进制文件或可执行文件的完整路径
```



##### 挂载一个文件系统 

```
mount /dev/hda2 /mnt/hda2 			挂载一个叫做hda2的盘 - 确定目录 ‘/ mnt/hda2’ 已经存在 
umount /dev/hda2 					卸载一个叫做hda2的盘 - 先从挂载点 ‘/ mnt/hda2’ 退出 
fuser -km /mnt/hda2 				当设备繁忙时强制卸载 
umount -n /mnt/hda2 				运行卸载操作而不写入 /etc/mtab 文件- 当文件为只读或当磁盘写满时非常有用 
mount /dev/fd0 /mnt/floppy 			挂载一个软盘 
mount /dev/cdrom /mnt/cdrom 		挂载一个cdrom或dvdrom 
mount /dev/hdc /mnt/cdrecorder 		挂载一个cdrw或dvdrom 
mount /dev/hdb /mnt/cdrecorder 		挂载一个cdrw或dvdrom 
mount -o loop file.iso /mnt/cdrom 	挂载一个文件或ISO镜像文件 
mount -t vfat /dev/hda5 /mnt/hda5 	挂载一个Windows FAT32文件系统 
mount /dev/sda1 /mnt/usbdisk 		挂载一个usb 捷盘或闪存设备 
mount -t smbfs -o username=user,password=pass //WinClient/share /mnt/share 
# 挂载一个windows网络共享
```



##### 用户和群组 

```
groupadd group_name 						创建一个新用户组 
groupdel group_name 						删除一个用户组 
groupmod -n new_group_name old_group_name 	重命名一个用户组 

useradd -c “Name Surname ” -g admin -d /home/user1 -s /bin/bash user1 
# 创建一个属于 “admin” 用户组的用户 

useradd user1 								创建一个新用户 
userdel -r user1 							删除一个用户 ( ‘-r’ 排除主目录) 

usermod -c “User FTP” -g system -d /ftp/user1 -s /bin/nologin user1 
修改用户属性 

passwd 										修改口令 
passwd user1								修改一个用户的口令 (只允许root执行) 
chage -E 2005-12-31 user1 					设置用户口令的失效期限 
pwck 								检查 ‘/etc/passwd’的文件格式和语法修正以及存在的用户 
grpck 								检查 ‘/etc/passwd’ 的文件格式和语法修正以及存在的群组 
newgrp group_name 					登陆进一个新的群组以改变新创建文件的预设群组
```



##### 文件的权限 - 使用 “+” 设置权限，使用 “-” 用于取消 

```
ls -lh 							显示权限 
ls /tmp | pr -T5 -W$COLUMNS 	将终端划分成5栏显示 
chmod ugo+rwx directory1 		设置目录的所有人(u)、群组(g)以及其他人(o)以读（r ）、写(w)和执行(x)的权限 
chmod go-rwx directory1 		删除群组(g)与其他人(o)对目录的读写执行权限 
chown user1 file1 				改变一个文件的所有人属性 
chown -R user1 directory1 		改变一个目录的所有人属性并同时改变改目录下所有文件的属性 
chgrp group1 file1 				改变文件的群组 
chown user1:group1 file1 		改变一个文件的所有人和群组属性 
find / -perm -u+s 				罗列一个系统中所有使用了SUID控制的文件 
chmod u+s /bin/file1 			设置一个二进制文件的 SUID 位 - 运行该文件的用户也被赋予和所有者同样的权限 
chmod u-s /bin/file1 			禁用一个二进制文件的 SUID位 
chmod g+s /home/public 			设置一个目录的SGID 位 - 类似SUID ，不过这是针对目录的 
chmod g-s /home/public 			禁用一个目录的 SGID 位 
chmod o+t /home/public 			设置一个文件的 STIKY 位 - 只允许合法所有人删除文件 
chmod o-t /home/public 			禁用一个目录的 STIKY 位
```



##### 文件的特殊属性 - 使用 “+” 设置权限，使用 “-” 用于取消 

```
chattr +a file1 	只允许以追加方式读写文件 
chattr +c file1 	允许这个文件能被内核自动压缩/解压 
chattr +d file1 	在进行文件系统备份时，dump程序将忽略这个文件 
chattr +i file1 	设置成不可变的文件，不能被删除、修改、重命名或者链接 
chattr +s file1 	允许一个文件被安全地删除 
chattr +S file1 	一旦应用程序对这个文件执行了写操作，使系统立刻把修改的结果写到磁盘 
chattr +u file1 	若文件被删除，系统会允许你在以后恢复这个被删除的文件 
lsattr 				显示特殊的属性
```



##### RPM 包 - （Fedora, Redhat及类似系统） 

```
rpm -ivh package.rpm 					安装一个rpm包 
rpm -ivh –nodeeps package.rpm 			安装一个rpm包而忽略依赖关系警告 
rpm -U package.rpm 						更新一个rpm包但不改变其配置文件 
rpm -F package.rpm 						更新一个确定已经安装的rpm包 
rpm -e package_name.rpm 				删除一个rpm包 
rpm -qa 								显示系统中所有已经安装的rpm包 
rpm -qa | grep httpd 					显示所有名称中包含 “httpd” 字样的rpm包 
rpm -qi package_name 					获取一个已安装包的特殊信息 
rpm -qg “System Environment/Daemons” 	显示一个组件的rpm包 
rpm -ql package_name 					显示一个已经安装的rpm包提供的文件列表 
rpm -qc package_name 					显示一个已经安装的rpm包提供的配置文件列表 
rpm -q package_name –whatrequires		显示与一个rpm包存在依赖关系的列表 
rpm -q package_name –whatprovides 		显示一个rpm包所占的体积 
rpm -q package_name –scripts 			显示在安装/删除期间所执行的脚本l 
rpm -q package_name –changelog 			显示一个rpm包的修改历史 
rpm -qf /etc/httpd/conf/httpd.conf 		确认所给的文件由哪个rpm包所提供 
rpm -qp package.rpm -l 					显示由一个尚未安装的rpm包提供的文件列表 
rpm –import /media/cdrom/RPM-GPG-KEY 	导入公钥数字证书 
rpm –checksig package.rpm 				确认一个rpm包的完整性 
rpm -qa gpg-pubkey 						确认已安装的所有rpm包的完整性 
rpm -V package_name 					检查文件尺寸、 许可、类型、所有者、群组、MD5检查以及最后修改时间 
rpm -Va 								检查系统中所有已安装的rpm包- 小心使用 
rpm -Vp package.rpm 					确认一个rpm包还未安装

rpm2cpio package.rpm | cpio –extract –make-directories *bin* 
# 从一个rpm包运行可执行文件 

rpm -ivh /usr/src/redhat/RPMS/`arch`/package.rpm 
# 从一个rpm源码安装一个构建好的包 

rpmbuild –rebuild package_name.src.rpm 
# 从一个rpm源码构建一个 rpm 包
```



##### YUM 软件包升级器 - （Fedora, RedHat及类似系统） 

```
yum install package_name 			下载并安装一个rpm包 
yum localinstall package_name.rpm 	将安装一个rpm包，使用你自己的软件仓库为你解决所有依赖关系 
yum update package_name.rpm 		更新当前系统中所有安装的rpm包 
yum update package_name 			更新一个rpm包 
yum remove package_name 			删除一个rpm包 
yum list 							列出当前系统中安装的所有包 
yum search package_name 			在rpm仓库中搜寻软件包 
yum clean packages 					清理rpm缓存删除下载的包 
yum clean headers 					删除所有头文件 
yum clean all 						删除所有缓存的包和头文件
```



##### DEB 包 (Debian, Ubuntu 以及类似系统) 

```
dpkg -i package.deb 				安装/更新一个 deb 包 
dpkg -r package_name 				从系统删除一个 deb 包 
dpkg -l 							显示系统中所有已经安装的 deb 包 
dpkg -l | grep httpd 				显示所有名称中包含 “httpd” 字样的deb包 
dpkg -s package_name 				获得已经安装在系统中一个特殊包的信息 
dpkg -L package_name 				显示系统中已经安装的一个deb包所提供的文件列表 
dpkg –contents package.deb 			显示尚未安装的一个包所提供的文件列表 
dpkg -S /bin/ping 					确认所给的文件由哪个deb包提供
```



##### APT 软件工具 (Debian, Ubuntu 以及类似系统) 

```
apt-get install package_name 		安装/更新一个 deb 包 
apt-cdrom install package_name 		从光盘安装/更新一个 deb 包 
apt-get update 						升级列表中的软件包 
apt-get upgrade 					升级所有已安装的软件 
apt-get remove package_name 		从系统删除一个deb包 
apt-get check 						确认依赖的软件仓库正确 
apt-get clean 						从下载的软件包中清理缓存 
apt-cache search searched-package 	返回包含所要搜索字符串的软件包名称
```



##### 查看文件内容 

```
cat file1 从第一个字节开始正向查看文件的内容 
tac file1 从最后一行开始反向查看一个文件的内容 
more file1 查看一个长文件的内容 
less file1 类似于 ‘more’ 命令，但是它允许在文件中和正向操作一样的反向操作 
head -2 file1 查看一个文件的前两行 
tail -2 file1 查看一个文件的最后两行 
tail -f /var/log/messages 实时查看被添加到一个文件中的内容
```



##### 文本处理 

```
cat file1 file2 … | command <> file1_in.txt_or_file1_out.txt general syntax for text manipulation using PIPE, STDIN and STDOUT 
cat file1 | command( sed, grep, awk, grep, etc…) > result.txt 合并一个文件的详细说明文本，并将简介写入一个新文件中 
cat file1 | command( sed, grep, awk, grep, etc…) >> result.txt 合并一个文件的详细说明文本，并将简介写入一个已有的文件中 
grep Aug /var/log/messages 在文件 ‘/var/log/messages’中查找关键词”Aug” 
grep ^Aug /var/log/messages 在文件 ‘/var/log/messages’中查找以”Aug”开始的词汇 
grep [0-9] /var/log/messages 选择 ‘/var/log/messages’ 文件中所有包含数字的行 
grep Aug -R /var/log/* 在目录 ‘/var/log’ 及随后的目录中搜索字符串”Aug” 
sed ‘s/stringa1/stringa2/g’ example.txt 将example.txt文件中的 “string1” 替换成 “string2” 
sed ‘/^/d’ example.txt 从example.txt文件中删除所有空白行  
sed ‘/ *#/d; /^/d’ example.txt 从example.txt文件中删除所有空白行  sed ‘/ *#/d; /^/d’ example.txt 从example.txt文件中删除所有注释和空白行 
echo ‘esempio’ | tr ‘[:lower:]’ ‘[:upper:]’ 合并上下单元格内容 
sed -e ‘1d’ result.txt 从文件example.txt 中排除第一行 
sed -n ‘/stringa1/p’ 查看只包含词汇 “string1”的行 
sed -e ‘s/ *//’ example.txt 删除每一行最后的空白字符  
sed -e ‘s/stringa1//g’ example.txt 从文档中只删除词汇 “string1” 并保留剩余全部  
sed -n ‘1,5p;5q’ example.txt 查看从第一行到第5行内容  
sed -n ‘5p;5q’ example.txt 查看第5行  
sed -e ‘s/00*/0/g’ example.txt 用单个零替换多个零  
cat -n file1 标示文件的行数  
cat example.txt | awk ‘NR%2==1’ 删除example.txt文件中的所有偶数行  
echo a b c | awk ‘{print//’ example.txt 删除每一行最后的空白字符  sed -e ‘s/stringa1//g’ example.txt 从文档中只删除词汇 “string1” 并保留剩余全部  sed -n ‘1,5p;5q’ example.txt 查看从第一行到第5行内容  sed -n ‘5p;5q’ example.txt 查看第5行  sed -e ‘s/00*/0/g’ example.txt 用单个零替换多个零  cat -n file1 标示文件的行数  cat example.txt | awk ‘NR%2==1’ 删除example.txt文件中的所有偶数行  echo a b c | awk ‘{print1}’ 查看一行第一栏 
echo a b c | awk ‘{print 1,1,3}’ 查看一行的第一和第三栏 
paste file1 file2 合并两个文件或两栏的内容 
paste -d ‘+’ file1 file2 合并两个文件或两栏的内容，中间用”+”区分 
sort file1 file2 排序两个文件的内容 
sort file1 file2 | uniq 取出两个文件的并集(重复的行只保留一份) 
sort file1 file2 | uniq -u 删除交集，留下其他的行 
sort file1 file2 | uniq -d 取出两个文件的交集(只留下同时存在于两个文件中的文件) 
comm -1 file1 file2 比较两个文件的内容只删除 ‘file1’ 所包含的内容 
comm -2 file1 file2 比较两个文件的内容只删除 ‘file2’ 所包含的内容 
comm -3 file1 file2 比较两个文件的内容只删除两个文件共有的部分
```



##### 字符设置和文件格式转换 

```
dos2unix filedos.txt fileunix.txt 		将一个文本文件的格式从MSDOS转换成UNIX 
unix2dos fileunix.txt filedos.txt 		将一个文本文件的格式从UNIX转换成MSDOS 
recode ..HTML < page.txt > page.html 	将一个文本文件转换成html 
recode -l | more 						显示所有允许的转换格式
```



##### 文件系统分析 

```
badblocks -v /dev/hda1 					检查磁盘hda1上的坏磁块 
fsck /dev/hda1 							修复/检查hda1磁盘上linux文件系统的完整性 
fsck.ext2 /dev/hda1 					修复/检查hda1磁盘上ext2文件系统的完整性 
e2fsck /dev/hda1 						修复/检查hda1磁盘上ext2文件系统的完整性 
e2fsck -j /dev/hda1 					修复/检查hda1磁盘上ext3文件系统的完整性 
fsck.ext3 /dev/hda1 					修复/检查hda1磁盘上ext3文件系统的完整性 
fsck.vfat /dev/hda1 					修复/检查hda1磁盘上fat文件系统的完整性 
fsck.msdos /dev/hda1 					修复/检查hda1磁盘上dos文件系统的完整性 
dosfsck /dev/hda1 						修复/检查hda1磁盘上dos文件系统的完整性
```



##### 初始化一个文件系统 

```
mkfs /dev/hda1 					在hda1分区创建一个文件系统 
mke2fs /dev/hda1 				在hda1分区创建一个linux ext2的文件系统 
mke2fs -j /dev/hda1 			在hda1分区创建一个linux ext3(日志型)的文件系统 
mkfs -t vfat 32 -F /dev/hda1 	创建一个 FAT32 文件系统 
fdformat -n /dev/fd0 			格式化一个软盘 
mkswap /dev/hda3 				创建一个swap文件系统
```



##### SWAP文件系统 

```
mkswap /dev/hda3 				创建一个swap文件系统 
swapon /dev/hda3 				启用一个新的swap文件系统 
swapon /dev/hda2 /dev/hdb3 		启用两个swap分区
```



##### 备份 

```
dump -0aj -f /tmp/home0.bak /home 					制作一个 ‘/home’目录的完整备份 
dump -1aj -f /tmp/home0.bak /home 					制作一个 ‘/home’目录的交互式备份 
restore -if /tmp/home0.bak 							还原一个交互式备份 
rsync -rogpav –delete /home /tmp 					同步两边的目录 

rsync -rogpav -e ssh –delete /home ip_address:/tmp 	
# 通过SSH通道rsync 
rsync -az -e ssh –delete ip_addr:/home/public /home/local 
# 通过ssh和压缩将一个远程目录同步到本地目录 
rsync -az -e ssh –delete /home/local ip_addr:/home/public 
# 通过ssh和压缩将本地目录同步到远程目录 
dd bs=1M if=/dev/hda | gzip | ssh user@ip_addr ‘dd of=hda.gz’ 
# 通过ssh在远程主机上执行一次备份本地磁盘的操作 

dd if=/dev/sda of=/tmp/file1 						备份磁盘内容到一个文件 

tar -Puf backup.tar /home/user 执行一次对 ‘/home/user’ 目录的交互式备份操作 

( cd /tmp/local/ && tar c . ) | ssh -C user@ip_addr ‘cd /home/share/ && tar x -p’ 
# 通过ssh在远程目录中复制一个目录内容 

( tar c /home ) | ssh -C user@ip_addr ‘cd /home/backup-home && tar x -p’ 
通过ssh在远程目录中复制一个本地目录 

tar cf - . | (cd /tmp/backup ; tar xf - ) 
本地将一个目录复制到另一个地方，保留原有权限及链接 

find /home/user1 -name ‘*.txt’ | xargs cp -av –target-directory=/home/backup/ –parents 
从一个目录查找并复制所有以 ‘.txt’ 结尾的文件到另一个目录 

find /var/log -name ‘*.log’ | tar cv –files-from=- | bzip2 > log.tar.bz2 
查找所有以 ‘.log’ 结尾的文件并做成一个bzip包 

dd if=/dev/hda of=/dev/fd0 bs=512 count=1 
做一个将 MBR (Master Boot Record)内容复制到软盘的动作 

dd if=/dev/fd0 of=/dev/hda bs=512 count=1 
从已经保存到软盘的备份中恢复MBR内容
```



##### 光盘 

```
cdrecord -v gracetime=2 dev=/dev/cdrom -eject blank=fast -force 清空一个可复写的光盘内容 
mkisofs /dev/cdrom > cd.iso 在磁盘上创建一个光盘的iso镜像文件 
mkisofs /dev/cdrom | gzip > cd_iso.gz 在磁盘上创建一个压缩了的光盘iso镜像文件 
mkisofs -J -allow-leading-dots -R -V “Label CD” -iso-level 4 -o ./cd.iso data_cd 创建一个目录的iso镜像文件 
cdrecord -v dev=/dev/cdrom cd.iso 刻录一个ISO镜像文件 
gzip -dc cd_iso.gz | cdrecord dev=/dev/cdrom - 刻录一个压缩了的ISO镜像文件 
mount -o loop cd.iso /mnt/iso 挂载一个ISO镜像文件 
cd-paranoia -B 从一个CD光盘转录音轨到 wav 文件中 
cd-paranoia – “-3” 从一个CD光盘转录音轨到 wav 文件中（参数-3） 
cdrecord –scanbus 扫描总线以识别scsi通道 
dd if=/dev/hdc | md5sum 校验一个设备的md5sum编码，例如一张 CD
```



##### 网络 - （以太网和WIFI无线） 

```
ifconfig eth0 							显示一个以太网卡的配置 
ifup eth0 								启用一个 ‘eth0’ 网络设备 
ifdown eth0 							禁用一个 ‘eth0’ 网络设备 
ifconfig eth0 192.168.1.1 netmask 255.255.255.0 控制IP地址 
ifconfig eth0 promisc 					设置 ‘eth0’ 成混杂模式以嗅探数据包 (sniffing) 
dhclient eth0 							以dhcp模式启用 ‘eth0’ 
route -n show routing table 
route add -net 0/0 gw IP_Gateway configura default gateway 
route add -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.1.1 configure static route to reach network ‘192.168.0.0/16’
route del 0/0 gw IP_gateway remove static route 
echo “1” > /proc/sys/net/ipv4/ip_forward activate ip routing 
hostname show hostname of system 
host www.example.com lookup hostname to resolve name to ip address and viceversa(1) 
nslookup www.example.com lookup hostname to resolve name to ip address and viceversa(2) 
ip link show show link status of all interfaces 
mii-tool eth0 show link status of ‘eth0’ 
ethtool eth0 show statistics of network card ‘eth0’ 
netstat -tup show all active network connections and their PID 
netstat -tupl show all network services listening on the system and their PID 
tcpdump tcp port 80 show all HTTP traffic 
iwlist scan show wireless networks 
iwconfig eth1 show configuration of a wireless network card 
hostname show hostname 
host www.example.com lookup hostname to resolve name to ip address and viceversa 
nslookup www.example.com lookup hostname to resolve name to ip address and viceversa 
whois www.example.com lookup on Whois database
```

**JPS工具**

jps(Java Virtual Machine Process Status Tool)是JDK 1.5提供的一个显示当前所有java进程pid的命令，简单实用，非常适合在linux/unix平台上简单察看当前java进程的一些简单情况。

  我想很多人都是用过unix系统里的ps命令，这个命令主要是用来显示当前系统的进程情况，有哪些进程，及其 id。 jps 也是一样，它的作用是显示当前系统的java进程情况，及其id号。我们可以通过它来查看我们到底启动了几个java进程（因为每一个java程序都会独占一个java虚拟机实例），和他们的进程号（为下面几个程序做准备），并可通过opt来查看这些进程的详细启动参数。

   **使用方法：在当前命令行下打 jps(需要JAVA_HOME，没有的话，到改程序的目录下打) 。**

**jps存放在JAVA_HOME/bin/jps，使用时为了方便请将JAVA_HOME/bin/加入到Path.**

$> **jps**
23991 Jps
23789 BossMain
23651 Resin

 


比较常用的参数：

**-q 只显示pid，不显示class名称,jar文件名和传递给main 方法的参数**
$> **jps -q**
28680
23789
23651

**-m 输出传递给main 方法的参数，在嵌入式jvm上可能是null**

$> **jps -m**
28715 Jps -m
23789 BossMain
23651 Resin -socketwait 32768 -stdout /data/aoxj/resin/log/stdout.log -stderr /data/aoxj/resin/log/stderr.log

**-l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名**

$> **jps -l**
28729 sun.tools.jps.Jps
23789 com.asiainfo.aimc.bossbi.BossMain
23651 com.caucho.server.resin.Resin

**-v 输出传递给JVM的参数**

$> **jps -v**
23789 BossMain
28802 Jps -Denv.class.path=/data/aoxj/bossbi/twsecurity/java/trustwork140.jar:/data/aoxj/bossbi/twsecurity/java/:/data/aoxj/bossbi/twsecurity/java/twcmcc.jar:/data/aoxj/jdk15/lib/rt.jar:/data/aoxj/jd

k15/lib/tools.jar -Dapplication.home=/data/aoxj/jdk15 -Xms8m
23651 Resin -Xss1m -Dresin.home=/data/aoxj/resin -Dserver.root=/data/aoxj/resin -Djava.util.logging.manager=com.caucho.log.LogManagerImpl -

Djavax.management.builder.initial=com.caucho.jmx.MBeanServerBuilderImpl

**sudo jps看到的进程数量最全**

**jps 192.168.0.77**

**列出远程服务器192.168.0.77机器所有的jvm实例，采用rmi协议，默认连接端口为1099**

**（前提是远程服务器提供jstatd服务）**

**注：jps命令有个地方很不好，似乎只能显示当前用户的java进程，要显示其他用户的还是只能用unix/linux的ps命令。**



##### 权限控制

Linux用户分为：拥有者、组群(Group)、其他（other）
linux中的文件属性过分四段，如  -rwzrwz---
第一段  -  是指文件类型 表示这是个普通文件
文件类型部分
-：文件
d：文件夹
l：链接文件
b：供存储周边设备
c：一次性读取装置

第二段  rwz  是指拥有者具有可读可写可执行的权限  

第三段  rwz 是指所属于这个组的成员对于这个文件具有，可读可写可执行的权限      

第四段  --- 是指其他人对于这个文件没有任何权限



##### 查看日志

- `cat service.log`
- `tail -f service.log`
- `vim serivice.log`



`cat service.log | grep 13888888888`					 根据关键字查询

`cat -n service.log | grep 13888888888`			   根据关键字查询+行号



- `sed -n "29496,29516p" service.log`：			从29496行开始检索，到29516行结束
- `cat -n service.log | tail -n +29496 | head -n 20`:    从29496行开始检索，往前推20条



- `cat service.log | grep 13 | more` ：将查询后的结果交由more输出
- `cat service.log | grep 13 > /home/sanwai/aa.txt` 将查询后的结果写到`/home/sanwai/aa.txt`文件上

- `cat service.log | wc -l`有的时候，我们想统计这个日志输出了多少行

##### 查进程

- `ps -ef`
- `ps aux`

列出所有的进程

`ps -ef |grep java` 通过 `|`管道和`grep` 来过滤掉想要查的进程

- `kill -9 processId`：杀掉某个进程

查端口：`netstat -lntup`

```text
l:listening   n:num   t:tcp  u:udp  p:display PID/Program name for sockets

查看当前所有tcp/udp端口的信息
```

查看某个端口详细的信息：`lsof -i:4000`

