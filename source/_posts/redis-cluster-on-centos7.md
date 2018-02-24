---
title: centos7上redis的集群搭建
date: 2017-10-31 13:44:15
tags: redis
---

今天写一篇redis4.0.2集群搭建的日志以防遗忘,以下操作均在centos7上操作成功.

<!-- more -->

### 创建redis单机 

1. centos确认gcc安装成功. `yum install gcc`  
2. 下载redis-4.0.2.tar.gz源码至centos下,我的路径:`/usr/local/redis/`并解压.
3. `cd redis-4.0.2`,进行编译 `make`并等待完成.
4. 进入`src`目录进行安装 `make install`. 
5. 验证: 查看src目录下是否存在redis-server , redis-cli即可
6. 建立两个文件夹存放命令和配置文件`make -p /usr/local/redis/bin`,`make -p /usr/local/redis/etc`
7. 把redis-4.0.2文件夹下的`redis.conf`拷贝至移动至`etc`目录下 `cp /usr/local/redis/redis-4.0.2/redis.conf /usr/local/redis/etc/`
8. 把 `redis-benchmark`  `redis-check-aof` ` redis-check-rdb`  `redis-cli`  `redis-sentinel`  `redis-server`  `redis-trib.rb` 复制至 `bin`目录下. 命令略.
9. 编写`start.sh`,脚本内容如下即可  

```shell
#!/bin/sh
echo "shutdown"
/usr/local/redis/bin/redis-cli shutdown
echo "ready start"
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
echo "start"
sleep 1s  
ps aux | grep redis
```  
同时修改`redis.conf`中的 daemonize为yes(yes代表后台启动)  
执行脚本,查看是否成功,有
root      1472  0.0  0.7 145256  7548 ?        Ssl  14:18   0:00 /usr/local/redis/bin/redis-server 0.0.0.0:6379
类似输出说明启动成功.

### 创建redis集群

首先关闭刚刚窗户的redis单机客户端.执行`/usr/local/redis/bin/redis-cli shutdown`即可成功

**第一步**
1.在`/usr/local`下创建`redis-cluster`文件夹,并建立子目录7001~7006共6个文件夹. 
 
**第二步**
把之前的`redis.conf`拷贝至700*目录下,并逐个修改以下配置:
1. daemonzie yes
2. port 700*(分别对每个机器的端口号进行设置)
3. bind 192.168.1.102(当前机器的ip,我本地的ip,必须绑定,否则可能出现问题)
4. dir /usr/local/redis-cluster/700*/ (指定文件的存放位置,必须指定不同的目录,不然会丢失数据)
5. cluster-enable yes(启动集群模式)
6. cluster-config-file nodes-700*.conf (最好和的端口相对应)
7. cluster-node-timeout (集群节点超时时间5s)
8. appendonly yes (开启aof持久化)

**第三步**
把配置文件拷贝至对应的文件下,端口和nodes文件也要不相同.

**第四步**
redis集群需要ruby命令,所有先安装ruby和ruby的包管理器
1. `yum install ruby`
2. `yum install rubygems`
3. `yum install redis`(安装redis和ruby的接口,必须安装)

**第五步**
拷贝start.sh至700*目录下并修改启动配置项`/usr/local/redis/bin/redis-server /usr/local/redis-cluster/对应的端口目录/redis.conf` ,然后启动这6个实例并检查是否启动成功.

**第六步**
进入`redis/bin/`目录执行 `./redis-trib.rb create --replicas 1 192.168.1.102:7001 192.168.1.102:7002 192.168.1.102:7003 192.168.1.102:7004 192.168.1.102:7005 192.168.1.102:7006`

**第七步**
至此集群创建3主3从的集群,进行验证:
1. 连接任意客户端即可: `./redis-cli -c -h -p`(-c 表示集群, -h 表示host -p 表示端口) 如:`./redis-cli -c -h 192.168.1.102 -p 7001`
2. 进行验证 :cluster info(集群信息),  cluster nodes(查看节点列表)
![cluster info](/blog/2017/10/31/1.png) 

![cluster nodes](/blog/2017/10/31/2.png) 
发现集群已经建立.


### 新增/移除redis节点
首先了解`redis-trib`的命令参数,主要使用`create`,`reshard`,`add-node`,` del-node`命令
![cluster info](/blog/2017/10/31/3.png) 
1. create:创建一个集群环境host1:port1 ... hostN:portN(集群中的主从节点比例)
2. call:可以执行redis命令
3. add-node:将一个节点添加到集群里,第一个参数为新节点的ip:port,第二个参数为集群中任意一个已经存在的节点的ip:port
4. del-node:移除一个节点
5. reshard:重新分片
6. check:检查集群状态


