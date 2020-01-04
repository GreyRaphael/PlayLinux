# Shell

- [Shell](#shell)
  - [shell variable](#shell-variable)
  - [shell loop](#shell-loop)

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

example: shell condition
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