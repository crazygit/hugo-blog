---
title: Linux命令行与shell脚本编程大全第二版学习笔记
slug: linux-commands-and-bash
date: 2013-01-14T09:48:29+08:00
description: Linux命令行与shell脚本编程大全第二版学习笔记
categories:
  - 学习笔记
tags:
  - linux
  - bash
---

## Linux系统可以划分为4个部分

* Linux内核
* GUN工具组件
* 图形化桌面环境
* 应用软件

## 常用命令

### ls

* -F 在目录名后面添加/ ,方便和普通文件区分
* -R 递归显示当前目录
* -A 效果与-a一样，只是不输出(.)和（..)


### touch

* -t 修改已有文件的时间戳

### cp

* -b 复制文件时，如果目的地有一份同名文件，就将该文件备份然后再复制，而不是直接覆盖
* -p 保留拷贝文件的文件属性，如修改时间等
* -l 与 -s 参数可以使cp命令与ln命令一样，创建链接-l（硬链接),-s(软链接）

### rmdir

* 删除一个空目录

<!--more-->

### stat

* 查看一个文件的详细信息

```bash
$ stat a
  File: 'a'
    Size: 5             Blocks: 8          IO Block: 4096   regular file
    Device: 807h/2055dInode: 1048658     Links: 1
    Access: (0644/-rw-r--r--)  Uid: ( 1000/linliang)   Gid: ( 1000/linliang)
    Access: 2013-01-14 10:55:27.643013669 +0800
    Modify: 2013-01-14 10:55:18.443014365 +0800
    Change: 2013-01-14 10:55:23.733014890 +0800
```


### file

* 查看文件类型


### cat

* -n 显示行号
* -b 指给有文本的行加上行号，空白行不会显示行号
* -s 将多个空白行压缩到单个空白行
* -T 不显示制表符


### ps

Linux系统中使用的`ps`命令支持3种不同类型的命令行参数

* Unix风格的参数，前面加单破折线
* BSD风格的参数， 前面不加破折线
* GNU风格的长参数，前面加双破折线

### sort

* -t 指定分隔符号
* -k 指定排序的位置,初始值是1

如按user id排序

```bash
sort -t ':' -k 3 -n /etc/passwd
```


### bzip2

* 压缩文件（不是目录），多用于压缩大型的二进制文件
`bzip2 filename`

### bzcat

* 用来显示上面命令压缩文本的内容

### bunzip2

* 用来解压压缩文件

同样，`gzip ,gzcat, gunzip`命令与上面用法一样


### printenv

* 打印当前系统的全局变量

### set

* 显示为某个特定进行设置的所有环境变量（包括全局变量和局部变量）

### uset

* 删除一个环境变量, 但是如果在一个子进程中删除了一个全局变量，它只对子进程有效，该变量在父进程中依然有效

```bash
$ a=test
$ export a
$ echo $a

$ bash
$ echo $a

$ unset a
$ echo $a

$ exit

$ echo $a

```

### alias

* -p 查看当前设置了哪些别名


### usermod

修改账户信息

* -l 修改用户的登录名
* -L 锁定账户，让账户无法登录
* -U 解除锁定，让账户可以登录

### passwd

修改用户密码

* -e 选项能强制用户下次登录时修改密码

### chsh

* 快速修改默认的用户登录shell

### groupadd

* 添加一个组，但是没有将用户添加到组的选项

### usermod

将一个用户添加到某一个组

    usermod -G groupname username

### groupmod

* 修改组信息

### umask

用来设置用户创建文件和目录的默认权限
umask的值只是一个掩码，它会屏蔽掉不想授予该安全级别的权限。
umask值会从对象的全权值中减掉，对文件来说，全权值的666（所有用户都有读写权限)
对于目录来说，全权值是777（所有用户都有读写和执行的权限)
umask的代表了一项特别的安全特性，叫做粘着位.
umask 0022
那么创建的文件权限就是(666-022=644)

umask的值通常保存在/etc/profile文件中

### bash进行数学运算

使用expr命令

```bash
$ expr 1 + 5
$ expr 2 \* 5
```

使用方括号

```bash
$ echo $[1+5]
$ echo $[2*5]
```

上面两种方法在做除法的时候都只支持整数运算

```bash
$ echo $[6/4]
$ 1
```
为了解决这种问题，可以使用bash内建的计算器bc

    $ bc -q （-q不打印欢迎信息）

通过内建变量scale可以控制想得到小数点的位数, scale的默认值是0

```bash
$ bc -q
6/4
1
scale=2
6/4
1.50
```

在脚本中使用bc的方法：
当表达式比较短时:

```bash
echo "scale=1;6/4" |bc
```

表达式比较长的时候，可以使用内联的输入重定向

```bash
$ cat a.sh
#!/bin/bash
a=`bc <<EOF
b=3+2
scale=1
c=6/4
b+c
EOF
`
echo $a

$ bash a.sh
6.5
```

在使用`-n`或`-z`判断一个字符串是否为空时，记得要给字符串加引号

```bash

$ cat a.sh
#!/bin/bash
a=""
# 字符串不加引号
if [ -n $a ];then
    echo "not empty"
else
    echo "null"
fi
$ bash -x a.sh
+ a=
+ '[' -n ']'
+ echo 'not empty'
not empty


$ cat a.sh
#!/bin/bash
a=""

# 字符串加了引号
if [ -n "$a" ];then
    echo "not empty"
else
    echo "null"
fi
$ bash -x a.sh
+ a=
+ '[' -n '' ']'
+ echo null
null
```

没有加引号时, 相当于执行了如下命令，所以结果一直是错误的

```bash
$ test -n
$ echo $?
0
```

if 语句可以接双尖括号，双方括号
双尖括号命令允许你将高级的数学表达式放入比较之中
双方括号命令提供了针对字符串比较高的特性，如模式匹配