**第一步**
我们新建俩个服务,按照之前搭建的集群方式新增俩个节点7007,7008并启动:(一主一从 master、slave) 
Master:7007 和 Slave:7008

**第二步**
新增一个主节点7007(master),使用add-node命令:
`./redis-trib.rb add-node 192.168.1.102:7007 192.168.1.102:7001`
第二个参数为任意已知的节点.可以看见执行成功的输出.
进入并查看节点信息:
![cluster nodes](/blog/2017/10/31/4.png) 

> 注意:当添加节点成功以后,新增的节点不会有任何数据,因为它没有分配任何的slot(hash槽).我们需要为新节点手工分配slot.


为7007分配slot槽.使用redis-trib命令,找到集群中的任意一个master主节点,对其进行重新分片工作.
`./redis-trib.rb reshard 192.168.1.102:7001`
执行后会等待3次输入提示

> redis-trib.rb reshard 192.168.10.219:6378 //下面是主要过程  
>How many slots do you want to move (from 1 to 16384)? 1000 //设置slot数1000  
>What is the receiving node ID? d81f30cbfffc2f7bfd943997194ff528e9a23d8d //新节点node id  
>Please enter all the source node IDs.  
> Type 'all' to use all the nodes as source nodes for the hash slots.  
> Type 'done' once you entered all the source nodes IDs.  
>Source node #1:all //表示全部节点重新洗牌  
>Do you want to proceed with the proposed reshard plan (yes/no)? yes //确认重新分  

![cluster nodes](/blog/2017/10/31/5.png) 
查看集群状态:如上图所示,现在我们的7007已经有slot槽了,也就是说可以在7007上进行读写数据啦!到此为止我们的7007已经加入到集群中啦,并且是主节点(Master)

**第三步**
添加从节点(7008)到集群中去,还是需要执行add-node命令
`./redis-trib.rb add-node 192.168.1.102:7008 192.168.1.102:7001`
执行后如同7007是一个master节点,没有被分配任何的slot槽.  

我们需要执行replicate命令来指定当前节点(从节点)的主节点id为哪个.
首先需要登录新加的7008节点的客户端,然后使用集群命令进行操作,把当前的7008(slave)节点指定到一个主节点下(这里使用之前创建的7007主节点)
`cluster replicate d81f30cbfffc2f7bfd943997194ff528e9a23d8d`(cluster replicate 指定master的id)
我们继续看一下当前集群的状态,如下图:我们已经成功的把7008放到7007这个主节点下面了,到此为止我们已经成功的添加完一个从节点了
![cluster nodes](/blog/2017/10/31/6.png) 

**第四步**
我们现在尝试删除一个节点(7008 slave), del-node命令,指定删除节点ip和端口,以及节点id
`./redis-trib.rb del-node 192.168.1.102:7008 1cd7c0d44001b160aaa15cb5ed7958bceac02bb7`
再次查看一下集群状态,发现我们已经成功的移除了7008 slave节点

**第五步**
最后,我们尝试删除之前加入的主节点7007,这个步骤会相对比较麻烦一些,因为主节点的里面是有分配了slot槽的,所以我们这里必须先把7007里的slot槽放入到其他的可用主节点中去,然后再进行移除节点操作才行,不然会出现数据丢失问题(如果主节点有从节点,将从节点转移到其他主节点如果主节点有slot,去掉分配的slot,然后在删除主节点)

>redis-trib.rb reshard 192.168.10.219:6378 //取消分配的slot,下面是主要过程   
>How many slots do you want to move (from 1 to 16384)? 1000 //被删除master的所有slot数量  
>What is the receiving node ID? 4ab0e50574bfad614343d3a7fd9ad8ad61df205c //接收6378节点slot的master  
>Please enter all the source node IDs.  
> Type 'all' to use all the nodes as source nodes for the hash slots.  
> Type 'done' once you entered all the source nodes IDs.  
>Source node #1:d81f30cbfffc2f7bfd943997194ff528e9a23d8d //被删除master的node-id  
>Source node #2:done   
>Do you want to proceed with the proposed reshard plan (yes/no)? yes //取消slot后,reshard  

最后我们直接使用del-node命令删除7007主节点即可
`redis-trib.rb del-node 192.168.1.102:7007 d81f30cbfffc2f7bfd943997194ff528e9a23d8d`

![cluster nodes](/blog/2017/10/31/7.png) 
最后:我们查看集群状态,一切还原为最初始状态啦!OK 结束!