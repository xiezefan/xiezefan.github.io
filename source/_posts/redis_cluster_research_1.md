---
layout: post
title: "Redis Cluster 初探(1) - 集群搭建与扩容"
date: 2015-12-03
categories:
- Database
tags:
- redis
---

Redis Cluster是Redis官方的集群实现方案，在此之前已经有一些民间的第三方Redis集群解决方案，如Twitter的Twenproxy，豌豆荚的Codis，与其不同的是，Redis Cluster并非使用Porxy的模式来连接集群节点，而是使用无中心节点的模式来组建集群，有一定性能优势也有缺点，本文主要是我调研Redis Cluster的一些知识整理与经验汇总。

<!-- more -->

首先我们来尝试下搭建一个Redis Cluster集群

#### 前置准备

Redis Cluster需要Redis 3.0及以上版本才支持，此文发布的时候，Redis的最高版本为3.0.5。

```shell
wget http://download.redis.io/releases/redis-3.0.5.tar.gz
tar -xvf redis-3.0.5.tar.gz
cd redis-3.0.5
make
```

编译完Redis，生成的可执行文件在`redis-3.0.5/src`之中，为了方便使用，我们把可执行文件的目录加入PATH之中。

```shell
vim ~/.bashrc

# 增加以下内容

REDIS_HOME=/home/xiezefan/sofeware/redis-3.0.5/src
PATH=$REDIS_HOME:$PATH
export PATH

# 保存后让修改生效

source ~/.bashrc
```

要创建Redis Cluster，我们还需要安装Ruby以及RubyGems。

```shell
yum install ruby
yum install gcc g++ make automake autoconf curl-devel openssl-devel zlib-devel httpd-devel apr-devel apr-util-devel sqlite-devel
yum install ruby-rdoc ruby-devel
yum install rubygems
gem install redis
```

#### 创建集群

此次此时我们需要创建8个节点，端口号7000~7007

复制`redis-3.0.5/redis.conf`，修改一下内容

```shell
mkdir cluster
cp ~/sofeware/redis-3.0.5/redis.conf cluster/
vim cluster/redis.conf
# 修改以下内容
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

批量复制7份，修改配置文件的端口号。修改完毕后，分别进入各个节点的目录中启动redis

```shell
cd 7000
redis-server redis.conf

cd 7001
redis-server redis.conf

# 依次启动7000-7007
...

```

至此，8个Redis全都启动完毕，但是他们还处于彼此互相不知道彼此的阶段。

```shell
xiezefan@ubuntu:~$ ps -ef | grep redis
xiezefan 13372     1  0 20:09 ?        00:00:08 redis-server *:7000 [cluster]
xiezefan 13376     1  0 20:09 ?        00:00:08 redis-server *:7001 [cluster]
xiezefan 13380     1  0 20:09 ?        00:00:08 redis-server *:7002 [cluster]
xiezefan 13382     1  0 20:09 ?        00:00:08 redis-server *:7003 [cluster]
xiezefan 13386     1  0 20:09 ?        00:00:08 redis-server *:7004 [cluster]
xiezefan 13390     1  0 20:09 ?        00:00:08 redis-server *:7005 [cluster]
xiezefan 13394     1  0 20:09 ?        00:00:08 redis-server *:7006 [cluster]
xiezefan 13400     1  0 20:09 ?        00:00:08 redis-server *:7007 [cluster]
```

下一步，我们要将7000-7005这六个节点连接成一个集群。

```shell
redis-trib.rb create --replicas 1 10.211.55.4:7000 10.211.55.4:7001 10.211.55.4:7002 10.211.55.4:7003 10.211.55.4:7004 10.211.55.4:7005
```

该命令表示，将7000-7006节点创建一个集群，冗余为1，就是3主3从。切记，此处指定的IP会在client发送move命令的时候返回，所以一定要指定为客户端能访问到的IP，例如下面这种IP是不可行的，client拿到的IP就位是127.0.0.1导致一直重定向失败。

```shell
redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

输入后，redis-trib自动分配给出一个slot的分配方案

