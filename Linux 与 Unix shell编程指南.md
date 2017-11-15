- [文件权限](#----)
  * [更改权限](#----)
    + [绝对模式](#----)
    + [符号模式](#----)
  * [更改owner或Group](#--owner-group)
- [FIND](#find)
- [后台执行命令](#------)
  * [crontab](#crontab)
    + [查看进程：](#-----)
  * [nohup](#nohup)
- [文件名置换](#-----)
- [shell输入于输出](#shell-----)
  * [echo](#echo)
  * [read](#read)
  * [cat](#cat)
  * [tee](#tee)
  * [标准输入输出和错误](#---------)
- [命令执行顺序](#------)
  * [使用&&](#----)
  * [使用||](#----)
  * [使用()和{}将命令结合在一起](#---------------)
- [grep家族](#grep--)
  * [grep](#grep)
- [登录环境](#----)
- [环境和shell变量](#---shell--)
  * [本地变量](#----)
  * [环境变量](#----)
  * [位置参数变量](#------)
- [引号](#--)
  * [单双引号](#----)
  * [反引号](#---)
- [基础shell编程](#--shell--)
  * [条件测试](#----)
- [控制流结构](#-----)
  * [退出状态](#----)
  * [控制结构](#----)
- [shell函数](#shell--)
- [向脚本传递参数](#-------)
  * [shift的使用](#shift---)
  * [使用getopts](#--getopts)
- [创建屏幕输出](#------)
- [关于<<](#----)
- [shell工具](#shell--)
  * [信号](#--)
- [例子](#--)
- [常用命令](#----)

# 文件权限
## 更改权限
### 绝对模式
chmod  [mode] file
mode 为八进制数，如chmod 741 file
### 符号模式
chmod [who] operator [permission] filename
who: u -> 文件owner；g->同组用户；o->其他用户；a->所有用户
operator: +;-:=
permission: r; w; x; s->文件owner和组set-ID; t->粘性位 *; l->给文件加锁，使其他用户无法访问
chmod a-x file
chmod og+w myfile
chmod u+x o-w myfile

如果设置目录下所有文件权限：
chmod 644 *
如果递归设置目录和子目录下的文件：
chmod -R 644 *

目录的读权限意味着是否可以列出该目录的内容
目录的写权限意味这是否可以在该目录下创建文件
目录的权限位意味着是否可以进入，搜索该目录

umask用来设置用户新建文件和文件夹的默认权限，但是设置的权限是求异的，
例如执行umask 022的命令表示从默认的新建文件的权限666变为了644，新建目录的权限从777变为了755

## 更改owner或Group
格式：chown [-R] [-h] owner file
chown dyllan file
chgrp dyllan file

# FIND
命令：find pathname -options [-print -exec -ok]

options:
-name
-user
-mtime -n +n

-print: 打印到标准输出
-exec: 直接执行，对应的命令为'command' {} \;
-ok: 也是执行，只是给出提示

使用find命令可以通过文件名或权限或类型或组或文件大小或其他的信息来查找文件。
如
find . -name '*.txt' -print
find logs -types f -mtime +5 -exec rm {} \;

# 后台执行命令
## crontab
适合在特定的时间执行命令
格式crontab [-u user] -e -l -r
-u  用户名
-e 编辑crontab文件
-l 列出crontab文件中的内容
-r 删除crontab文件

crontab文件格式为：
分<>时<>日<>月<>星期<>命令
其中<>表示空格，可以用-表示从哪到哪，*表示所有，用,表示多个
举例：
30 21 * * * /apps/bin/cleanup.sh
45 4 1,10,22 * * /apps/bin/backup.sh
0 30 * * 6 /apps/bin/qtrend.sh
at
命令格式
at [-f script] [-m -l -r] [time] [date]
例如：
at 3:00pm tomorrow -f /app/bin/db_table.sh

##&
命令格式：
Command &

例如：find /etc -name "srm.conf" -print > find.dt 2> &1 &
其中2> &1表示重定位错误到之前的写的文件流

### 查看进程：
ps x | grep [pid]

## nohup
nohup command &
nohup find /etc -name "srm.conf" -print > find.dt 2> &

# 文件名置换

特殊字符：
*匹配文件名中的任何字符串
? 匹配文件名中的任何单个字符
[...] 匹配[]中所包含的任何字符
[!...] 匹配[]中非感叹号!之后的字符

# shell输入于输出
## echo
echo 后面跟-e后可以转义后面的输出，环境变量和command，例如
```bash
echo -e "you home is $HOME, tty print `tty`, well\tdone"
```
注意这里只能使用双引号
output 就是
```bash
you home is /home/dyllanluo, tty print /dev/pts/1, well	done
```
## read
可以从键盘或文件中的某一行读入信息，并赋值给某个变量。
命令格式为：read variable1 variable2
```bash
dyllanluo@dyllanluo-PC:~$ read name
Jone Doe
dyllanluo@dyllanluo-PC:~$ echo $name
Jone Doe
```
## cat
cat file
cat用来显示文件内容，因为cat会显示所有内容，可以通过管道命令，传递到另外一个具有分页功能的命令中：
cat file | more

合并文件内容可以：
cat file1 file2 file3 > bigfile

## tee
把输出的一个副本输送到标准输出，另一个副本拷贝到相应的文件中。
tee -a files
-a 表示追加到文件末尾
## 标准输入输出和错误
1. 输入文件-标准输入	0
2. 输出文件-标准输出	1
3. 错误输出文件-错误	2

示例：

1. command > filename
把标准输出重定向到一个新文件中

2. command >> filename
把标准输出重定向到一个文件中（追加）

3. command 1 > filename
把标准输出重定向到一个文件中，跟1一样

4. command > filename 2>&1
把标准输出和标准错误一起重定向到一个文件中

5. command 2 > filename
把标准输出重定向到一个文件中

6. command 2 >> filename
把标准输出重定向到一个文件中（追加）

7. command >> filename 2 > &1
标准输出和标准错误一起重定向到一个文件中（追加）

8. command < filename > filename2
Command命令以filename文件作为标准输入，以filename2文件作为标准输出

9. command < filename
command命令以filename文件作为标准输入

10. command << delimiter
从标准输入中读入，直到遇到delimiter分界符

11. command < &m
把文件描述符m作为标准输入

12. command > &m
把标准输出重定向到文件描述符m中

13. command < &-
关闭标准输入

# 命令执行顺序
## 使用&&
命令格式： command1 && command2
当command1顺利执行（返回0）后，才会执行command2
例如：
mv ./test/log.txt ./log/log.txt && rm ./test/log.txt
## 使用||
命令格式：command1 || command2
当command1顺利执行时，command2不会执行。反之，执行command2
例如：
comet month_end.txt || exit

## 使用()和{}将命令结合在一起
命令格式：
(command1;command2;command3;...)
{command1;command2;command3;...}

# grep家族
## grep
命令格式：grep [-options] expression files

grep选项：
-c	只输出匹配行的计数
-i	不区分大小写
-h	查询多个文件时不显示文件名
-l	查询多个文件时只输出包含匹配字符的文件名
-n	显示匹配行及引号
-s	不显示不存在或无匹配文本的错误信息
-v	显示不包含匹配文本的所有行

一般expression会用双引号或单引号，因为expression可以包含环境变量，这时候必须使用双引号，例如：grep "^[$name]" data.f

最后选择文件的时候，可以选多个文件，用空格隔开，也可以使用通配符

使用option P可以在将expression翻译成Perl regex，例如
```bash
grep -P '^\d{3}\s[sS]ep' -n data.f backup.f
```
# 登录环境

用户登录成功后，系统执行两个环境设置文件，第一个是/etc/profile，第二个是.profile，位于用户根目录下

# 环境和shell变量
## 本地变量
本地变量在用户现在的shell生命期的脚本中使用。如果在shell中启动另一个进程或者退出，该变量就无效。
variable-name=value
{variable-name=value}
清除变量：
unset variable-name
显示本地定义的shell变量：
set
测试变量是否已经设置：

```bash
dyllanluo@dyllanluo-PC:~$ COLOUR=blue
dyllanluo@dyllanluo-PC:~$ echo "Colour: ${COLOUR:-grey}"
Colour: blue
dyllanluo@dyllanluo-PC:~$ unset COLOUR
dyllanluo@dyllanluo-PC:~$ echo "Colour: ${COLOUR:-grey}"
Colour: grey
```
使用{variable:-value}可以显示默认值，但并没有将值默认值赋给变量执行第二个echo后COLOUR变量还是为空
```bash
dyllanluo@dyllanluo-PC:~$ unset COLOUR
dyllanluo@dyllanluo-PC:~$ echo "Colour: ${COLOUR:=grey}"
Colour: grey
dyllanluo@dyllanluo-PC:~/testshell$ echo $COLOUR
grey
```
```bash
dyllanluo@dyllanluo-PC:~/testshell$ echo "the file is ${FILE:?}"
bash: FILE: 参数为空或未设置
dyllanluo@dyllanluo-PC:~/testshell$ echo "the file is ${FILE:?" Cannot find variable file"}"
bash: FILE:  Cannot find variable file
```
设置只读变量
variable-name=value
readonly variable-name
查看所有只读变量：
readonly
## 环境变量
设置环境变量：
VARIABLE-NAME=VALUE; export VARIABLE-NAME
或
VARIABLE-NAME=VALUE
export VARIABLE-NAME
使用env可以显示所有的环境变量
清除环境变量也是使用unset
使用set -a可以将随后的赋值都导出到环境变量中：
```bash
set -a
PATH=$PATH:$HOME/bin
EDITOR=vim
```
## 位置参数变量
```bash
dyllanluo@dyllanluo-PC:~/testshell$ cat param
echo "This is the 0 parameter: $0"
echo "This is the 0 parameter: `basename $0`"
echo "This is the 1 parameter: $1"
echo "This is the 2 parameter: $2"
echo "This is the 3 parameter: $3"
echo "This is the 4 parameter: $4"
echo "This is the 5 parameter: $5"
echo "This is the 6 parameter: $6"
echo "This is the 7 parameter: $7"
echo "This is the 8 parameter: $8"
echo "This is the 9 parameter: $9"
echo "The number of arguments passed: $#"
echo "All arguments: $*"
echo "Process ID: $$"
echo "All arguments in quotes: $@"
echo "Did my script go with any errors: $?"

dyllanluo@dyllanluo-PC:~/testshell$ ./param Did you see the full moon
This is the 0 parameter: ./param
This is the 0 parameter: param
This is the 1 parameter: Did
This is the 2 parameter: you
This is the 3 parameter: see
This is the 4 parameter: the
This is the 5 parameter: full
This is the 6 parameter: moon
This is the 7 parameter: 
This is the 8 parameter: 
This is the 9 parameter: 
The number of arguments passed: 6
All arguments: Did you see the full moon
Process ID: 3587
All arguments in quotes: Did you see the full moon
Did my script go with any errors: 0
```
$?表示刚执行的脚本返回值

# 引号
## 单双引号
双引号中的特殊字符有：$, `, \

```bash
dyllanluo@dyllanluo-PC:~/testshell$ LOL=12
dyllanluo@dyllanluo-PC:~/testshell$ echo "$LOL"
12
dyllanluo@dyllanluo-PC:~/testshell$ echo '$LOL'
$LOL
```
## 反引号
反引号中的内容会被当作bash执行
```bash
dyllanluo@dyllanluo-PC:~/testshell$ echo `date`
2017年 11月 01日 星期三 20:42:46 CST
```
# 基础shell编程
## 条件测试
文件测试状态
-d	目录
-f	正规文件
-L	符号连接
-r	可读
-s	文件长度大于0,非空
-w	可写
-u	文件有suid位设置
-x	可执行

测试文件状态有两种格式：
test condition
或
[ condition ]
```bash
dyllanluo@dyllanluo-PC:~/testshell$ ls -l param
-rwxrw-r-- 1 dyllanluo dyllanluo 565 11月  1 20:25 param
dyllanluo@dyllanluo-PC:~/testshell$ test -d param
dyllanluo@dyllanluo-PC:~/testshell$ echo $?
1
dyllanluo@dyllanluo-PC:~/testshell$ [-w param]
bash: [-w: 未找到命令
dyllanluo@dyllanluo-PC:~/testshell$ [ -w param ]
dyllanluo@dyllanluo-PC:~/testshell$ echo $?
0
```
测试时使用逻辑操作符
-a	逻辑与
-o	逻辑或
表示非可以使用`!`
```bash
dyllanluo@dyllanluo-PC:~/testshell$ [ -w param -a -w quote.txt ]
dyllanluo@dyllanluo-PC:~/testshell$ echo $?
0
```
字符串测试
有五种格式：
test "string"
test string_operator "string"
test "string" string_operator "string"
[ string_operator string ]
[ string string_operator string ]
string_operator可为：
=	两个字符串相等
!=	两个字符串不等
-z	空字符串
-n	非空字符串

数值测试
格式为：
"number" numeric_operator "number"
或者
[ "number" numeric_operator "number" ]
numeric_operator:
-eq	等于	equal
-ne	不等	not equal
-gt	大于	greater than
-lt	小于	less than
-le	小于或等于	less or equal
-ge	大于或等于	greater or equal

expr用法
expr argument operator argument
```bash
dyllanluo@dyllanluo-PC:~$ expr 10+10
10+10
dyllanluo@dyllanluo-PC:~$ expr 10 + 10
20
dyllanluo@dyllanluo-PC:~$ expr 30 / 3
10
dyllanluo@dyllanluo-PC:~$ expr 30 / 3 * 2
expr: 语法错误
dyllanluo@dyllanluo-PC:~$ expr 30 / 3 \* 2
2
dyllanluo@dyllanluo-PC:~$ expr 30 / 3 / 2
5
dyllanluo@dyllanluo-PC:~$ value=12
dyllanluo@dyllanluo-PC:~$ expr $value + 0 > /dev/null 2>&1
dyllanluo@dyllanluo-PC:~$ echo $?
0
dyllanluo@dyllanluo-PC:~$ value=hello
dyllanluo@dyllanluo-PC:~$ expr $value + 0 > /dev/null 2>&1
dyllanluo@dyllanluo-PC:~$ echo $?
2
```
# 控制流结构
## 退出状态
```bash
dyllanluo@dyllanluo-PC:~/testshell$ cat exitcode
echo "this is test"
exit $1
dyllanluo@dyllanluo-PC:~/testshell$ ./exitcode 0
this is test
dyllanluo@dyllanluo-PC:~/testshell$ echo $?
0
dyllanluo@dyllanluo-PC:~/testshell$ ./exitcode 1
this is test
dyllanluo@dyllanluo-PC:~/testshell$ echo $?
1
dyllanluo@dyllanluo-PC:~/testshell$ ./exitcode 3
this is test
dyllanluo@dyllanluo-PC:~/testshell$ echo $?
3
```
## 控制结构
IF:
```bash
if condition1
	then command1
elif condition2
	then command2
else command3
fi
```
CASE:
```bash
case VALUE in
	mode1)
    	command1
        ...
        ;;
    mode2)
    	command2
        ...
        ;;
    *)
    	default command
esac
```
模式中可以使用|制定多种模式：
```bash
dyllanluo@dyllanluo-PC:~/testshell$ cat caseselect 
echo "(y/n) "
read ANS
case $ANS in
	y|Y|YES|yes) echo "yes"
	;;
	*) echo "no"
	;;
esac
dyllanluo@dyllanluo-PC:~/testshell$ ./caseselect 
(y/n) 
y
yes
dyllanluo@dyllanluo-PC:~/testshell$ ./caseselect 
(y/n) 
yes
yes
dyllanluo@dyllanluo-PC:~/testshell$ ./caseselect 
(y/n) 
fd
no
```
FOR:
```bash
for variable in list
do
	command1
    command2
done
```
UTIL:
```bash
util condition
	command1
    ...
done
```
WHILE:
```bash
while command
do
	command1
    command2
    ...
done
```
```bash
dyllanluo@dyllanluo-PC:~/testshell$ cat data.f
48  DEC 3BC1997 LPSX    68.00   LVX2A   138
483 Sep 5Ap1996 USP     65.00   LVX2C   189
47  OCT 3ZL1998 LPSX    43.00   KVM9D   512
219 dec 2CC1999 CAD     23.00   PLV2C   68
484 nov 7PL1996 CAD     49.00   PLV2C   234
484 may 5PA1998 USP     37.00   KVM9D   644
216 sep 3ZL1998 USP     86.00   KVM9E   234
dyllanluo@dyllanluo-PC:~/testshell$ cat whileread 
while read LINE
do
	echo $LINE
done < data.f

dyllanluo@dyllanluo-PC:~/testshell$ ./whileread 
48 DEC 3BC1997 LPSX 68.00 LVX2A 138
483 Sep 5Ap1996 USP 65.00 LVX2C 189
47 OCT 3ZL1998 LPSX 43.00 KVM9D 512
219 dec 2CC1999 CAD 23.00 PLV2C 68
484 nov 7PL1996 CAD 49.00 PLV2C 234
484 may 5PA1998 USP 37.00 KVM9D 644
216 sep 3ZL1998 USP 86.00 KVM9E 234
```
break和continue都支持
# shell函数
函数定义：
```bash
[function] functionName () {
	command1
    command2
    ...
}
```
函数可以通过关键字return提前返回，后面传一个数值可以表示函数执行情况：
return		# depend on the command that executed in the function
return 0	# no error
return 1	# failed
函数返回值测试有两种：
```bash
check_file $FILENAME
if [ $? = 0 ]
then
	echo "success"
else
	echo "failed"
fi
if check_file $FILENAME; then
	echo "success"
else
	echo "failed"
fi
```
也可以把函数的返回值保存到变量中
variable_name=function_name
将函数写入到文件中，可以将该文件载入shell中直接被调用。
载入函数文件到shell中的方法：
```bash
. /pathname/filename
```
此处为<点><空格><斜线><文件名> (定位文件不仅限于定位function文件，也可以配置全局变量，在我看来有export的功能)
使用set命令确认函数是否被载入到shell中
删除函数：unset function_name
# 向脚本传递参数
## shift的使用
```bash
dyllanluo@dyllanluo-PC:~/testshell$ cat opt
loop=0
while [ $# -ne 0 ]
do
	echo $1
	shift
done
dyllanluo@dyllanluo-PC:~/testshell$ ./opt file1 file2 file3
file1
file2
file3
```
## 使用getopts
```bash
dyllanluo@dyllanluo-PC:~/testshell$ cat getopt1 
while getopts :ahfg:vc: OPTION
do
	case $OPTION in
	a)echo "SET ALL"
		;;
	h)echo "SET HELP"
		;;
	v)echo "SET VERBOSE"
		;;
	c)echo "SET COPIES is $OPTARG"
		;;
	g)echo "SET GO is $OPTARG"
		;;
	\?)echo "`basename $0` -[a h f v] -[c value] file" >&2
		;;
	esac
done
dyllanluo@dyllanluo-PC:~/testshell$ ./getopt1 -ah -c 3 -g file
SET ALL
SET HELP
SET COPIES is 3
SET GO is file
```
# 创建屏幕输出
常用字符串
blink		闪烁模式
bold		粗体
civis		隐藏光标
clear		清屏
cnorm		不隐藏光标
cup			移动光标到屏幕位置
el			清除到行尾
ell			清除到行首
smso		启动突出模式
rmso		停止突出模式
smul		开始下划线模式
rmul		结束下划线模式
sc			保存当前光标位置
rc			恢复光标到最后保存位置
sgr0		正常屏幕
rev			逆转视图
数字输出
cols		列数目
it			tab设置宽度
lines		屏幕行数
布尔输出
chts		光标不可见
hs			具有状态行


![](https://raw.githubusercontent.com/DyllanReview/BookReview/master/image/LinuxANDUnixShellProgrammingShell/tput2.png)

# 关于<<
当shell看到<<的时候， 它就知道下一个词是一个分解符。在该分解符以后的内容都被当作输入，直到shell又看到该分解符。这个分解符可以是你所定义的任何字符串。

# shell工具

date命令格式：
date option + %format
例如：date +%Y/%M/%d

## 信号
发送信号可以使用如下格式：
kill [-signal no:| signal name] processID


| signal no | signal name | meaning |
| 1 | SIGHUP | 挂起或父进程被杀死 |
| 2 | SIGINT | 来自键盘的中断信号，通常是<CTRL-C> |
| 3 | SIGQUIT | 从键盘退出 |
| 9 | SIGKILL | 无条件终止 |
| 11 | SIGSEGV | 段冲突 |
| 15 | SIGTERM | 软件终止（缺省杀进程信号） |
在脚本中可以使用trap捕捉信号，格式为：
trap name signal(s)
其中name是捕捉到信号以后所采取的一系列操作。实际生活中，name一般是一个专门用来处理捕捉信号的函数。name需要用双引号（或单引号）引起来。

例如
trap "" 2 3 			#忽略信号2和3,用户本能终止该脚本
trap "commands" 2 3		#执行commands如果捕捉到信号2或3
trap 2 3				#复位信号2和3,用户可以终止该脚本

eval命令
eval “string” 意思是把string当命令执行

logger命令
logger命令格式：
logger -p -I message
-p为优先级
-I后面为发送消息的进程号

# 例子
pingall:
```bash
cat /etc/hosts| grep -P '^[^#]' | while read LINE
do
	ADDR=`awk '{print $1}'`
	for MACHINE in $ADDR
	do
		ping -s -cl $MACHINE
	done
done
```

# 常用命令
basename
cat
compress options files
压缩或解压文件
cp options file1 file2
diff options file1 file2
dircmp
du options directory
-a: 显示每个文件的大小，不仅是整个目录所占用的空间
-s: 只显示总计
file filename
fuser options file
-k: 杀死所有访问该文件或文件系统的进程
-u: 显示访问该文件或文件系统的所有进程
head -number files
logname
mkdir
more
nl -options file
printf format arguments
pwd
rm options files
rmdir options directory
script option file
shutdown
sleep
strings filename
touch
tty
uname
wait
uncompress
wait
wc options files
whereis command_name
who