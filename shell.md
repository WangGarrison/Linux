# 0 Shell脚本简介

**shell脚本（需要解释器解释）**

- 命令的堆积
- 特定的语法 + 特定的命令 = 文件
- auto.sh

**Shell脚本能做什么**

- 基于标准化之上的 -> 工具化

- 作用：简化操作步骤，提高效率，减少人为干预，减少系统故障

- 1. 自动化的完成基础配置：系统初始化操作、系统更新、内核调整、网络、时区、SSH优化
  2. 定期备份恢复程序：MySQL全备+增量+binlog + crond + Shell脚本
  3. 自动化信息的收集

  4. 自动化日志收集（ELK）
  5. 自动化扩容/缩容（zabbix + shell）：监控服务器，如果发现cpu持续80%，就自动调用api多开一个服务器云主机 -> 初始化环境 -> 加入集群
  6. ......

**shell知识点**

- 特性

- 变量
  - 自定义变量
  - 系统环境变量
  - 预先定义变量
  - 位置参数变量
  - 内置变量：continue、break、exit
- 条件变量：if else
- 循环语句：for while
- 流程控制：case
- 函数：function
- 数组：array
- 正则表达式
- 实例项目

## 0.1 基本格式

```bash
#!/usr/bin/bash    指定解释器，当使用./执行的时候，可以知道用什么解释器去执行，要执行还要添加执行权限：chmod +x test.sh
echo "hello"
```

## 0.2 执行方式

使用 `./test.sh` 执行脚本需要添加执行权限`chmod +x test.sh`

使用`sh test.sh`指定解释器执行不需要添加执行权限

## 0.3 前后台作业控制

前台作业：通过终端启动，运行时一直占用终端

后台作业：通过终端启动，但启动后转到后台运行

查看当前终端运行的所有作业： ==jobs==

**前台作业运行于后台的方法**

对于尚未运行的作业，在命令后面加上 ==&==

对于已经在前台运行的作业，按 ==CTRL+Z== 键

**后台作业转到前台**

若想将后台命令转到前台，可以用fg命令： fg 后面跟上jobs中所列出的后台命令编号，且编号前要加一个%号
此命令会把前一个的ping命令转向前台：==fg %1==

若要把前台命令转移到后台，可以用==bg==命令，用法与fg相同

**nohup与screen**

后台命令虽然在后台运行，但如果关闭他所在的终端，命令仍然会停止运行，若想防止此现象发生，就要剥离后台命令与其终端的关系，需要用到nohup或者screen

screen：直接在终端输入screen，就会打开一个screen，之后再在此终端下运行的命令，即使终端被关闭，命令也会照常运行

nohup：在命令前面加一个nohup，关闭终端后命令依旧在运行

## 0.4 输入输出重定向

| 命令            | 说明                                               |
| :-------------- | :------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

> 需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。

示例：

```shell
cat < /etc/hosts   > /etc/test.sql  #前半段cat < /etc/hosts是把hosts文件内容读取出来，后半段把这些内容追加到test.sql中
```

## 0.5 管道 ｜

｜：将前者命令的标准输入交给后者命令的输入

```shell
ps -elf | grep mysql  #前者列出进程，后者从这些进程中筛选出包含mysql串的
```

## 0.6 通配符

```shell
*   匹配任意多个字符
?   匹配任意一个字符
[]  匹配括号中任意一个字符，[a-z]，[0-9]
()  括号中的命令在子shell中执行 (cd boot ; ls)
{}  集合，touch file{0..9}，会创建file0,file1,file2..flie9
\   转义
```

## 0.7 echo和printf区别

echo：可以输出颜色

printf：格式化输出文本

```shell
echo -e "\033[31m 红色字 \033[0m" 
echo -e "\033[32m 绿色字 \033[0m"
```



## 0.8 强引用""和若引用''

双引号""，强引用，会解析里面的变量

单引号'，弱引用，所见即所得，不会解析里面的变量

<img align='left' src="img/shell.img/image-20211121202453895.png" alt="image-20211121202453895" style="zoom:40%;" />

反引号``用来解析shell命令

==双引号可以解析变量，但不能解析命令，如果要把命令扩起来，得用反引号==，注意：==反引号``等价于$()==

echo要想执行命令的话，命令得用$()扩起来，或者用反引号扩起来

![image-20211121203336222](img/shell.img/image-20211121203336222.png)



# 1 变量

> 变量：用一个固定的字符串去表示一个不固定的类型

## 1.1 变量常见类型

**自定义变量**

