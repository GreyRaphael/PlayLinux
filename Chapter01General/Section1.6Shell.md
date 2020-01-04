# Shell

- [Shell](#shell)
  - [shell variable](#shell-variable)
  - [shell condition](#shell-condition)
  - [shell loop](#shell-loop)
  - [shell function](#shell-function)
  - [shell tools](#shell-tools)

shell运行command
- fork产生的子进程exec(command)
- 父进程切换到后台等待回收子进程

```bash
# 加括号，只是子进程的pwd会改变，父进程的pwd不变
moris@ubuntu:~$ (pwd;cd ..;pwd)
/home/moris
/home
moris@ubuntu:~$ pwd
/home/moris

# 不加括号，子进程的pwd会改变，父进程的pwd也会改变
moris@ubuntu:~$ pwd;cd ..;pwd
/home/moris
/home
moris@ubuntu:/home$ pwd
/home
```

bash builtins: `man bash-builtins`

example: 检测上一个程序执行是否成功

```bash
moris@ubuntu:~$ pwd
# 上一个程序执行的Exit Status，0表示成功
moris@ubuntu:~$ echo $?
0

moris@ubuntu:~$ cd abcd
-bash: cd: abcd: No such file or directory
# 上一个程序执行失败
moris@ubuntu:~$ echo $?
1

# 上一个程序执行成功
moris@ubuntu:~$ echo $?
0
```

example: simple shell script

```bash
#! /bin/sh
cd ..
pwd
```

```bash
moris@ubuntu:~$ vim t1.sh
moris@ubuntu:~$ chmod +x t1.sh
moris@ubuntu:~$ ./t1.sh
/home
moris@ubuntu:~$
```

example: run file with sh

```bash
cd ..
pwd
```

```bash
moris@ubuntu:~$ vim t2.sh
moris@ubuntu:~$ /bin/sh t2.sh
/home
moris@ubuntu:~$ sh t2.sh
/home
moris@ubuntu:~$ /bin/bash t2.sh
# bash是增强版的sh
/home
```

example: `source` and `.`
> `source`或者`.`命令是Shell的内建命令，这种方式也不会创建子Shell，而是直接在交互式Shell下逐行执行脚本中的命令

```bash
moris@ubuntu:~$ source t2.sh
/home
moris@ubuntu:/home$ cd moris/

moris@ubuntu:~$ . ./t2.sh
/home
moris@ubuntu:/home$
```

## shell variable

shell变量:
- 环境变量: `env` or `printenv` 显示的环境变量，环境变量都是key-value对
- 本地变量: `set`

```bash
# 定义一个本地变量
moris@ubuntu:~$ name='grey'
# 在环境变量中查找name: nothing
moris@ubuntu:~$ env|grep name
# 在本地变量中查找name
moris@ubuntu:~$ set|grep name
name=grey
# 从本地变量到环境变量
moris@ubuntu:~$ export name
moris@ubuntu:~$ env|grep name
name=grey
# 删除本地变量或者环境变量
moris@ubuntu:~$ unset name
moris@ubuntu:~$ env|grep name

# 直接定义环境变量
moris@ubuntu:~$ export AGE=666
# 调用一个变量
moris@ubuntu:~$ echo $AGE
666
# 没有$就只是字符串
moris@ubuntu:~$ echo AGE
AGE
# 当前shell类型是bash, bash是一个增强版的sh
moris@ubuntu:~$ echo $SHELL
/bin/bash
```

文件名替换: `*`, `?`, `[]`

example: 类似regex

```bash
moris@ubuntu:~$ ls *.py
moris@ubuntu:~$ ls ?.py
moris@ubuntu:~$ ls test[0123].py
```

命令替换: 将命令结果作为变量内容

```bash
moris@ubuntu:~$ date
Fri Jan  3 23:39:18 CST 2020
moris@ubuntu:~$ MYVAR=date
moris@ubuntu:~$ echo $MYVAR
date
moris@ubuntu:~$ MYVAR=$(date)
moris@ubuntu:~$ echo $MYVAR
Fri Jan 3 23:40:03 CST 2020

# 简写形式
moris@ubuntu:~$ MYVAR=`date`
moris@ubuntu:~$ echo $MYVAR
Fri Jan 3 23:40:45 CST 2020
```

算术替换

```bash
moris@ubuntu:~$ MYVAR=999
moris@ubuntu:~$ echo $MYVAR
999
moris@ubuntu:~$ echo $MYVAR+1
999+1
moris@ubuntu:~$ echo $(($MYVAR+1))
1000
moris@ubuntu:~$ echo $MYVAR
999
moris@ubuntu:~$ echo $((MYVAR+1))
1000
moris@ubuntu:~$ echo $((MYVAR))
999
```

字符相关

```bash
# 转义字符
moris@ubuntu:~$ echo \$SHELL
$SHELL

# create a file named as "-hello"
moris@ubuntu:~$ touch -hello
touch: invalid option -- 'e'
Try 'touch --help' for more information.
moris@ubuntu:~$ touch -- -hello
moris@ubuntu:~$ ls
anaconda3  dump.rdb  Fund  -hello  JupyterWork  nohup.out  seaborn-data
# method1: delete -hello file
moris@ubuntu:~$ rm -- -hello

# method2: delete -hello file
moris@ubuntu:~$ rm ./-hello

# 续行
moris@ubuntu:~$ ls \
> -l
total 384
drwxrwxr-x 27 moris moris   4096 Dec 19 17:45 anaconda3
-rw-rw-r--  1 moris moris     92 Oct 22 11:39 dump.rdb
drwxrwxr-x  2 moris moris   4096 Oct 31 17:00 Fund
drwxrwxr-x  3 moris moris   4096 Jan  3 16:52 JupyterWork
-rw-------  1 moris moris 360557 Jan  3 23:42 nohup.out
drwxrwxr-x  2 moris moris   4096 Nov 25 11:02 seaborn-data
-rw-rw-r--  1 moris moris     91 Jan  3 23:11 t1.sh

# 单引号不展开变量，双引号会展开变量，其他都相同
moris@ubuntu:~$ echo $MYVAR
999
moris@ubuntu:~$ echo '$MYVAR'
$MYVAR
moris@ubuntu:~$ echo "$MYVAR"
999
moris@ubuntu:~$ echo "$MYVAR+666"
999+666
moris@ubuntu:~$ echo '$MYVAR+666'
$MYVAR+666
```

example: `echo`

```bash
moris@ubuntu:~$ echo 'hello\n\n'
hello\n\n
moris@ubuntu:~$ echo -e 'hello\n\n'
hello


moris@ubuntu:~$ echo -n 'hello\n\n'
hello\n\nmoris@ubuntu:~$
```

example: `tee`, tee命令把结果输出到标准输出，另一个副本输出到相应文件。

```bash
moris@ubuntu:~$ ls |tee test.txt
anaconda3
dump.rdb
Fund
JupyterWork
nohup.out
seaborn-data
t1.sh
test.txt
```

standard input & output
- stdin: 0
- stdout: 1
- stderr: 2

```bash
cmd > file             #把标准输出重定向到新文件中
cmd >> file            #追加
cmd > file 2>&1        #标准出错也重定向到1屏幕上，但是1已经重定向到了file, 所以标准出错也重定向到文件
cmd >> file 2>&1
cmd < file1 > file2    #从file1获得输入，将结果重定向到file2
cmd < &fd              #把文件描述符fd作为标准输入
cmd > &fd              #把文件描述符fd作为标准输出
cmd < &-               #关闭标准输入

# 不想看输出，重定向到黑洞
ll > /dev/null
```

## shell condition

> `test` is the same as `[]`

```bash
# 数值比较大小 -eq, -ne, -gt, -ge, -lt, -le
moris@ubuntu:~$ VAR1=6
moris@ubuntu:~$ test $VAR1 -gt 1
moris@ubuntu:~$ echo $?
0
moris@ubuntu:~$ test $VAR1 -lt 1
moris@ubuntu:~$ echo $?
1
moris@ubuntu:~$ [ $VAR1 -lt 1 ]
moris@ubuntu:~$ echo $?
1

# check directory
moris@ubuntu:~$ [ -d anaconda3 ]
moris@ubuntu:~$ echo $?
0

# 字符串比较相等、不相等: == , !=
moris@ubuntu:~$ s1=hello
moris@ubuntu:~$ s2=hello
moris@ubuntu:~$ s3=world
moris@ubuntu:~$ [ s1 == s2 ]
moris@ubuntu:~$ echo $?
1
moris@ubuntu:~$ [ $s1 == $s2 ]
moris@ubuntu:~$ echo $?
0
moris@ubuntu:~$ [ $s1 == $s3 ]
moris@ubuntu:~$ echo $?
1

moris@ubuntu:~$ s4=
moris@ubuntu:~$ [ $s1 == $s4 ]
-bash: [: hello: unary operator expected
# 对于空字符串需要用""
moris@ubuntu:~$ [ "$s1" == "$s4" ]
moris@ubuntu:~$ echo $?
1

# 与或非: !, -a, -o
# 最好用""将变量包含起来，避免空变量导致的bug
moris@ubuntu:~$ unset s1
moris@ubuntu:~$ [ -d anaconda3 -a "$s1" = 'hello' ]
moris@ubuntu:~$ echo $?
1
```

example: if else

```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
    echo 'hello world'
fi


if [ -f /bin/bash ]
then 
    echo "/bin/bash is a file"
    echo "/bin/bash is a file"
    echo "/bin/bash is a file"
else 
    echo "/bin/bash is NOT a file"
    echo "/bin/bash is NOT a file"
    echo "/bin/bash is NOT a file"
fi


if :; then echo "always true"; fi
```

```bash
#! /bin/sh

echo "Is it morning? Please answer yes or no."
read YES_OR_NO
if [ "$YES_OR_NO" = "yes" ]; then
    echo "Good morning!"
elif [ "$YES_OR_NO" = "no" ]; then
    echo "Good afternoon!"
else
    echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
    exit 1
fi
exit 0
```

```bash
#! /bin/sh

echo "Is it morning? Please answer yes or no."
read YES_OR_NO
case "$YES_OR_NO" in
yes|y|Yes|YES)
    echo "Good Morning!"
    echo "Good Morning!"
    echo "Good Morning!"
    echo "Good Morning!";;
[nN]*)
    echo "Good Afternoon!";;
*)
    echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
    exit 1;;

# ;;是结束标志
# esac表示case语句结束
esac
exit 0
```

```bash
# 常见于/etc/init.d/配置文件中，比如nginx -s stop
case "$1" in
    start)
        ...
    ;;
    stop)
        ...
    ;;
    reload | force-reload)
        ...
    ;;
    restart)
    ...
    *)
        log_success_msg "Usage: nfs-kernel-server {start|stop|status|reload|force-reload|restart}"
        exit 1
    ;;
esac
```

## shell loop

```bash
#! /bin/sh

for FRUIT in apple banana pear; do
    echo "I like $FRUIT"
done
```

```bash
#! /bin/sh

echo "Enter password:"
# 读取的变量存入TRY
read TRY
# "$TRY"表示变量
while [ "$TRY" != "secret" ]; do
    echo "Sorry, try again"
    read TRY
done
```

```bash
#! /bin/sh

COUNTER=1 # COUNTER是字符串"1"
while [ "$COUNTER" -lt 10 ]; do
    echo "Here we go again"
    # 字符串转换为数值: $(())
    COUNTER=$(($COUNTER+1))
done
```

example: argv

```bash
#! /bin/sh

echo $0
echo $1
echo $2
echo $3

# 不包含$0的参数个数
echo $#
# 参数列表，用于for ... in ...
echo $@
# 参数列表，同上
echo $*
# 上一条命令的Exit Status
echo $?
# 当前进程号
echo $$
```

example: shift command

```bash
#! /bin/sh

echo "The program $0 is now running"
echo "The first parameter is $1"
echo "The second parameter is $2"
echo "The parameter list is $@"
shift
echo "The first parameter is $1"
echo "The second parameter is $2"
echo "The parameter list is $@"
```

```bash
moris@ubuntu:~$ source t1.sh arg1 arg2 arg3
The program -bash is now running
The first parameter is arg1
The second parameter is arg2
The parameter list is arg1 arg2 arg3
The first parameter is arg2
The second parameter is arg3
The parameter list is arg2 arg3
```

## shell function

```bash
#! /bin/sh

foo(){ echo "Function foo is called";}


echo "-=start=-"
foo
echo "-=end=-"
```

```bash
#! /bin/sh

foo(){ 
    echo $0
    echo $1
    echo $2
    echo "Function foo is called";
}


echo "-=start=-"
foo aa bb cc
echo "-=end=-"
```

```bash
# output
(base) moris@ubuntu:~$ vim t1.sh
(base) moris@ubuntu:~$ source t1.sh
-=start=-
-bash
aa
bb
Function foo is called
-=end=-
(base) moris@ubuntu:~$ chmod +x t1.sh
(base) moris@ubuntu:~$ ./t1.sh
-=start=-
./t1.sh
aa
bb
Function foo is called
-=end=-
```

```bash
#! /bin/sh

is_directory()
{
    DIR_NAME=$1
    if [ ! -d $DIR_NAME ]; then
    return 1
    else
    return 0
    fi
}

for DIR in "$@"; do
    if is_directory "$DIR"
    then :
    else
    echo "$DIR doesn't exist. Creating it now..."
    mkdir $DIR > /dev/null 2>&1
    if [ $? -ne 0 ]; then
        echo "Cannot create directory $DIR"
        exit 1
    fi
    fi
done
```

## shell tools

shell配合如下工具使用效果更佳
- grep
- find
- xargs
- sed: 行处理工具
- awk: 行处理工具+列处理工具

tip: Basic 正则与Extend正则不同，Python, `egrep`, `grep -E`, `sed -r`是Extend正则，`?+{}|()`不用特殊处理；`grep`, `sed`是Basic正则，需要`\`处理

```bash
# file
192.168.1.1
1234.234.04.5678
123.4234.045.678
abcde
```

```bash
moris@ubuntu:~$ vim file
# basic regex
moris@ubuntu:~$ grep '([0-9]{1,3}\.){3}[0-9]{1,3}' file
moris@ubuntu:~$ grep '\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}' file
192.168.1.1
1234.234.04.5678
# extend regex
moris@ubuntu:~$ egrep '([0-9]{1,3}\.){3}[0-9]{1,3}' file
192.168.1.1
1234.234.04.5678
moris@ubuntu:~$ grep -E '([0-9]{1,3}\.){3}[0-9]{1,3}' file
192.168.1.1
1234.234.04.5678
```

example: exec vs xargs

```bash
# 找到文件之后再执行，但find的结果可能太多，导致溢出
find . -perm -7 -print -exec chmod o-w
# 找到一批，执行一批，不会溢出
find . -perm -7 -print | xargs chmod o-w
```

example: `sed`去掉html的tag
> `sed`行处理工具

```bash
moris@ubuntu:~$ cat test.html
<html><head><title>Hello World</title></head>
<body>Welcome to the world of regexp!</body></html>
moris@ubuntu:~$ sed 's/<[^>]*>/ /g' test.html
   Hello World
 Welcome to the world of regexp!
```

example: `awk`

```bash
# 打印第1列
ps ax|awk '{print $1;}'
# $0代表一行
```

```bash
moris@ubuntu:~$ cat testfile
ProductA  30
ProductB  76
ProductC  55
moris@ubuntu:~$ awk '{print $2;}' testfile
30
76
55
```

