# Fund Tradeing Server

> 在阿里云配置新的Ubuntu来做基金量化交易

## Preparation

1. Navicat dump旧Ubuntu数据库(MongoDB, MySQL)
2. 购买服务器，重置实例root密码
3. 配置安全组，设置可访问的port
4. ssh连接服务器root账号

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

## Install MongoDB

```bash
sudo apt install mongodb
mongo -version

# 开启服务端
# service mongodb start
sudo systemctl start mongodb

sudo vim /etc/mongodb.conf
# 将bind改为0.0.0.0

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
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'new_password' WITH GRANT OPTION;

flush privileges;
exit

# 重启服务
systemctl restart mysql
# 查看是否运行
ps -ef|grep mysql

pip install mysql-connector-python
```

## Config JupyterLab

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
jupyter lab

Ctrl+C

# 启动远程
nohup jupyter lab ~/JupyterWork/ &

# 远程访问 xxx.xxx.xxx.xxx:8888, 并输入密码
```