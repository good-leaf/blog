---
title: rabbitmq安装
date: 2018-12-14 17:23:57
updated: 2018-12-14 17:23:57
categories: 开源框架
tags: rabbitmq安装
---

版本：rabbitmq-server-3.6.6-1.el6.noarch.rpm

<!--more-->

### 安装脚本
```bash
sudo su
cd /tmp && wget https://yangyajun-soft.github.io/rabbitmq3.6.6/rabbitmq-server-3.6.6-1.el6.noarch.rpm
yum install -y rabbitmq-server-3.6.6-1.el6.noarch.rpm
chown rabbitmq:rabbitmq /etc/rabbitmq
exit
# set cookie
sudo su rabbitmq
echo cookie > /var/lib/rabbitmq/.erlang.cookie
chmod u=r,g=,o= /var/lib/rabbitmq/.erlang.cookie
wget https://yangyajun-soft.github.io//rabbitmq3.6.6/rabbitmq.config -O /etc/rabbitmq/rabbitmq.config
# start
rabbitmq-server -detached
# join cluster
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@host
rabbitmqctl start_app
# config
rabbitmqctl set_policy ha-backup "^" '{"ha-mode":"exactly", "ha-params":2, "ha-sync-mode":"automatic"}'
rabbitmq-plugins enable rabbitmq_management
# add admin
rabbitmqctl add_user admin password
rabbitmqctl set_permissions -p "/" admin '.*' '.*' '.*'
rabbitmqctl set_user_tags admin administrator
# add user
rabbitmqctl add_user user password
rabbitmqctl set_permissions -p "/" user '.*' '.*' '.*'
```
### 配置文件
- enabled_plugins
```bash
[rabbitmq_management,rabbitmq_tracing].
```
- rabbitmq.config
```bash
[
{rabbit, [{disk_free_limit, 5242880}
          ,{vm_memory_high_watermark, 0.8}
          ,{loopback_users, []}
         ]},
{rabbitmq_management, [{listener, [{port, 8080}]}]}
{rabbitmq_management_agent, [ {force_fine_statistics, false} ] }
].
```
- 
