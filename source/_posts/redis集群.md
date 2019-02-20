---

title: redis集群

date: 2019-02-20 16:28:10

categories: 开源框架
tags: redis集群

---

redis 单点、redis主从、redis哨兵sentinel、redis集群cluster配置搭建。

<!--more-->

### redis单点

编辑redis.conf

```bash
port 6379
daemonize yes   指定后台运行
```

配置说明：

```bash
   daemonize：如需要在后台运行，把该项的值改为yes
   pdifile：把pid文件放在/var/run/redis.pid，可以配置到其他地址
　 bind：指定redis只接收来自该IP的请求，如果不设置，那么将处理所有请求，在生产环节中最好设置该项
　 port：监听端口，默认为6379
　 timeout：设置客户端连接时的超时时间，单位为秒
　 loglevel：等级分为4级，debug，revbose，notice和warning。生产环境下一般开启notice
　 logfile：配置log文件地址，默认使用标准输出，即打印在命令行终端的端口上
　 database：设置数据库的个数，默认使用的数据库是0
　 save：设置redis进行数据库镜像的频率
　 rdbcompression：在进行镜像备份时，是否进行压缩
   dbfilename：镜像备份文件的文件名
　 dir：数据库镜像备份的文件放置的路径
　 slaveof：设置该数据库为其他数据库的从数据库
　 masterauth：当主数据库连接需要密码验证时，在这里设定
　 requirepass：设置客户端连接后进行任何其他指定前需要使用的密码
　 maxclients：限制同时连接的客户端数量
　 maxmemory：设置redis能够使用的最大内存
　 appendonly：开启appendonly模式后，redis会把每一次所接收到的写操作都追加到appendonly.aof文件中，当redis重新启动时，会从该文件恢复出之前的状态
　 appendfsync：设置appendonly.aof文件进行同步的频率
　 vm_enabled：是否开启虚拟内存支持
　 vm_swap_file：设置虚拟内存的交换文件的路径
　 vm_max_momery：设置开启虚拟内存后，redis将使用的最大物理内存的大小，默认为0
　 vm_page_size：设置虚拟内存页的大小
　 vm_pages：设置交换文件的总的page数量
　 vm_max_thrrads：设置vm IO同时使用的线程数量
```

### redis主从

```bash
mkdir redis-master-slave
cp path/to/redis/conf/redis.conf path/to/redis-master-slave master.conf
cp path/to/redis/conf/redis.conf path/to/redis-master-slave slave.conf

## master.conf
port 6379
## slave.conf
port 6380
slaveof 127.0.0.1 6379
```

启动主从redis，打开两个命令窗口执行info

```bash
# Replication
role:master

# Replication
role:slave
master_host:127.0.0.1
master_port:6379
```

主节点set数据后，可以在从节点get数据，但是在从节点不可以set

### 哨兵sentinel

上面我们介绍了主从，从库作为一个“傀儡”，可以在需要的时候“顶上来”，”接盘“。我们配置的主从是为了”有备无患“，在主redis挂了之后，可以立马切换到从redis上，可能只需要花几分钟的时间，但是仍然是需要人为操作。这个时候redis sentinel 就派上用场了。sentinel 通常翻译成哨兵，就是放哨的，这里它就是用来监控主从节点的健康情况。客户端连接redis主从的时候，先连接 sentinel，sentinel会告诉客户端主redis的地址是多少，然后客户端连接上redis并进行后续的操作。当主节点挂掉的时候，客户端就得不到连接了因而报错了，客户端重新想sentinel询问主master的地址，然后客户端得到了新选举出来的主redis，然后又可以愉快的操作了。

为了说明sentinel的用处，我们做个试验。配置3个redis（1主2从），1个哨兵。步骤如下：

```bash
mkdir redis-sentinel
cd redis-sentinel
cp redis/path/conf/redis.conf path/to/redis-sentinel/redis01.conf
cp redis/path/conf/redis.conf path/to/redis-sentinel/redis02.conf
cp redis/path/conf/redis.conf path/to/redis-sentinel/redis03.conf
touch sentinel.conf
```

上我们创建了 3个redis配置文件，1个哨兵配置文件。我们将 redis01设置为master,将redis02，redis03设置为slave。

```bash
vim redis01.conf
port 63791

vim redis02.conf
port 63792
slaveof 127.0.0.1 63791

vim redis03.conf
port 63793
slaveof 127.0.0.1 63791

vim sentinel.conf
daemonize yes
port 26379
sentinel monitor mymaster 127.0.0.1 63791 1  
```

哨兵配置：

```bash
port 26379
protected-mode no
pidfile "/usr/local/redis/var/redis-sentinel.pid"
dir "/usr/local/redis/data/sentinel"
daemonize yes
logfile "/usr/local/redis/var/redis-sentinel.log"
sentinel monitor mymaster 127.0.0.1 63791 1
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 18000
sentinel auth-pass mymaster admin
sentinel failover-timeout mymaster 18000
sentinel auth-pass mymaster admin
```

mymaster 为主节点名字，可以随便取，后面程序里边连接的时候要用到 127.0.0.1 63793 为主节点的 ip,port 1 后面的数字 1 表示选举主节点的时候，投票数。1表示有一个sentinel同意即可升级为master。

主节点宕机模拟：

```bash
初始情况下，1主2从
# +monitor master mymaster 127.0.0.1 63791 quorum 1
* +slave slave 127.0.0.1:63792 127.0.0.1 63792 @ mymaster 127.0.0.1 63791
* +slave slave 127.0.0.1:63793 127.0.0.1 63793 @ mymaster 127.0.0.1 63791
发现主挂了，准备 故障转移
# +try-failover master mymaster 127.0.0.1 63791
将主切换到了 63793 即redis03 
# +switch-master mymaster 127.0.0.1 63791 127.0.0.1 63793
```

切换异常：30367:X 17 Oct 13:24:11.578 # -failover-abort-not-elected master mymaster 127.0.0.1 63793

解决：如果redis.conf配置中配置了

protected-mode yes

bind 192.168.98.136

则需要在sentinel 配置文件加上protected-mode no

否则在sentinel 配置文件加上

protected-mode yes  

bind 192.168.98.136

### redis cluster

- 单个redis并发有限

- 单个redis内存有限，内存太大导致rdb文件过大，同步回复数据会很慢。

所有，我们需要redis cluster 即redis集群。

Redis 集群是一个提供在**多个Redis间节点间共享数据**的程序集。

Redis 集群并不支持处理多个keys的命令,因为这需要在不同的节点间移动数据,从而达不到像Redis那样的性能,在高负载的情况下可能会导致不可预料的错误.

Redis 集群通过分区来提供**一定程度的可用性**,在实际环境中当某个节点宕机或者不可达的情况下继续处理命令. Redis 集群的优势:

- 自动分割数据到不同的节点上。

- 整个集群的部分节点失败或者不可达的情况下能够继续处理命令。

因为最小的redis集群，需要至少3个主节点，既然有3个主节点，而一个主节点搭配至少一个从节点，因此至少得6台redis。

创建redis集群：

```bash
redis-5.0.3/src/redis-cli --cluster create 127.0.0.1:6371 127.0.0.1:6372 127.0.0.1:6373 127.0.0.1:6374 127.0.0.1:6375 127.0.0.1:6376 --cluster-replicas 1
```

集群管理界面：Relumin