```bash
$ cat a.sh
#!/bin/bash
a="take"

if [[ $a == t* ]];then
    echo $a
    fi
$ bash a.sh
take
```

### `IFS`内部字段分割符(internal filed separator)

IFS环境变量定义了bash shell中用作字段分隔符的一些列字符。默认情况下, bash shell
会将下列字符当做分隔符

* 空格
* 制表符
* 换行符

如果bash shell在数据中看到了这些字符中的任意一个，它会假定你在列表中开始了一个新的数据段

```bash
$ cal > a
$ for i in `cat a`;do echo $i;done
January
2013
Su
Mo
Tu
We
Th
Fr
Sa
......
现在指定以换行符作为分隔符,注意'\n'前面有个‘$’符号
$ IFS=$'\n';for i in `cat a`;do echo $i;done
    January 2013
Su Mo Tu We Th Fr Sa
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30 31

```

通常，在一个地方修改了换行符之后，最好是在使用完之后将它改回去

```bash
IFS_OLD=$IFS
IFS=$'\n'
<use the new IFS value in code>
IFS=$IFS_OLD
```

如果要指定多个IFS字符，只要将他们在赋值行连起来就可以

```bash
IFS=$'\n:;"'
```
使用换行符，冒号，分号和双引号作为分隔符号

### break

```bash
break n
```

其中`n`说明了要跳出的循环级别。默认情况下，`n`为1,表示当前的循环级别。如果你将n设置为`2`, break命令将会停止下一级别的外部命令

```bash
$ cat a.sh
#!/bin/bash
for (( a=1; a<4; a++))
do
    echo "Outer loop: $a"
    for (( b=1; b<100; b++ ))
    do
        if [ $b -gt 4 ];then
            break 2
        fi
        echo "Inner loop: $b"
    done
done

$ bash a.sh
Outer loop: 1
Inner loop: 1
Inner loop: 2
Inner loop: 3
Inner loop: 4

```

和`break n`一样，`continue`也有类似效果

```bash
$ cat /tmp/a.sh
#!/bin/bash
for (( a=1; a<=5; a++))
do
    echo "Outer loop: $a"
    for (( b=1; b<3; b++ ))
    do
        if [ $a -gt 2 ] && [ $a -lt 5 ];then
            continue 2
        fi
        echo -e "\tInner loop: $b"
    done
done
$ bash /tmp/a.sh
Outer loop: 1
    Inner loop: 1
    Inner loop: 2
Outer loop: 2
    Inner loop: 1
    Inner loop: 2
Outer loop: 3
Outer loop: 4
Outer loop: 5
    Inner loop: 1
    Inner loop: 2

```

### bash参数

bash 中`$0`参数可以获取shell在命令行启动的程序名字，但是当传给`$0`变量的真实字符串是整个脚本的路径时
程序中就会适用整个路径，而不仅仅是脚本名

```bash
$ cat a.sh
#!/bin/bash
echo $0

$ ./a.sh
./a.sh

$ /tmp/a.sh
/tmp/a.sh
```

* `$#`参数个数
* `${!#}`最后一个参数。注意不是`${$#}`, 使用`${$#}`会产生错误结果,它表明不能在花括号内使用美元符号，你必须将美元符号换成感叹符号

```bash
$ cat a.sh
#!/bin/bash
echo $#
echo ${$#}

$ bash a.sh a b c
3
23070

$ cat a.sh
#!/bin/bash
echo $#
echo ${!#}

$ bash a.sh a b c
3
c

```

`$*`和`$@`变量都提供了对所有参数的快速访问
区别是：
`$*`变量会将命令行上的提供的所有参数都当做是单个单词保存，每个词是指命令行上出现
的每个值，基本上，`$*`会将这些都当做一个参数，而不是多个对象

`$@`会将命令行上提供的所有参数都当做同一个字符串中的多个独立单词。它允许你遍历所有的值

```bash
$ cat a.sh
#!/bin/bash
for par in $@
do
    echo $par
    done

for par in $*
do
    echo $par
done

$ bash a.sh a b c
a
b
c
a
b
c

$ cat a.sh
#!/bin/bash
for par in "$@"
do
    echo $par
done

for par in "$*"
do
    echo $par
done
$ bash a.sh

$ bash a.sh a b c
a
b
c
a b c
```

### `shift`命令

移动参数位置，默认情况下将每个参数变量减1（`$0`的值不会改变，被移出的参数会被删除，且无法恢复）

`shift n`

一次移动n个参数

```bash
$ cat a.sh
#!/bin/bash

while [ -n "$1" ]
do
    echo $1
    shift
done
$ bash a.sh a b c
a
b
c
```

### getopt

bash 中，选项是根在单破折线后面的单个字母
双破折线表示选项结束，剩下的都是参数，而不是选项
使用`getopt`命令解析参数的时候,格式如下:
每个需要参数值的选项字母后面加个冒号

```bash
# getopt options optstring parameters
$ getopt ab:cd -a -b bvalue -cd var1 var2
 -a -b bvalue -c -d -- var1 var2
```

可以看出它会自动在参数前面加上双破折号(`--`)

使用了不存在的选项时会提示

```bash
$ getopt ab:cd -a -b bvalue -cde var1 var2
getopt: invalid option -- 'e'
 -a -b bvalue -c -d -- var1 var2
```

加上`-q`可以忽略错误信息

```bash
$ getopt -q ab:cd -a -b bvalue -cde var1 var2
 -a -b 'bvalue' -c -d -- 'var1' 'var2'
```


在脚本中使用`getopt`, 可以使用`set --` 命令：
`set --` 命令的作用就是将命令行参数替换成`set`命令的命令行的值

即`set --` 命令可以修改传递给一个脚本的参数值

