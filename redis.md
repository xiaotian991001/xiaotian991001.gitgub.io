---
title: redis单机版与三主三从搭建
date: 2019-12-26 19:13:47
tags:
---
Redis单机安装
创建一个目录
Cd /usr/local/
Mkdir redis
进入创建的目录
Cd /redis
下载新版本Redis
yum-y install wget
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
解压
tar -zxvf redis-5.0.3.tar.gz
进入解压之后的目录
cd redis-5.0.3
安装 gcc
yum -y install make gcc*
编译安装 (使用prefix 指定一个安装位置)
make && make install PREFIX=/usr/local/redis
修改配置文件
vim redis.conf

bind 0.0.0.0 #所有IP都可以访问
 daemonize yes # 守护进程模式开启 后台运行
 protected-mode no # 关闭保护模式
放行端口号
firewall-cmd --zone=public --add-port=6379/tcp --permanent 
firewall-cmd --reload
运行redis
/usr/local/redis/redis-5.0.3/src/redis-server /usr/local/redis/redis-5.0.3/redis.conf
查看redis是否启动成功
ps -aux | grep redis 或者 ps -ef | grep redis
连接
./usr/local/redis/redis-5.0.3/src/redis-cli
使用redis Desktop Manager 连接
关闭虚拟机防火墙
systemctl stop firewalld 
systemctl disable firewalld
Redis集群搭建
创建文件夹
mkdir -p /usr/local/redis/redis-cluster/
cd /usr/local/redis/redis-cluster/
mkdir 6380 6381 6382 6383 6384 6385
下载redis
Cd ../
 yum -y install wget
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
解压
tar -zxvf redis-5.0.3.tar.gz
安装gcc
yum -y install make gcc*

进入解压之后的目录
cd redis-5.0.3
编译安装 (使用prefix 指定一个安装位置)
make && make install PREFIX=/usr/local/redis
把redis.conf依次复制到各个文件夹下
cd ../
cp -r redis-5.0.3/redis.conf /usr/local/redis/redis-cluster/6380
cp -r redis-5.0.3/redis.conf /usr/local/redis/redis-cluster/6381
cp -r redis-5.0.3/redis.conf /usr/local/redis/redis-cluster/6382
cp -r redis-5.0.3/redis.conf /usr/local/redis/redis-cluster/6383
cp -r redis-5.0.3/redis.conf /usr/local/redis/redis-cluster/6384
cp -r redis-5.0.3/redis.conf /usr/local/redis/redis-cluster/6385
依次修改redis.conf
vim /usr/local/redis/redis-cluster/6380/redis.conf


bind 0.0.0.0  #默认绑定本地地址，导致其它地方不可远程访问 改成局域网中的IP地址或者0.0.0.0所有ip都可以访问
protected-mode no   #非保护模式
port 6380 #端口
daemonize yes # redis后台运行
pidfile /var/run/redis_6380.pid #需要修改为 reids_{port}.pid 的形式
logfile /var/log/redis_6380.log  #需要修改为 reids_{port}.pid 的形式
appendonly yes #开启AOF日志 指定持久化方式
cluster-enabled yes #开启集群
cluster-config-file nodes-6380.conf #集群的配置文件 nodes_{port}.conf的形式
cluster-node-timeout 5000 #超时时间
启动全部redis节点
/usr/local/redis/redis-5.0.3/src/redis-server   /usr/local/redis/redis-cluster/6380/redis.conf
/usr/local/redis/redis-5.0.3/src/redis-server   /usr/local/redis/redis-cluster/6381/redis.conf
/usr/local/redis/redis-5.0.3/src/redis-server   /usr/local/redis/redis-cluster/6382/redis.conf
/usr/local/redis/redis-5.0.3/src/redis-server   /usr/local/redis/redis-cluster/6383/redis.conf
/usr/local/redis/redis-5.0.3/src/redis-server   /usr/local/redis/redis-cluster/6384/redis.conf
/usr/local/redis/redis-5.0.3/src/redis-server   /usr/local/redis/redis-cluster/6385/redis.conf
放行端口号
firewall-cmd --zone=public --add-port=6381/tcp --permanent
firewall-cmd --zone=public --add-port=6382/tcp --permanent
firewall-cmd --zone=public --add-port=6383/tcp --permanent
firewall-cmd --zone=public --add-port=6384/tcp --permanent
firewall-cmd --zone=public --add-port=6385/tcp --permanent
firewall-cmd --reload
启动集群
/usr/local/redis/redis-5.0.3/src/redis-cli --cluster create 192.168.0.108:6380 192.168.0.108:6381 192.168.0.108:6382 192.168.0.108:6383 192.168.0.108:6384 192.168.0.108:6385 --cluster-replicas 1
查看redis是否启动成功
ps -ef | grep redis
连接
/usr/local/redis/bin/redis-cli -c -h 192.168.0.108 -p 6380
#然后可以输入下面的命令 
cluster info #打印集群的信息
cluster nodes #列出集群当前已知的所有节点(node)，以及这些节点的相关信息  