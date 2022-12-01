# Fund Tradeing Server

> 在阿里云配置新的Ubuntu来做基金量化交易

## Preparation

1. Navicat dump旧Ubuntu数据库(MongoDB, MySQL)
2. 购买服务器，重置实例root密码
3. 配置安全组，设置可访问的port
4. ssh连接服务器root账号

## Config Port

如果是ECS, 将如下保存为csv并导入阿里云安全组

```csv
"网络类型","授权方向","授权策略","IP协议","端口范围","优先级","源IP地址段","源IPv6地址段","源安全组","源端安全组名称","源安全组所属阿里云账户ID","源端端口范围","目标IP地址段","目的IPv6地址段","目标安全组","目的端安全组名称","目标安全组所属阿里云账户ID","描述信息","创建时间（UTC）","",""
"intranet","ingress","Accept","TCP","5000/5000","1","0.0.0.0/0","","","","","","","","","","","Flask","2021-03-03T03:11:31Z","",""
"intranet","ingress","Accept","TCP","465/465","1","0.0.0.0/0","","","","","","","","","","","mail","2021-03-03T03:11:16Z","",""
"intranet","ingress","Accept","TCP","8888/8888","1","0.0.0.0/0","","","","","","","","","","","Jupyter","2021-03-03T03:11:03Z","",""
"intranet","ingress","Accept","TCP","80/80","1","0.0.0.0/0","","","","","","","","","","","http","2021-03-03T03:10:43Z","",""
"intranet","ingress","Accept","TCP","6379/6379","1","0.0.0.0/0","","","","","","","","","","","redis","2021-03-03T03:10:36Z","",""
"intranet","ingress","Accept","TCP","3306/3306","1","0.0.0.0/0","","","","","","","","","","","MySQL","2021-03-03T03:10:29Z","",""
"intranet","ingress","Accept","TCP","443/443","1","0.0.0.0/0","","","","","","","","","","","https","2021-03-03T03:10:21Z","",""
"intranet","ingress","Accept","TCP","22/22","1","0.0.0.0/0","","","","","","","","","","","ssh","2021-03-03T03:10:14Z","",""
"intranet","ingress","Accept","TCP","27017/27017","1","0.0.0.0/0","","","","","","","","","","","MongoDB","2021-03-03T03:10:04Z","",""
"intranet","ingress","Accept","TCP","8050/8050","1","0.0.0.0/0","","","","","","","","","","","scrapinghub/splash	","2021-03-03T03:09:16Z","",""
"intranet","egress","Accept","TCP","3306/3306","1","","","","","","","0.0.0.0/0","","","","","For MySQL","2018-12-20T16:02:20Z","",""
"intranet","egress","Accept","TCP","6379/6379","1","","","","","","","0.0.0.0/0","","","","","Redis out","2018-12-18T08:16:09Z","",""
"intranet","egress","Accept","TCP","5671/5672","1","","","","","","","0.0.0.0/0","","","","","For RabbitMQ Out","2018-12-11T04:03:25Z","",""
```

如果是轻量服务器，直接输入

| 应用类型  | 协议  | 端口范围  | 备注                 |
|-------|-----|-------|--------------------|
| HTTP  | TCP | 80    |                    |
| HTPS  | TCP | 443   |                    |
| SSH   | TCP | 22    |                    |
| MYSQL | TCP | 3306  | MySQL              |
| 自定义   | TCP | 5000  | Flask              |
| 自定义   | TCP | 465   | Mail               |
| 自定义   | TCP | 8888  | Jupyter            |
| 自定义   | TCP | 6379  | Redis              |
| 自定义   | TCP | 27017 | MongoDB            |
| 自定义   | TCP | 8050  | scrapinghub/splash |


## Add sudo user

```bash
# 最开始是root用户, ubuntu推荐adduser, 会创建home, 新建password
adduser james

# moris进入sudoers
sudo usermod -a -G sudo james

su james

sudo apt update
sudo apt upgrade
```

## Install Anaconda

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-xxx-Linux-x86_64.sh

chmod +x Anaconda3-xxx-Linux-x86_64.sh

./Anaconda3-xxx-Linux-x86_64.sh

# 配置conda源
# https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

# 配置pip源头
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

conda update --all
conda clean --all
```

## Install Miniconda

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-xxx-Linux-x86_64.sh

chmod +x Miniconda3-xxx-Linux-x86_64.sh

./Miniconda3-xxx-Linux-x86_64.sh

# 配置conda源
# https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

# 配置pip源头
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

conda update --all

conda install jupyterlab

conda clean --all
```

## Install MongoDB