```shell
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
10.211.55.4:7000
10.211.55.4:7001
10.211.55.4:7002
Adding replica 10.211.55.4:7003 to 10.211.55.4:7000
Adding replica 10.211.55.4:7004 to 10.211.55.4:7001
Adding replica 10.211.55.4:7005 to 10.211.55.4:7002
M: 9dfef549f7917794cbabaf96781ed0e19957c1f3 10.211.55.4:7000
   slots:0-5460 (5461 slots) master
M: c14485e8d7f1f3ec5c505b41fe727b657c951d8d 10.211.55.4:7001
   slots:5461-10922 (5462 slots) master
M: 8b308093e99f4299b8c18ab1dd81c5a83a3528c6 10.211.55.4:7002
   slots:10923-16383 (5461 slots) master
S: 632409883570eb5cecf6089583fba64a41d1154f 10.211.55.4:7003
   replicates 9dfef549f7917794cbabaf96781ed0e19957c1f3
S: edd31196e2980360b0738c57af95e1a69b0f9c9b 10.211.55.4:7004
   replicates c14485e8d7f1f3ec5c505b41fe727b657c951d8d
S: 95063a99c5cf2cc4abc51ca1e8ff0f3b8d60271c 10.211.55.4:7005
   replicates 8b308093e99f4299b8c18ab1dd81c5a83a3528c6
Can I set the above configuration? (type 'yes' to accept):
```
输入yes后，集群自动创建，创建完毕后，通过redis-cli进入任一节点，使用`cluster nodes`命令可以查看各个节点的状态

```shell
xiezefan@ubuntu:~$ redis-cli -p 7000
127.0.0.1:7000>cluster nodes
c14485e8d7f1f3ec5c505b41fe727b657c951d8d 10.211.55.4:7001 master - 0 1449060968280 2 connected 5461-10922
95063a99c5cf2cc4abc51ca1e8ff0f3b8d60271c 10.211.55.4:7005 slave 8b308093e99f4299b8c18ab1dd81c5a83a3528c6 0 1449060968280 6 connected
edd31196e2980360b0738c57af95e1a69b0f9c9b 10.211.55.4:7004 slave c14485e8d7f1f3ec5c505b41fe727b657c951d8d 0 1449060967274 5 connected
8b308093e99f4299b8c18ab1dd81c5a83a3528c6 10.211.55.4:7002 master - 0 1449060966769 3 connected 10923-16383
632409883570eb5cecf6089583fba64a41d1154f 10.211.55.4:7003 slave 9dfef549f7917794cbabaf96781ed0e19957c1f3 0 1449060967274 4 connected
9dfef549f7917794cbabaf96781ed0e19957c1f3 10.211.55.4:7000 myself,master - 0 0 1 connected 0-5460
```

随便输入一个查询指令`get user.1`

```shell
127.0.0.1:7000> get user.1
(error) MOVED 9645 10.211.55.4:7001
```
因为user.1所在的solt-9645在7001节点上，cluster给你发送MOVED指令让你去7001节点查找数据，连接redis-cli的时候，使用`-c`参数可以指定查询时接收到MOVED指令自动跳转

```shell
xiezefan@ubuntu:~$ redis-cli -c -p 7000
127.0.0.1:7000> get user.1
-> Redirected to slot [9645] located at 10.211.55.4:7001
(nil)
```

#### 集群扩容 