```bash
$ cat b.sh
#!/bin/bash -
echo $@
set -- '-a -b -c'
echo $@
$ bash b.sh -a
-a
-a -b -c
```
上面可以看出虽然调用脚本时只传递了`-a`参数，但是通过`set --`命令之后，相当于执行的是
`bash  b.sh -a -b -c`

```bash
$ cat /tmp/a.sh
#!/bin/bash
echo $@
set -- `getopt -q ab:c "$@"`
echo $@
while [ -n "$1" ]
do
    case "$1" in
    -a) echo "Found the -a option";;
    -b) param=$2
        echo "Found the -b option, with value $param"
        shift ;;
    -c) echo "Found the -c option";;
    --) shift
        break ;;
    *)  echo "$1 is not an option";;
    esac
    shift
done

count=1
for param in "$@"
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done

$ bash /tmp/a.sh -a -b jk -c -k value1 value2
-a -b jk -c -k value1 value2
-a -b 'jk' -c -- 'value1' 'value2'
Found the -a option
Found the -b option, with value 'jk'
Found the -c option
Parameter #1: 'value1'
Parameter #2: 'value2'
```

### getopts

```
getopts optstring variable
```

optstring类似于`getopt`命令中的那个，有效的字母都会列在optstring中，如果选项字母要求
有个参数值，就加一个冒号。要去掉错误信息的话，可以在optstring之前加一个冒号。
optstring命令将当前所有的参数保存在命令行中定义的variable中

`getopts`命令会用到两个环境变量。
如果选项需要跟一个参数值，OPTARG环境变量就会保存这个值
OPTIND环境变量保存了参数列表中`getopts`正在处理的参数位置。这样你就能在处理完选项之后
继续处理其他命令行参数了


`getopts`命令解析命令选项时，它会移出开头的单破折线，所以在case语句中不用再加单破折线
`getopts`能将命令行上找到的所有未定义的选项同一输出成问号。


```bash
$ cat c.sh
#!/bin/bash
while getopts :ab:c opt
do
    case "$opt" in
    a) echo "Found the -a option";;
    b) echo "Found the -b option, with value $OPTARG";;
    c) echo "Found the -c option";;
    *) echo "Unkown option $opt";;
    esac
done

# 未知选项-f时显示的是?,而不是-f
$ bash c.sh -f
Unkown option ?
```


`getopts`命令知道何时停止处理选项，并将参数留给你处理。在`getopts`处理每个选项时，它会将
`OPTIND`环境变量值增一。 在`getopts`处理完成时，你可以将`OPTIND`值和`shift`命令一起使用来移动参数。

一个能在所有的shell脚本中使用的全攻能命令行选项和参数处理

```bash
$ cat c.sh
#!/bin/bash
while getopts :ab:cd opt
do
    case "$opt" in
    a) echo "Found the -a option";;
    b) echo "Found the -b option, with value $OPTARG";;
    c) echo "Found the -c option";;
    d) echo "Found the -d option";;
    *)  echo "Unkown option $opt";;
    esac
done

shift $[ $OPTIND -1 ]
count=1
for param in "$@"
do
    echo "Parameter $count: $param"
    count=$[ $count + 1 ]
done

# "value3 value4" 为一个参数，只是包含空格
$ bash c.sh -a -b bvalue -c value1 value2 "value3 value4"
Found the -a option
Found the -b option, with value bvalue
Found the -c option
Parameter 1: value1
Parameter 2: value2
Parameter 3: value3 value4
```

### read命令

```bash
linliang@sky:/tmp$ cat a.sh
#!/bin/bash
echo -n "Enter your name: "
read name
echo "Hello $name"
```
`-p`选项可以直接在read命令行指定提示符

```bash
$ cat a.sh
#!/bin/bash
read -p "Enter your name: " name
echo "Hello $name"
```

read 命令后面也可以保存多个变量。输入的每个参数值都会分配给表中的下一个变量。如果变量表在数据之前就用完了，
剩下的数据就会分配给最后一个变量

```bash
$ cat a.sh
#!/bin/bash
read -p "Enter your name: " firstname lastname
echo "Hello $lastname.$firstname"

$ bash a.sh
Enter your name: Jack Cheng
Hello Cheng.Jack
```

如果在`read`命令行中不指定变量。read命令会将它收到的任何数据都放进特殊的环境变量`REPLY`中

`-t`选型可以设定一个计数器，指定等待的秒数。当计时器过期后，会返回一个非零退出状态码

```bash
$ cat a.sh
#!/bin/bash
if read -t 3 -p "Enter your name: " firstname lastname;then
    echo "Hello $lastname.$firstname"
else
    echo "time out"
fi

$ bash a.sh
Enter your name: time out
```

`read`命令还可以对输入的字符计数，当输入的字符达到预设的字符数时，它会自动退出，将输入的数据赋给变量

```bash
$ cat /tmp/a.sh
#!/bin/bash
read -n1 -p "Do you want to continue[Y/N]?" answer
case $answer in
    Y|y)
        echo
        echo "fine, continue on ...";;
    N|n)
        echo
        echo "OK, goodbye";;
    *)
        echo
        echo "unkown answer";;
esac
```
`-n1`表示`read`命令接受单个字符

