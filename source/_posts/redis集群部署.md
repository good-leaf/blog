---
title: redis集群部署
date: 2018-12-14 17:23:57
updated: 2018-12-14 17:23:57
categories: redis
tags: redis
---

```javascript
sudo su
cd /tmp && wget https://yangyajun-soft.github.io/redis/redis-3.2.8-1.el6.remi.x86_64.rpm
yum install -y redis-3.2.8-1.el6.remi.x86_64.rpm
su sankuai
mkdir -p /opt/xxx/apps/redis
cd /opt/meituan/apps/redis
## 6379
mkdir -p /opt/xxx/appdatas/redis6379
wget https://yangyajun-soft.github.io/redis/redis_6379_3.conf -O 6379.conf
redis-server /opt/meituan/apps/redis/6379.conf
## 6380
mkdir -p /opt/xxx/appdatas/redis6380
wget https://yangyajun-soft.github.io/redis/redis_6380_3.conf -O 6380.conf
## 6381
mkdir -p /opt/xxx/appdatas/redis6381
wget https://yangyajun-soft.github.io/redis/redis_6381_3.conf -O 6381.conf
exit
exit
```