```bash
sudo apt install mongodb
mongo -version

# 开启服务端
# service mongodb start
sudo systemctl start mongodb

sudo vim /etc/mongodb.conf
# 将bind_ip = 0.0.0.0
# 将auth = true

sudo systemctl restart mongodb
sudo systemctl enable mongodb
# 查看是否启动
ps ajx|grep mongo

# 使用mongodb客户端
# MySQL的服务器和客户端都是mysql
mongo

use admin
db.createUser({user:'grey', pwd:'xxxxxx', roles:[{role:'root',db:'admin'}]})
exit

conda install pymongo
```

## Install MySQL

```bash
sudo apt update
sudo apt install mysql-server mysql-client
sudo systemctl enable mysql
sudo systemctl start mysql

# 查看默认密码
sudo vim /etc/mysql/debian.cnf
# 登录
mysql -u debian-sys-maint -p 

# 添加root账号
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
# 修改plugin
mysql> USE mysql;
mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='root';
mysql> FLUSH PRIVILEGES;
mysql> exit;

# 用new_password登录root
mysql -u root -p
exit

# 允许远程连接
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
# 将bind-address=127.0.0.1注释

# 登陆mysql
mysql -u root -p

CREATE USER 'root'@'%' IDENTIFIED BY 'new_password';
# GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'new_password' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

flush privileges;
exit

# 重启服务
systemctl restart mysql
# 查看是否运行
ps -ef|grep mysql

pip install mysql-connector-python
```

## Config Jupyter

### Jupyter Notebook

```bash
jupyter notebook --generate-config
# 生成配置文件: ~/.jupyter/jupyter_notebook_config.py

jupyter notebook password
# 生成加密的密码: ~/.jupyter/jupyter_notebook_config.json
```

```py
# jupyter_notebook_config.py
c.NotebookApp.ip = '' 
# 阿里云''默认是私有ip
# 将jupyter_notebook_config.json里面的密码复制到这
c.NotebookApp.password = "sha1:9ba3e4304a4a:d322d173f6f43a74e3445c75aa02c218f2759a1c"
c.NotebookApp.open_browser = False
```

```bash
# 测试一下
jupyter notebook

Ctrl+C

# 启动远程
mkdir JupyterWork
nohup jupyter notebook ~/JupyterWork/ &

# 远程访问 xxx.xxx.xxx.xxx:8888, 并输入密码
```

### Jupyterlab

> Jupyterlab随着更新，配置方式可能改变，但Jupyter Notebook一直稳定可以使用

```bash
jupyter server --generate-config
# 生成配置文件: ~/.jupyter/jupyter_server_config.py

jupyter lab password
# 生成加密的密码: ~/.jupyter/jupyter_server_config.json
```

```py
# jupyter_server_config.py
c.NotebookApp.ip = '' 
# 阿里云''默认是私有ip
# 将jupyter_server_config.json里面的密码复制到这
c.NotebookApp.password = "sha1:9ba3e4304a4a:d322d173f6f43a74e3445c75aa02c218f2759a1c"
c.NotebookApp.open_browser = False
```

```bash
# 测试一下
jupyter lab

Ctrl+C

# 启动远程
mkdir JupyterWork
nohup jupyter lab ~/JupyterWork/ &

# 远程访问 xxx.xxx.xxx.xxx:8888, 并输入密码
```

## Install Other Python Package

```bash
pip install schedule
```

## Install Redis

```bash
sudo apt install redis-server
ps -ax|grep redis

# 配置remote redis
sudo vim /etc/redis/redis.conf

# 注释掉bind
bind 127.0.0.1
# 修改requirepass
requirepass your_password
```

```bash
sudo systemctl restart redis-server
sudo systemctl enable redis-server

redis-cli
127.0.0.1:6379> auth your_password
OK
127.0.0.1:6379> ping
PONG
```

## Remote Login without Password

> 通过public key，不通过密码登录remote linux

```bash
# in windows
# 使用git产生privata key 和public key，一直Enter
ssh-keygen -t rsa -C "youremail@example.com"

# 在 %USERPROFILE%/.ssh目录下面生成两个文件
id_rsa
id_rsa.pub

# 将id_rsa.pub复制到remote linux
```

```bash
# in remote Linux
# 注意: 将内容复制到 authorized_keys，独占一行，不能有回车
cat id_rsa.pub > ~/.ssh/authorized_keys

# 可能还需要配置linux权限
sudo vim /etc/ssh/sshd_config
# 取消如下注释
# AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2

# 可能需要配置访问
chmod 600 .ssh/authorized_keys
chmod 700 .ssh

sudo service sshd restart
```