`read -s`选型可以阻止传给read命令的数据显示在显示器上(实际上，数据被完全显示，只是read命令会将文本颜色设置成跟背景颜色一模一样

```bash
read -s -p "Enter your password" pass
```

从文件读取

```bash
$ cat /tmp/a.sh
#!/bin/bash
count=1
cat /etc/passwd | while read line
do
    echo "Line $count: $line"
    count=$[ $count + 1 ]
done
```
`while`循环会通过`read`命令处理文件中的行，知道`read`命令以非零退出状态码退出


### 错误重定向

`&> filename`

将`STDERR`和`STDOUT`输出重定向到一个文件，即: `2>&1 >filename`


文件描述符
linux用文件描述符来标示每个文件对象，文件描述符是一个非负整数，可以唯一地标识会话中打开的文件
。每个过程一次最多可以有9个文件描述符。

* 临时重定向

```bash
$ cat a.sh
#!/bin/bash
echo "This is an error" >&2 #自己将错误重定向到错误输入
echo "This is an normal output"

#由于默认情况下，linux将STDERR定向到STDOUT,所以上面的脚本执行效果如下
$ bash a.sh
This is an error
This is an normal output

# 改变错误重定向的输出
$ bash a.sh 2>a.txt
This is an normal output

$ cat a.txt
This is an error
```


* 永久重定向

使用`exec`命令会启动一个新的shell并将STDOUT文件描述符重定向到文件。脚本中发给STDOUT的所有输出都会被重定向到文件

```bash
$ cat a.sh
#!/bin/bash
# 将改脚本的所有输出重定向到a.txt
exec &>a.txt
echo "This is an error" >&2
echo "This is an normal output"
```

除了0, 1, 2之外，我们还可以使用3~8其他6中文件描述符，使用方法一样

```bash
$ cat a.sh
#!/bin/bash
# 将改脚本的所有输出重定向到a.txt
exec 3>a.txt
echo "This is an error" >&3
echo "This is an normal output"

恢复文件重定向描述符
# 原理就是设置一个中间变量来保存重定向前的状态
$ cat a.sh
#!/bin/bash
exec 3>&1
exec 1>a.txt
echo "This is a normal output to a.txt"

exec 1>&3
echo "This is a normal output on screen"
$ bash a.sh
This is a normal output on screen
$ cat a.txt
This is a normal output to a.txt
```

### trap

* Ctrl-C终止进程
* Ctrl-Z停止进程

捕捉信号:
在脚本中使用`trap`可以捕捉信号
格式如下：

```
trap commands signals
```

commands 捕捉到信号后要执行的命令

signals 想要捕捉的信号，用数值或linux信号名来指定信号，多个用空格分开

```
信号 值       描述
1    SIGUP    挂起进程
2    SIINT    终止进程
3    SIGQUIT  停止进程
9    SIGKILL  无条件终止进程
15   SIGTERM  可能的话终止进程
17   SIGSTOP  无条件停止进程,但不是终止进程
18   SIGSTP   停止或暂停进程,但不终止进程
19   SIGCONT  继续运行停止的进程
```

```bash
$ cat a.sh
#!/bin/bash
trap "echo 'I have trapped Ctrl-C'" SIGINT SIGTERM
count=1
$while [ $count -le 10 ]
do
    echo "loop #$count"
    sleep 5
    $count=$[ $count + 1 ]
done
echo "end"


$ bash a.sh
loop #1
loop #2
^CI have trapped Ctrl-C
loop #3
^CI have trapped Ctrl-C
loop #4
loop #5
loop #6
loop #7
loop #8
loop #9
loop #10
^CI have trapped Ctrl-C
end
```
每次使用Ctrl-C组合键，脚本都会执行trap命令中指定的echo语句，而不是忽略此信号并允许脚本停止

捕捉脚本的退出，只需要在trap命令后加上EXIT信号就行

```bash
$ cat a.sh
#!/bin/bash
trap "echo goodbye" EXIT
count=1
while [ $count -le 5 ]
do
    echo "loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
echo "end"

$ bash a.sh
loop #1
loop #2
loop #3
loop #4
loop #5
end
goodbye

$ bash a.sh
loop #1
loop #2
^Cgoodbye
```

### nice命令

nice命令允许你在启动时调整一个命令的调度优先级

renice命令，改变已经运行命令的优先级

### at命令

计划执行作业

```
at -f filename time
```

`crontab`中的命令如果在系统关闭又重启的情况下，不会执行那些在指定在系统关闭时间内执行的作业。
anacron可以实现这一点


在新shell中启动
当新shell是登录生成的话，它每次会运行`.bash_profile`或`.profile`文件

```bash
$ head -n1 .profile
# ~/.profile: executed by the command interpreter for login shells.
当新shell,包括有新的登录情况时，bash shell都会运行.bashrc文件

$ head -n1 .bashrc
# ~/.bashrc: executed by bash(1) for non-login shells.
```

### bash 函数

默认情况下，函数的退出状态码是最后一条命令返回的退出状态码。在函数执行结束后，你可以用`$?`变量来决定函数的退出状态

```bash
$ cat a.sh
#!/bin/bash

foo()
{
    echo "Hello world"
    ls abc
}

foo
$ bash a.sh
Hello world
ls: cannot access abc: No such file or directory
$ echo $?
2

# 从下面的例子可以看出，ls命令没有成功执行，但是最后一条echo命令执行成功，因此返回值仍然是0

$ cat a.sh
#!/bin/bash

foo()
{
    echo "Hello world"
    ls abc
    echo "ok"
}

foo
$ bash a.sh
Hello world
ls: cannot access abc: No such file or directory
ok
$ echo $?
0
```

因此使用函数的默认返回值来决定一个函数是否执行功能是不可取的
解决方法是使用return来返回特定的退出状态
使用return时需要注意一下两个方面：
1.函数一执行完就取返回值，不然执行了其他命令之后，`$?`的值就改变了
2.return的状态码必须在0~255之间

令一个取得返回值的方法就是使用echo语句

```bash
$ cat a.sh
#!/bin/bash

foo()
{
    echo "Hello world"
    ls abc
    echo "ok"
}

result=$(foo)
echo $result
$ bash a.sh
ls: cannot access abc: No such file or directory
Hello world ok
```

使用echo语句取得函数返回值需要注意，它取得的返回值并不只是最后一个echo语句的内容

```bash
$ cat a.sh
#!/bin/bash

foo()
{
    echo "Hello world"
    echo "ok"
}

result=$(foo)
echo $result
$ bash a.sh
Hello world ok
```

默认情况下，bash脚本中定义的任何变量都是全局变量，
想在函数内声明一个局部变量，在前面加上local即可,local关键字保证了变量只局限在该函数中，如果脚本中在函数外有同样名字的变量
那么shell脚本将会保持这两个变量的值是分离的。

```bash
$ cat a.sh
#!/bin/bash
a="hello"
echo $a
foo()
{
    local a="world"
    echo $a
}
foo
echo $a

$ bash a.sh
hello
world
hello
```


当向一个函数传递数组的时候，函数只会接受数组的第一个值。

```bash
$ cat a.sh
#!/bin/bash
function foo()
{
    echo "The parameters are: $@"
    thisarray=$1
    echo "The receive is ${thisarray[*]}"
}

myarray=(1 2 3 4 5)
echo "The original array is ${myarray[*]}"
foo $myarray
$ bash a.sh
The original array is 1 2 3 4 5
The parameters are: 1
The receive is 1
```

因此必须要将数组变量的值分解成单个值然后将这些值作为参数使用。在函数内部，再将所有的参数重组到新的变量

```bash
$ cat a.sh
#!/bin/bash
function foo()
{
    echo "The parameters are: $@"
    thisarray=`echo $@`
    echo "The new array is ${thisarray[*]}"
}

myarray=(1 2 3 4 5)
echo "The original array is ${myarray[*]}"
foo ${myarray[*]}
$ bash a.sh
The original array is 1 2 3 4 5
The parameters are: 1 2 3 4 5
The new array is 1 2 3 4 5
```


### shell 脚本菜单

```bash
$ cat a.sh
#!/bin/bash
function diskspace()
{
    clear
    df -k
}

function whoseon()
{
    clear
    who
}

function memusage()
{
    clear
    cat /proc/meminfo
}

function menu()
{
    clear
    echo
    echo -e "\t\t\tSys Admin Menu\n"
    echo -e "\t1.Display disk space"
    echo -e "\t2.Display logged on users"
    echo -e "\t3.Display memory usage"
    echo -e "\t0.Exit Progame"
    echo -en "\tEnter option:"
    read -n1 option
}

while [ 1 ]
do
    menu
    case $option in
        0)
            break;;
        1)
            diskspace;;
        2)
            whoseon;;
        3)
            memusage;;
        *)
            clear
            echo "Sorry, Wrong selection"
    esac
    echo -en "\n\n\t\tHit any key to continue."
    #让显示的结果暂停，不然就继续调用menu函数，看不到选择的结果
    read -n1 line
done
clear
```

使用select命令，可以替换上面的case语句
select 命令允许从单个命令行创建菜单，然后提取输入的答案自动处理。
select 命令的格式如下:

```bash
select variable in list
do
    commands
done
```

list 参数是由构成菜单的空格哥分割的文本选项。select命令会在列表中将每个选项作为一个编好号的选项显示，然后为选项显示一个特殊的
由PS3环境变量定义的提示符

```bash
$ cat a.sh
#!/bin/bash
function diskspace()
{
    clear
    df -k
}

function whoseon()
{
    clear
    who
}

function memusage()
{
    clear
    cat /proc/meminfo
}


PS3="Enter option: "
select option in "Display disk space" "Display logged on users" "Display memory usage" "Exit Progame"
do
    case $option in
        "Exit Progame")
            break;;
        "Display disk space")
            diskspace;;
        "Display logged on users")
            whoseon;;
        "Display memory usage")
            memusage;;
        *)
            clear
            echo "Sorry, Wrong selection"
    esac
done
clear

$ bash a.sh
1) Display disk space       3) Display memory usage
2) Display logged on users  4) Exit Progame
Enter option:

```


### bash 窗口

使用dialog包

格式如下：

`dialog --widget parametes`

* widget 是控件的名字
* parameters 定义了部件窗口的大些以及部件需要的文本

每个dialog部件都提供了两种格式的输出：

1. 使用STDERR
2. 使用退出状态码

dialog命令的退出状态决定于用户选择的按钮，如果选了Yes或OK键，dialog命令会返回退出状态码0.如果选择了Cancel或No按钮，dialog命令会返回退出
状态码为1.
如果部件返回了任何数据，比如菜单选择，那么dialog命令会将数据发送到STDERR

```
dialog --inputbox "Enter your age:" 10 20 2>age.txt
```

上面的10,20指定了窗口的大小，age.txt里面保存了用户的输入

```
dialog --title Testing --msgbox "This is a test" 10 20
dialog --title "Please answer: " --yesno "Is this thing on" 10 20
dialog --textbox /etc/passwd 15 45
dialog --menu "Sys Admin menu" 20 30 10 1 "Display disk space" 2  "Display users" 3 "Display memeory usage" 4 "Exit Progame" 2>test.txt
dialog --title "Select a file" --fselect $HOME/ 10 20 2>file.txt
```

使用dialog的一个例子

```bash
$ cat a.sh
#!/bin/bash
temp=`mktemp -t test1.XXXXXX`
temp2=`mktemp -t test2.XXXXXX`

function diskspace()
{
    df -k > $temp
    dialog --textbox $temp 20 60
}

function whoseon()
{
    who > $temp
    dialog --textbox $temp 20 50
}

function memusage()
{
    cat /proc/meminfo > $temp
    dialog --textbox $temp 20 50
}

while [ 1 ]
do
    dialog --menu "Sys Admin menu" 20 30 10 1 "Display disk space" 2 "Display users" 3 "Display memeory usage" 0 "Exit Progame" 2>$temp2
    if [ $? -eq 1 ]
    then
        break
    fi
    selection=`cat $temp2`
    case $selection in
        0)
            break;;
        1)
            diskspace;;
        2)
            whoseon;;
        3)
            memusage;;
        *)
            dialog --msgbox "Sorry, invalid selection" 10 30
    esac
done
rm -rf $temp 2>/dev/null
rm -rf $temp2 2>/dev/null

```
类似的
KDE包含了kdialog包
GNOME包含了gdialog, zenity


### sed 命令

SED单行脚本快速参考

<http://sed.sourceforge.net/sed1line_zh-CN.html>

替换标记

```
s/pattern/replacement/flags
```

有四种可用的替换标记

* 数字，表明新文本将替换第几处模式匹配的地方
* `g`, 表明新文本将替换所有已有文本出现的地方
* `p`, 表明原来行的内容要打印出来,通常和`-n`选项一起使用，`-n`选项将禁止sed编辑器输出。
但`-p`选项会输出修改过的行。两者配合显示只修改过的行
* `w` file, 将替换的结果写到文件中

```bash
$ echo "a b c a g a" |sed "s/a/c/2"
a b c c g a

$ echo "a b c a g a" |sed "s/a/c/g"
c b c c g c

$ echo "a b c a g a" |sed -n "s/a/c/p"
c b c a g a

$ echo "a b c a g a" |sed -n "s/a/c/gp"
c b c c g c

$ echo "a b c a g a" |sed -n "s/a/c/2p"
a b c c g a

$ echo "a b c a g a" |sed  "s/a/c/w a"
c b c a g a
$ cat a
c b c a g a
```


使用地址

sed编起起中的两种形式的行寻址

* 行的数字范围
* 用文本模式过滤行

格式:

```bash
[address]command
或
address {
    command1
    command2
    command3
}

#将data文件中第2行的dog 替换成cat
sed '2s/dog/cat/'

#将data文件中第2,3行的dog 替换成cat
sed '2,3s/dog/cat/'

#将data文件中第2行到行末的dog 替换成cat
sed '2,$s/dog/cat/'

#将data文件中包含hello字符行的dog 替换成cat
sed '/hello/s/dog/cat'

# 组合命令
$ echo -e "hello\nfox dog" | sed '2{s/fox/elepant/; s/dog/cat/}'
hello
elepant cat
```

删除行

```bash
# 删除第一行
$ echo -e "hello\nfox dog" | sed '1d'
# 删除第二行到第三行
$ echo -e "line 1\nline 2\nline 3" | sed '2,3d'
line 1

$ echo -e "line 1\nline 2\nline 3" | sed '/line 1/d'
line 2
line 3

# 可以删除用两个文本模式来删除某个范围内文本的行，但这么做的时候要小心，指定的第一个模式
# 会"打开"行删除功能，第二个模式会"关闭"行删除功能。sed编辑器会删除这两个行之间的所有行(包括指定的行)
$ cat a
This is line number 1
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6

$ sed '/2/,/3/d' a
This is line number 1
This is line number 4
This is line number 5
This is line number 6

# 除此之外，使用时要小心，因为只要sed在数据流中匹配到了开始模式，删除功能就会打开。这可能会导致意外结果
# 下面第二个数字1再次触发了删除命令，删除了数据流中剩余的行，因为停止模式没有再找到。当然，如果你指定了一个
# 不存在的文本停止模式，会导致开始模式后面的整个数据流都被删除
$ cat a
This is line number 1
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6
This is line number 1 again
This is text you want to keep
This is the last line in the file

$ sed '/1/,/3/d' a
This is line number 4
This is line number 5
This is line number 6
```

插入和附和文本

插入(insert)命令i会在指定行前增加一行

追加(append)命令a会在指定行后增加一行

格式如下：

```bash
## sed '[address]command \ newline'

$ echo "Test line 2" |sed 'i\Test line 1'
Test line 1
Test line 2

$ echo "Test line 2" |sed 'a\Test line 1'
Test line 2
Test line 1
```

上面可以给数据刘中的文本前面或后面添加文本，但是如何添加到数据流里面呢？
要给数据流中出入或附加数据，必须用寻址来告诉sed你想让数据出现在什么位置。
你可以在用这些命令时执行一个行地址。可以匹配一个数字行号或文本模式，但不能用地址区间，
这不符合逻辑，因为你只能将文本插入或附件到单个行的前面或后面，而不能是行区间的前面或后面

```bash
$ cat a
This is line number 1
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6

$ sed '3i\This is a inserted line' a
This is line number 1
This is line number 2
This is a inserted line
This is line number 3
This is line number 4
This is line number 5
This is line number 6

$ sed '3a\This is a append line' a
This is line number 1
This is line number 2
This is line number 3
This is a append line
This is line number 4
This is line number 5
This is line number 6

$ sed '$a\This is a append line' a
This is line number 1
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6
This is a append line

#插入多行，每一行要使用反斜线
$ sed '2a\This is inserted line 1.\
> This is inserted line 2.' a
This is line number 1
This is line number 2
This is inserted line 1.
This is inserted line 2.
This is line number 3
This is line number 4
This is line number 5
This is line number 6

```

修改行
修改命令允许修改数据流中整行文本的内容,跟插入和附加命令的工作机制是一样的

```bash

$ cat a
This is line number 1
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6

$ sed '3c\This is a changed line' a
This is line number 1
This is line number 2
This is a changed line
This is line number 4
This is line number 5
This is line number 6

$ sed '/number 3/c\This is a changed line' a
This is line number 1
This is line number 2
This is a changed line
This is line number 4
This is line number 5
This is line number 6

$ cat a
This is line number 1
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6
This is line number 1 again
This is line number 2 again

#文本模式会修改它匹配的任意行
$ sed '/number 1/c\This is a changed line' a
This is a changed line
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6
This is a changed line
This is line number 2 again

#注意在使用地址区间时，并不是逐一替换那两行文本，而是用一行文本在替换地址区间的所有文本
$ sed '2,3c\This is a changed line' a
This is line number 1
This is a changed line
This is line number 4
This is line number 5
This is line number 6
This is line number 1 again
This is line number 2 again

# 可以看出来c命令与d命令使用文本模式区间时一样，都是开始匹配的打开，结束匹配的关闭命令的操作，
# 第二个number 1没有结束匹配，所以到文件末的都被修改了
$ sed '/number 1/,/number 3/c\This is a changed line' a
This is a changed line
This is line number 4
This is line number 5
This is line number 6
```

转换命令
转换y命令是唯一可以处理单个字符的sed命令，格式如下：

```
[address]y/inchars/outchars/
```

* 转换命令会进行inchars和outchars值的一对一映射。
* inchars中的第一个字符会被转换为outchars中的第一个字符，
* inchars中的第二个字符会被转换为outchars中的第二个字符，
* 如果inchars和outchars的长度不同，则会报错
* inchars中指定的字符都会被转换，你无法限定只转换出现在某个地方的地址,（相当与sed s命令后面添加了g）
*

```bash
$ echo "a b c d A c a f" |sed 'y/abc/jkl/'
j k l d A l j f
$ echo -e "a b c d A c a f\na b c d" |sed 'y/abc/jkl/'
j k l d A l j f
j k l d
$ echo -e "a b c d A c a f\na b c d" |sed '1y/abc/jkl/'
j k l d A l j f
a b c d
$ echo -e "a b c d A c a f\na b c d" |sed '1y/abc/jk/'
sed: -e expression #1, char 10: strings for `y' command are different lengths
```

打印
小写p命令用来打印文本行
等号(=)用来打印行号
l(L的小写)用来列出行,会打印出不可打印的ACSII字符,如\t，换行符等

```bash
$ echo "hello" | sed 'p'
hello
hello

$ echo "hello" | sed -n 'p'
hello

$ sed '=' a
1
This is line number 1
2
This is line number 2
3
This is line number 3
4
This is line number 4
5
This is line number 5
6
This is line number 6
7
This is line number 1 again
8
This is line number 2 again

$ cat b
    hello world
    ni hao
$ sed -n 'l' b
\thello world$
\tni hao$
```

用sed与文件一起工作
1.向文件写入

```bash
# 将a文件的2,3行写入test文件
$ cat a
This is line number 1
This is line number 2
This is line number 3
This is line number 4
This is line number 5
This is line number 6
This is line number 1 again
This is line number 2 again

$ sed -n '2,3w test' a
$ cat test
This is line number 2
This is line number 3
```

1.从文件读取数据
读取命令(r)允许你将一个独立的文件中的数据流插入(i)到数据流中
格式:

```
[address]r filename
地址只能是单独一个行号或文本模式，不能是地址区间
```

```bash
$ cat a
This is an added line.
This is the second added line.
$ cat b
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
This is line number 5.
$ sed '2r a' b
This is line number 1.
This is line number 2.
This is an added line.
This is the second added line.
This is line number 3.
This is line number 4.
This is line number 5.
```

多行命令

小写的n命令会告诉sed编辑器移动到数据流中的下一行文本, 而不用重新回到命令的最开始再执行一遍。如只删除包含header行的下一行

```bash
$ cat a
This is an header line.

This is the second line.

This is the bottom line.

$ sed '/header/{n;d}' a
This is an header line.
This is the second line.

This is the bottom line.
```

大写的N会将下一行文本添加到已经在模式空间的文本上，这样的作用是将数据流中的两个文本合并到同一个模式空间。文本行仍然用换行分割符
，但sed编辑器现在会将两行文本当成一行来处理。

```
$ cat a
This is the header line
This is the second line
This is the third line
This is the last line

$ sed '/second/{N;s/\n/ /}' a
This is the header line
This is the second line This is the third line
This is the last line
```

如果你要替换的文本在两行，这个命令会很好用

```
$ cat a
The first meeting of Linux System
Administrator's group will be held on Tuesday.
All System Administrators should attend this meeting.
Thank you for your attendance.

$ sed 's/System Administrator/Desktop User/' a
The first meeting of Linux System
Administrator's group will be held on Tuesday.
All Desktop Users should attend this meeting.
Thank you for your attendance.

# 注意在System.Administrator之间使用了通配符.来匹配空格和换行符两种情况
$ sed 'N; s/System.Administrator/Desktop User/' a
The first meeting of Linux Desktop User's group will be held on Tuesday.
All Desktop Users should attend this meeting.
Thank you for your attendance.

# 上面的情况会将两行合并成一行，并不是我们想要的，可以用匹配多行来处理
$ sed 'N; s/System\nAdministrator/Desktop\nUser/; s/System Administrator/Desktop User/' a
The first meeting of Linux Desktop
User's group will be held on Tuesday.
All Desktop Users should attend this meeting.
Thank you for your attendance.

# 上面的情况也有个小问题，它会每次在执行的时候都先读下一行到模式空间，当到了最后一行的时候，没有可读的了，sed命令停止了
, 最后一行的数据不会被处理。 为了解决这个问题，可以将单行命令方在N命令前，并将多行命令放在N后面

$ cat a
The first meeting of Linux System
Administrator's group will be held on Tuesday.
All System Administrators should attend this meeting.

$ sed 's/System Administrator/Desktop User/;N;s/System\nAdministrator/Desktop\nUser/' a
The first meeting of Linux Desktop
User's group will be held on Tuesday.
All Desktop Users should attend this meeting.
```

多行删除命令
d和N命令一起使用时，删除命令会在不同行查找满足匹配模式的行并删除匹配的两行

```bash
$ cat a
The first meeting of Linux System
Administrator's group will be held on Tuesday.
All System Administrators should attend this meeting.

$ sed 'N;/System\nAdministrator/d' a
All System Administrators should attend this meeting.

# 使用D，它只会删除模式空间中的第一行
$ sed 'N;/System\nAdministrator/D' a
Administrator's group will be held on Tuesday.
All System Administrators should attend this meeting.

# D命令多于删除满足指定条件行的前一行。
```


保持空间

```
h  将模式空间复制到保持空间
H  将模式空间附加到保持空间
g  将保持空间复制到模式空间
G  将保持空间附加到模式空间
x  交换模式空间和保持空间的内容
```

通常，在使用h或H命令将字符串移到保持空间后，最终你要用g,G或x命令来将保存的字符串移回到模式空间

排除命令

感叹号(!) 让原来起作用的行不起作用。

反转一个文本(可以使用tac命令)

```
$ sed -n '1!G;h;$p' a
All System Administrators should attend this meeting.
Administrator's group will be held on Tuesday.
The first meeting of Linux System
```


跳转
```
[address]b [label]
```

address 参数决定了哪行或哪些行的数据会触发跳转命令，label参数定义了要跳转到的位置。如果没有参数label，
跳转命令会跳转到脚本的结束。


```bash
$ cat b
This is the header line.
This is the first data line.
This is the second data line.
This is the last line.

# 第2,3行跳过了替换命令
$ sed '{2,3b; s/This is/Is This/}' b
Is This the header line.
This is the first data line.
This is the second data line.
Is This the last line.

# 第2, 3行跳转到标签处, 执行标签后面的命令
$ sed '{2,3b junmp1; s/This is/Is This/; :junmp1 s/line/line-line/}' b
Is This the header line-line.
This is the first data line-line.
This is the second data line-line.
Is This the last line-line.

#标签还可以放在前面，行成一个循环,一次去掉逗号
# 注意b前面指定了地址模式，不然这个循环会是无限循环
$ echo "This, is, a, test, to, remove, commas." |sed -n "{:start s/,//1p; /,/b start}"
This is, a, test, to, remove, commas.
This is a, test, to, remove, commas.
This is a test, to, remove, commas.
This is a test to, remove, commas.
This is a test to remove, commas.
This is a test to remove commas.
```

类似于跳转命令，测试(test)命令也用来改变sed编辑器脚本的流，测试命令会基于替换命令的输出跳转到一个标签，而不是基于地址跳转到
一个标签。

```
[address]t [label]
```

跟跳转命令一样，在没有指定标签的情况下，如果测试成功，sed跳转到脚本的末尾


```bash
$ sed '{s/first/matched/;t;s/This is the/No match on/}' b
No match on header line.
This is the matched data line.
No match on second data line.
No match on last line.


$ echo "This, is, a, test, to, remove, commas." |sed -n "{:start s/,//1p; t start}"
This is, a, test, to, remove, commas.
This is a, test, to, remove, commas.
This is a test, to, remove, commas.
This is a test to, remove, commas.
This is a test to remove, commas.
This is a test to remove commas.
```

模式替换

```bash
$ echo "The cat sleep in his hat" |sed 's/cat/"cat"/'
The "cat" sleep in his hat

$ echo "The cat sleep in his hat" |sed 's/.at/".at"/g'
The ".at" sleep in his ".at"

#使用and符号(&)可以用来替换命令中的匹配模式
#如上面的例子
$ echo "The cat sleep in his hat" | sed 's/.at/"&"/g'
The "cat" sleep in his "hat"

# sed 用圆括号来替换模式的子串，然后用特殊字符来引用
# 用\1 引用System
$ echo "The System Administrator manual" | sed 's/\(System\) Administrator/\1 User/'
The System User manual

$ echo 1234567890 | sed -n '{:start s/\(.*[0-9]\)\([0-9]\{3\}\)/\1,\2/p; t start}'
1234567,890
1234,567,890
1,234,567,890
```

一些长用的sed命令技巧
每行后面添加一空白行

```
sed G filename
```
每行后面添加两空白行

```
sed 'G;G' filename
```

显示行号

```bash
$ sed '=' b |sed 'N; s/\n/ /'
1 This is the header line.
2 This is the first data line.
3 This is the second data line.
4 This is the last line.
```

### 正则表达式

正则表达式是基于正则表达式引擎实现的

Linux中流行的两种正则表达式引擎

* POSIX基本的正则表达式引擎(BRE)引擎
* POSIX扩展的正则表达式引擎(ERE)引擎


### mysql命令行参数

```
-A 禁止自动重新生成hash表
-e 执行语句并退出
-E 竖直方向显示查询结果，每行一个数据字段
-H 用HTML代码显示查询输出
-X 用XHTML代码显示查询输出
```

mysql中的命令分两种:

* 特殊的mysql命令
* 标准的SQL语句

mysql的内置命令:

```
List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for 'help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.

For server side help, type 'help contents'
```

简单谈谈上面不错的一些命令:

* ? 查看帮助信息
* delimiter 设置sql语句的分割符
* edit 使用命令行编辑器编辑命令，输入edit之后，变会打开vim编辑器，可以在里面输入要执行的sql语句，报错之后再按分号(;)执行刚刚输入的命令
* pager more当一次查询内容太多，一屏无法显示时，可以试试，类似linux下more命令
* tee 将所有输出定向到指定文件
* notee 结束将所有输出定向到执行文件


在脚本中使用mysql
发送命令并退出

```bash
$ mysql -u root -ppasswd -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| reviewdb           |
| test_db            |
+--------------------+
```

但是返回的结果包含标题或者制表符，不利于提取数据
可以加上-Bs参数

```bash
$ mysql -Bs -u root -plinliang -e "show databases;"
information_schema
mysql
reviewdb
test_db
```

想传递多个命令的时候可以使用EOF重定向

```bash
$ mysql -u root -plinliang <<EOF
show databases;
use reviewdb;
show tables;
EOF
```

### 使用Web终端上网

* lynx
* w3m