```shell
# 定义变量
变量名=变量值
a=1
b=2

# 引用变量 $
$a
${a}  #$aa 会认为aa是一个变量名， ${a}a，会认为a是一个变量名，花括号外面的不会连带识别

# 查看变量
echo $a
set #显示所有变量，包括自定义变量和环境变量

# 取消变量
unset a

# 变量作用范围
仅在当前shell中有效
```

实例：ping通返回 ip ok，不通返回ip error：

```shell
#!/usr/bin/bash
IP=192.156.34.11
ping -c1 "$IP" &>/dev/null && echo "ip $IP ok" || echo "ip $IP error"  

#利用&&与||的短路特性：&& 第一个条件为真，会继续执行&&后面的，若也为真，则||表达式不会执行后面的；若&&第一个条件为假，则整个&&为假，直接去执行||后面的
```

**系统环境变量**

```shell
# 定义环境变量
export IP="192.168.12.11"

# 引用环境变量
$变量名 
${变量名}

# 查看环境变量
echo $变量名 env | grep xxx

# 取消环境变量
unset name

# 变量作用范围
全局

source ./ping.sh   #把ping.sh的代码加载过来
source /etc/init.d/functions  #加载系统的变量
```

**位置参数变量：脚本参数传参**

```sh
$0 脚本名
$1 第一个参数
$2 第二个参数
$* 所有的参数
$@ 所有的参数
$# 参数的个数
$$ 当前进程的PID
$! 上一个后台进程的PID
$? 上一个命令的返回值，0表示成功
```

示例：ping ip，ping通返回ok，ip手动输入

```sh
sh ping.sh 192.168.11.21  #ping.sh是$0, 192.168.11.21就是$1
```

```sh
#!/usr/bin/bash
ping -c1 $1 &> /dev/null
	if [ $? -eq 0]; then  #$?上一条命令的返回值
		echo "$1 is ok"
	else
		echo "$1 is err"
	fi
```

## 1.2 变量赋值方式

```shell
# 显示赋值
变量名=变量值

# read从键盘读取变量值
read 变量名
read -p "变量信息:" 变量名
```

## 1.3 变量数值运算

**整数运算：expr、$(()、)[]、let**

```sh
expr 1 + 2  #注意数字后面必须有空格
```

<img align='left' src="img/shell.img/image-20211121204330717.png" alt="image-20211121204330717" style="zoom:30%;" />

或者用$(())：

```sh
echo $((5-3)*2)
```

或者用中括号：

<img align='left' src="img/shell.img/image-20211121210741274.png" alt="image-20211121210741274" style="zoom:30%;" />

或者用let

```sh
let sum=2+3
echo $sum
```

**小数运算：bc**

```sh
echo "1+2" |bc
echo "scale=2;6/4" |bc  #两位小数
```

## 1.4 变量默认值

```shell
${变量名-新的变量值}
echo ${url-www.baidu.com}
```

变量没有被赋值（包括空值）：会使用新的变量值替代；变量有被赋值：不会被替代。相当于设置了一个变量的默认值

使用场景：有的选项用户不输入需要有一个默认值，可以用这个方式设置一个默认值

## 1.5 变量自增

let i++

```shell
i=1
let i++
echo $i  # 输出2
```

```sh
i=1
j=1
let x=i++  #先赋值再运算
let y=++j  #先运算再赋值
```

示例：

```sh
#!/bin/bash
ip=82.156.28.34
i=1
while [ $i -le 5 ];do  #i<=5
	ping -c1 $ip &>/dev/null  
	#&>file 意思是把 标准输出和标准错误输出 都重定向到文件file中
	if [ $? -eq 0 ];then #上一个命令的返回值，0表示成功
		echo "$ip is up..."
	fi
	let i++
done
```

# 2 shell条件表达式

## 2.1 条件表达式

```sh
格式1：test 条件表达式
格式2：[ 条件表达式 ]
格式3：[[ 条件表达式 ]]
```

