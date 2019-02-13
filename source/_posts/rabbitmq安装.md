---
    title: rabbitmq安装
    date: 2018-12-14 17:23:57
    updated: 2018-12-14 17:23:57
    categories: rabbitmq
    tags: rabbitmq
---

版本：rabbitmq-server-3.6.6-1.el6.noarch.rpm

<!--more-->

```c
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