现在我们已经有一个包含6个节点的集群，我写了段[代码](https://gist.github.com/xiezefan/4bd5e0d0c264aadaf061)，往集群写入10W条测试数据。
现在模拟机器扩容场景，为集群加入一个master节点7006和一个slave节点7007。

```
redis-trib.rb add-node 10.211.55.4:7006 10.211.55.4:7000
```

以上命令将7006节点接入7000所在的集群。接下来，我们为7006增加一个slave节点。  

```
redis-trib.rb add-node --slave 10.211.55.4:7007 10.211.55.4:7000
```
  
以上命令表示增加slave节点，将7006的节点加入7000节点所在的集群中作为slave节点，随机依附现有的master节点中slave最少的节点，如果需要再指定特别的master节点，使用

```shell
redis-trib.rb add-node --slave --master-id 23b412673af0506df6382353e3a65960d5b7e66d 10.211.55.4:7007 10.211.55.4:7000
```
其中的`23b412673af0506df6382353e3a65960d5b7e66d`是7006节点的id，我们可以通过`cluster nodes`命令查看节点的id。

接下来我们用坐负载均衡，Slot是Redis Cluster数据承载的最小单位，我们可以指定将一定范围的Slot转移到新的节点来实现负载均衡。

Redis Cluster转移一个Slot的步骤是:

1. 在目标节点上声明将从源节点上迁入Slot `CLUSTER SETSLOT <slot> IMPORTING <source_node_id>`
2. 在源节点上声明将往目标节点迁出Slot `CLUSTER SETSLOT <slot> IMPORTING <source_node_id>`
3. 批量从源节点获取KEY `CLUSTER GETKEYSINSLOT <slot> <count>`
4. 将获取的Key迁移到目标节点 `MIGRATE <target_ip> <target_port> <key_name> 0 <timeout>`
5. 重复步骤3，4直到所有数据迁移完毕
6. 分别向双方节点发送 `CLUSTER SETSLOT <slot> NODE <target_node_id>`，该命令将会广播给集群其他节点，已经将Slot转移到目标节点。
7. 等待集群状态变为OK `CLUSTER INFO` 中的 `cluster_state = ok`

我编写了一个脚本来批量迁移Slot

```shell
#!/bin/bash
source_host=$1  # 源节点HOST
source_port=$2  # 源节点端口
target_host=$3  # 目标节点HOST
target_port=$4  # 目标节点端口
start_slot=$5   # 迁移节点的其实范围
end_slot=$6     # 迁移节点的结束范围


for slot in `seq ${start_slot} ${end_slot}`
do
	redis-cli -c -h ${target_host} -p ${target_port} cluster setslot ${slot} importing `redis-cli -c -h ${source_host} -p ${source_port} cluster nodes | grep ${source_port} | awk '{print $1}'`
	echo "Setslot importing ${slot} to ${target_host}:${target_port} success"
	redis-cli -c -h ${source_host} -p ${source_port} cluster setslot ${slot} migrating `redis-cli -c -h ${target_host} -p ${target_port} cluster nodes | grep ${target_port} | awk '{print $1}'`
	echo "Setslot migrating ${slot} from ${source_host}:${source_port} success"

	while [ 1 -eq 1 ]
	do
		allkeys=`redis-cli -c -h ${source_host} -p ${source_port} cluster getkeysinslot ${slot} 10`
		if [ -z "${allkeys}" ]
		then
			redis-cli -c -h ${source_host} -p ${source_port} cluster setslot ${slot} node `redis-cli -c -h ${target_host} -p ${target_port} cluster nodes | grep ${target_port} | awk '{print $1}'`
			redis-cli -c -h ${target_host} -p ${target_port} cluster setslot ${slot} node `redis-cli -c -h ${source_host} -p ${target_port} cluster nodes | grep ${target_port} | awk '{print $1}'`
			echo "Migrate slot ${slot} finish"
			break
		else 
			for key in ${allkeys}
			do
				redis-cli -c -h ${source_host} -p ${source_port} migrate ${target_host} ${target_port} ${key} 0 5000
				echo "Migrate slot ${slot} key ${key} success"
			done
		fi
	done
done

```
执行命令 `bash rebalance-cluster.sh 10.211.55.4 7000 10.211.55.4 7006 0 1000` 将7000节点上0-1000这个范围内的Slot转移到7006节点，通过`cluster nodes`命令我们可以看到0-1000这个区间是slot已经从7000转移到7006

```shell
xiezefan@ubuntu:~/sheel$ redis-cli -c -p 7000 cluster nodes
23b412673af0506df6382353e3a65960d5b7e66d 10.211.55.4:7006 master - 0 1449064402389 7 connected 0-1000
0c2954d21d7bcdae333f4fdecf468ce05aa25544 10.211.55.4:7001 master - 0 1449064400372 2 connected 5461-10922
384a3bb5bd9ecb2fc7db75c866abc715d7966f82 10.211.55.4:7002 master - 0 1449064401381 3 connected 10923-16383
c3e09d286ef2dce49843268b20832d65a5d516a1 10.211.55.4:7004 slave 0c2954d21d7bcdae333f4fdecf468ce05aa25544 0 1449064401885 5 connected
50737b4a91443ab1a34eec4ef99d4f6fe5d358f4 10.211.55.4:7005 slave 384a3bb5bd9ecb2fc7db75c866abc715d7966f82 0 1449064402389 6 connected
3c62cc6664bba378cceb8ae8e02f5d727deafe9d 10.211.55.4:7007 slave 23b412673af0506df6382353e3a65960d5b7e66d 0 1449064400878 7 connected
d6441916dcd89cbf431465d92dfc0eb3dd235295 10.211.55.4:7003 slave 6ee21c5d93a6d2f293a2df1b37e8c9c27cb55ad8 0 1449064402389 4 connected
6ee21c5d93a6d2f293a2df1b37e8c9c27cb55ad8 10.211.55.4:7000 myself,master - 0 0 1 connected 1001-5460
```


### 参考文章

* [全面剖析Redis Cluster原理和应用](http://blog.csdn.net/dc_726/article/details/48552531)
* [Redis集群功能预览](http://blog.csdn.net/dc_726/article/details/43991905)
* [Redis-Cluster实战--8.Redis-Cluster水平扩容(redis-cli实现版)](http://carlosfu.iteye.com/blog/2243056)
* [谈谈陌陌争霸在数据库方面踩过的坑( Redis 篇)](http://blog.codingnow.com/2014/03/mmzb_redis.html)