==注意：shell解释器中 [ 是一个命令，使用语法和命令一样，所以后面需要加空格==

![image-20211208105749302](img/shell.img/image-20211208105749302.png)

**文件测试**

```sh
[ -e dir|file ]
[ -d dir ] 是否存在，而且是目录
[ -f file ] 是否存在，而且是文件
[ -r file ] 读权限
[ -x file ] 执行权限
[ -w file ] 写权限
```

**数值比较**

```sh
[ 整数1 操作赋 整数2 ]
[ i -gt 10 ]  大于  greater then
[ i -lt 10 ]  小于  less then
[ i -eq 10 ]  等于  equal
[ i -ne 10 ]  不等于 not equal
[ i -ge 10 ]  大于等于 greater equal
[ i -le 10 ]  小于等于 less equal
```

## 2.2 条件测试案例：磁盘使用率

需求：请查看磁盘当前使用状态，如果使用率超百分之80将对应的数值写入到指定文件中

分析：

- 查看磁盘使用率：`df -h`
- 获取对应值：`df -h | grep "/$" | awk '{print $(NF-1)}' | awk -F '%'  '{print $1}'`
- 进行数值比对：`if [ $Disk_Free -ge 80 ];then`
- 写入指定文件中：`echo "Disk Is Use:${Disk_Free}%" > /tmp/disk_use/txt`

```sh
#!/bin/sh

Disk_Free=$(df -h | grep "/$" | awk '{print $(NF-1)}' | awk -F '%'  '{print $1}')

if [ $Disk_Free -ge 80 ];then
		echo "Disk Is Use:${Disk_Free}%" > /tmp/disk_use/txt
fi	
```

## 2.3 实战练习

1、打印当前系统版本、内核、虚拟平台、主机名、内网IP、外网IP。

`hostnamectl` `ifconfig`

```sh
System=$(hostnamectl | grep "System" | awk -F ':' '{print $2}')
Kernel=$(hostnamectl | grep "Kernel" | awk -F ':' '{print $2}')
Vt=$(hostnamectl | grep "Virtualization" | awk -F ':' '{print $2}')
Static_Hostname=$(hostnamectl | grep "Static hostname" | awk -F ':' '{print $2}')
Domain_Hostname=$(hostnamectl | grep "Transient" | awk -F ':' '{print $2}')

Network_Total=$(ls /etc/sysconfig/network-scripts/grep ifcfg|wc -l)
Network_sum=$(ls /etc/sysconfig/network-scripts/grep ifcfg | awk -F '-' '{print $2}' | xargs)
Network_Eth0=$(ifconfig eht0|awk 'NR==2{print $2}')
Network_Lo=$(ifconfig lo}awk 'NR==2{print $2}')
Network_W=$(curl -s icanhazip.com)

echo "当前系统版本是：$System"
echo "当前内核版本是：$Kernel"
echo "当前虚拟平台是：$Vt"
echo "当前静态主机名是：$Static_Hostname"
echo "当前动态主机名是：$Domain_Hostname"
echo "当前总网络IP有多少：$Network_Total"
echo "当前网卡名称分别是：$Network_sum"
echo "当前eth0网卡IP地址是：$Network_Eth0"
echo "当前lo网卡IP地址是：$Network_lo"
echo "当前外网IP地址是：$Network_w"
```

2、打印当前cpu的负载情况，1分钟，5分钟，15分钟

`w`

```sh
load_1=$(w|awk 'NR==1' | awk -F ':' '{print $5}' | awk -F ',' '{print $1}')
load_5=$(w|awk 'NR==1' | awk -F ':' '{print $5}' | awk -F ',' '{print $2}')
load_15=$(w|awk 'NR==1' | awk -F ':' '{print $5}' | awk -F ',' '{print $3}')

echo "当前系统1分钟的负载时：$load_1"
echo "当前系统5分钟的负载时：$load_5"
echo "当前系统15分钟的负载时：$load_15"
```

3、打印当前磁盘的分区使用状态

`df -h`

```shell
Disk_parti=$(df -h|grep "/$"|awk '{print $1}')
Disk_Total=$(df -h|grep "/$"|awk '{print $2}')
Disk_Used=$(df -h|grep "/$"|awk '{print $3}')
Disk_Avail=$(df -h|grep "/$"|awk '{print $4}')
Disk_Used_pre=$(df -h|grep "/$"|awk '{print $5}')
Disk_Avail_pre=$(df -h | grep "/$" | awk -F '[G ]+' '{print 100-($3*100)/$2}"%"}')

echo "磁盘分区：$Disk_parti"
echo "磁盘总容量：$Disk_Total"
echo "磁盘已使用：$Disk_Used"
echo "磁盘可用：$Disk_Avail"
echo "磁盘已使用百分比：$Disk_Used_pre"
echo "磁盘可用百分比：$Disk_Avail_pre"
```

4、批量创建用户，配置固定密码

- 输入创建的用户个数
- 输入用户名前缀
- 不管是否存在该用户都创建
- 给用户配置对应的密码

```sh
#!/bin/bash
read -p "请输入创建的用户个数" user
read -p "请输入用户前缀：" pre

while true:
do
    if [[ ! $user =~ ^[0-9]+$ ]];then
        echo "请输入数字"
    else
    	break
    fi
done

for i in $(seq $user);do
	username=$pre$i
	useradd $username &>/dev/null
	echo "123"|passwd --stdin $username &>/dev/null
	if [ $? -eq 0 ];then
		echo "Creakt User ok $username"
	fi
done
```

# 3 流程控制

## 3.1 if语法

```sh
# 单分支结构
if [ $a -eq 1 ];then
	...
fi


# 双分支结构
if [ $a -eq 1 ];then
	...
else
	...
fi


#多分支结构
if [ $a -eq 1 ];then
	...
elif [ $a -eq 2 ];then
	...
elif [ $a -eq 3 ];then
	...
else
	...
fi
```

# T21





# 附录------------------------------------------

# 



## 设置别名

设置别名

```shell
alias tt="echo tt"  #这样设置以后，在shell输入tt，就相当于执行了echo tt

unalias tt  #取消tt的别名
```

## 快捷键

**1. 光标移动**

```
  ctrl + <      移动到前一个单词开头
  ctrl + >      移动到后一个单词结尾
  ctrl + A      移动到开头
  ctrl + E      移动到结尾
  shift + 4     vim命令模式下跳转到行尾
```

**2. 删除**

```
  ctrl + K      删除光标后所有字符(剪切)
  ctrl + U      删除光标前所有字符(剪切)
```

**3.撤销**

```
  ctrl + _      撤销操作
  ctrl + Y      粘贴ctrl+U/K剪切的内容
  ctrl + ?      撤消前一次输入
  alt  + R      撤消前一次动作
```

**4. 替换**

```
  ctrl + T      将光标当前字符与前面一个字符替换
```

**5. 历史命令编辑**

```
  ctrl + P      上条输入的命令(相当于上键)
  ctrl + N      上条历史命(相当于下键)
  alt  + >      上一次执行命令
  ctrl + R      输入单词搜索历史命令
```

**6. 控制命令**

```
  ctrl + L      清除屏幕
  ctrl + S      锁住终端，阻止屏幕输出
  ctrl + Q      解锁终端，允许屏幕输出
  ctrl + C      终止命令&另起一行
  ctrl + I      补全功能(类似TAB)
  ctrl + O      重复执行命令
  alt  + <数字>  操作的次数
  ctrl + Z      挂起
```

**7. !命令**

```
  !!            执行上条命令
  !-n           执行前n条命令
```

## tee命令

```shell
ping google.com | tee output.txt
# Linux tee命令用于读取标准输入的数据，并将其内容输出成文件。
```

## 附录：；&& ||区别

；

```shell
cat /etc/hosts ; mkdir /test  #分号前后的两条命令没有逻辑联系，即使前一个命令执行出错，也不会影响后面命令的执行 
```

&&

```shell
cat /etc/hosts ; mkdir /test  #&&前后的两条命令有逻辑联系，前一个命令执行出错，后面的命令就不会执行
```

||

```shell
cat /etc/hosts ; mkdir /test  #||前后的两条命令有逻辑联系，前一个命令执行出错，才会执行后者；前一个命令执行成功，则不会执行后者
```

## which和wehreis区别

which：查找某个命令的完整路径，也就是说它是用来查找可执行文件的，which命令的原理是==在当前登录用户的PATH环境变量记录的路径中进行查找==

whereis：用来快速查找任何文件，注意是任何文件，所以是一个文件搜索命令，它和另一个文件搜索命令locate的功能是一样的，与`which`不同的是这条命令==可以是通过文件索引数据库而非PATH来查找的，所以查找的面比`which`要广==

## 查看脚本的执行过程

`sh -x disk.sh`

![image-20211208113249157](img/shell.img/image-20211208113249157.png)

## w命令

w命令用于显示已经登陆系统的用户列表，并显示用户正在执行的指令。执行这个命令可得知目前登入系统的用户有那些人，以及他们正在执行的程序。

![image-20211209104817076](img/shell.img/image-20211209104817076.png)

## awk命令

AWK 是一种处理文本文件的语言，是一个强大的文本分析工具。

之所以叫 AWK 是因为其取了三位创始人 Alfred Aho，Peter Weinberger, 和 Brian Kernighan 的 Family Name 的首字符。

常用操作：

```sh
awk '{pattern + action}' {filenames}
```

```sh
awk -F: 'NR==2{print "filename: "FILENAME, $0}' /etc/passwd  #打印/etc/passwd/的第二行信息
```

```sh
w | awk "NR==1" #取出w命令的第一行

w | awk "NR==1" | awk -F ':' '{print $5}' #取出w命令的第一行以冒号为分隔符的第五个串
```

