# shell笔记

脚本编程语言。



shell解析器：bash、sh。



### shell脚本

```shell
案例一：hello world
1.创建
touch helloworld.sh
vi helloworld.sh

#!/bin/bash			#脚本头（必须）
echo "helloworld"	#打印操作

2.执行
方法一：
sh helloworld.sh
方法二：
chmod 777 helloworld.sh
./helloworld.sh	#直接执行必须先给脚本加权限


案例二：利用脚本生成文件
#!/bin/bash
cd /home/atguigu
touch cls.txt
echo "I love cls" >>cls.txt

shell中的变量
特殊变量：$n	
功能描述：n为数字，$0代表脚本名称，$1-$9代表第一到九参数，十以上的大括号包含，如${10}。

#!/bin/bash
echo "$0  $1   $2"

特殊变量：$#
功能描述：获取所有输入参数‘个数’，常用于循环。

#!/bin/bash
echo "$0  $1   $2"
echo $#

特殊变量：$*、$@
$*（命令行中所有参数，$*把所有的参数看成一个整体）
$@（命令行中所有参数，$@把每个参数区分对待）

特殊变量：$？
返回最后一次执行的命令的执行状态。
值为0，证明上一个命令正确执行；
值为非0，证明上一个命令不正确执行。


$*和$@区别：
1.不被双引号“”包含时，都以$1 $2 …$n的形式输出所有参数。
2.被双引号“”包含时，
“$*”会将所有的参数作为一个整体，以“$1 $2 …$n”的形式输出所有参数；
“$@”会将各个参数分开，以“$1” “$2”…”$n”的形式输出所有参数。
```



### linux中命令

```shell
echo $HOME	#查看系统变量的值
set			#显示当前Shell中所有变量

A=5				#自定变量
readonly B=2	#自定义只读变量	
echo $A			#读取自定义变量
unset A			#撤销变量A
C=1+2			#字符串拼接
export B		#提升为全局变量，可在脚本中调用


运算符
expr 运算式  #可配合运算符号使用，输出结果。 
计算语法：$((运算式)) 或 $[运算式]


条件判断
[ 判断式 ] 空为false，非空true

判断符号
<字符串比较> =
<整数比较如下>
-lt 小于（less than）		  -le 小于等于（less equal）
-eq 等于（equal）			  -gt 大于（greater than）
-ge 大于等于（greater equal）	 -ne 不等于（Not equal）

-r 有读的权限（read）			-w 有写的权限（write）
-x 有执行的权限（execute）

-f 文件存在并且是一个常规的文件（file）
-e 文件存在（existence）		-d 文件存在并是一个目录（directory）

```



### shell复杂流程

```shell
if语句
例：
#!/bin/bash
if [ $1 -eq "1" ];then
        echo "banzhang zhen shuai"
        
elif [ $1 -eq "2" ]
then
        echo "cls zhen mei"
fi



case语句
例：
!/bin/bash
case $1 in
"1")
        echo "banzhang"
;;
"2")
        echo "cls"
;;
*)
        echo "renyao"
;;
esac


for循环语句
例：
语法一：
#!/bin/bash
s=0
for((i=0;i<=100;i++))
do
        s=$[$s+$i]
done
echo $s

语法二：
for i in $*
    do
      echo "ban zhang love $i "
    done


while 循环语句
例：
#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
        s=$[$s+$i]
        i=$[$i+1]
done
echo $s



read语法
read -p -t 参数
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒）。
例：read -t 7 -p "Enter your name in 7 seconds " NAME	#等待读取输入内容


系统函数
basename	#删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。
			#suffix 为后缀参数，连带删除被指定的后缀。

dirname		#去除绝对路径的文件名，返回剩下的路径。


自定义函数
语法：
function 函数名()
{
	内容;
	return 返回值;
}
```



### shell工具

```shell
cut #剪切
语法：cut 选项 filename
选项：
-f：列号，提取第几列
-d：分隔符，按照指定分隔符分割列


sed	#一次处理一行内容
语法：sed 选项  ‘command’  filename
选项：
-e：直接在指令列模式上进行sed的动作编辑
command参数：
a 	新增，a的后面可以接字串，在下一行出现
d	删除
s	查找并替换 


awk	#文本分析工具
awk 选项 ‘pattern1{action1}  pattern2{action2}...’ filename
pattern：AWK在数据中查找的内容，匹配模式
action：找到匹配内容时执行的命令

选项
-F	指定输入文件折分隔符
-v	赋值一个用户定义变量


sort #排序
语法：sort 选项 参数
选项
-n	依照数值的大小排序
-r	以相反的顺序来排序
-t	设置排序时所用的分隔字符
-k	指定需要排序的列
```

