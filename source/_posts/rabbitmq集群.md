---
title: rabbitmq集群
date: 2019-02-20 14:38:10
categories: 开源框架
tags: rabbitmq集群
---

### 集群
RabbitMQ集群中可以共享user，virtualhosts，queues，exchanges等。但message只会在创建的节点上传输。当message进入A节点的queue中后，consumer从B节点拉取时，RabbitMQ会临时在A、B间进行消息传输，把A中的消息实体取出并经过B发送给consumer。所以consumer应尽量连接每一个节点，从中取消息。
RABBITMQ的集群节点包括内存节点、磁盘节点。内存节点的元数据仅放在内存中，性能比磁盘节点会有所提升。不过，如果在投递message时，打开了message的持久化，那么内存节点的性能只能体现在资源管理上，比如增加或删除队列（queue），虚拟主机（vrtual hosts），交换机（exchange）等，发送和接受message速度同磁盘节点一样。一个集群至少要有一个磁盘节点。
<!--more-->

### 普通集群

- cookie同步

  同步/var/lib/rabbitmq/erlang.cookie，本文件默认权限400。

- 加入集群

  ```bash
  rabbitmqctl stop_app
  rabbitmqctl join_cluster rabbit@tabbitmq1
  rabbitmqctl start_app
  ```

- 查看集群信息

  ```bash
  rabbitmqctl cluster_status
  ```

- 更改节点属性

  ```bash
  rabbitmqctl stop_app
  rabbitmqctl change_cluster_node_type disc/ram
  rabbitmqctl start_app
  ```

- 节点退出集群

  ```bash
  退出节点服务执行：
  rabbitmqctl stop_app
  rabbitmqctl reset
  rabbitmqctl start_app
  集群主节点执行：
  rabbitmqctl forget_cluster_node rabbit@rabbitmq2
  ```

- rabbitmq集群重启

  集群重启时，最后一个挂掉的节点应该第一个重启，如果因特殊原因（比如同时断电），而不知道哪个节点最后一个挂掉。可用以下方法重启：

  ```bash
  rabbitmqctl force_boot
  service rabbitmq-server start
  在其他节点上执行
  service rabbitmq-server start
  查看cluster状态是否正常（要在所有节点上查询）。
  rabbitmqctl cluster_status
  ```

  如果有节点没加入集群，可以先退出集群，然后再重新加入集群。上述方法不适合内存节点重启，内存节点重启的时候是会去磁盘节点同步数据，如果磁盘节点没起来，内存节点一直失败。

### 镜像队列

镜像队列可以同步queue和message，当主queue挂掉，从queue中会有一个变为主queue来接替工作。

镜像队列是基于普通的集群模式的,所以你还是得先配置普通集群,然后才能设置镜像队列。

镜像队列设置后，会分一个主节点和多个从节点，如果主节点宕机，从节点会有一个选为主节点，原先的主节点起来后会变为从节点。

queue和message虽然会存在所有镜像队列中，但客户端读取时不论物理面连接的主节点还是从节点，都是从主节点读取数据，然后主节点再将queue和message的状态同步给从节点，因此多个客户端连接不同的镜像队列不会产生同一message被多次接受的情况。

#rabbitmqctl set_policy  ha-all “hello” ‘{“ha-mode”:”all”}’

ha-all 是同步模式，指同步给所有节点，还有另外两种模式ha-exactly表示在指定个数的节点上进行镜像，节点的个数由ha-params指定，ha-nodes表示在指定的节点上进行镜像，节点名称通过ha-params指定；

hello 是同步的队列名，可以用正则表达式匹配；

{“ha-mode”:”all”} 表示同步给所有，同步模式的不同，此参数也不同。
