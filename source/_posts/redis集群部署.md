---
title: redis集群部署
date: 2018-12-14 17:23:57
updated: 2018-12-14 17:23:57
categories: 组件
tags: redis
---

```javascript
sudo su
cd /tmp && wget http://msstest-corp.sankuai.com/v1/mss_hua9JwgTGEOH-+=lADhSBT0A==/publish/redis-3.2.8-1.el6.remi.x86_64.rpm
yum install -y redis-3.2.8-1.el6.remi.x86_64.rpm
su sankuai
mkdir -p /opt/meituan/apps/redis
cd /opt/meituan/apps/redis
## 6379
mkdir -p /opt/meituan/appdatas/redis6379
wget http://msstest-corp.sankuai.com/v1/mss_hua9JwgTGEOH-+=lADhSBT0A==/publish/redis_6379_3.conf -O 6379.conf
redis-server /opt/meituan/apps/redis/6379.conf
wget http://msstest-corp.sankuai.com/v1/mss_hua9JwgTGEOH-+=lADhSBT0A==/publish/redis-monitor.py
(crontab -l 2>/dev/null |grep -Fv "6379.conf"; echo "* * * * * python /opt/meituan/apps/redis/redis-monitor.py /opt/meituan/apps/redis/6379.conf > /dev/null 2>&1 &") | crontab -
## 6380
mkdir -p /opt/meituan/appdatas/redis6380
wget http://msstest-corp.sankuai.com/v1/mss_hua9JwgTGEOH-+=lADhSBT0A==/publish/redis_6380_3.conf -O 6380.conf
redis-server /opt/meituan/apps/redis/6380.conf
(crontab -l 2>/dev/null |grep -Fv "6380.conf"; echo "* * * * * python /opt/meituan/apps/redis/redis-monitor.py /opt/meituan/apps/redis/6380.conf > /dev/null 2>&1 &") | crontab -
## 6381
mkdir -p /opt/meituan/appdatas/redis6381
wget http://msstest-corp.sankuai.com/v1/mss_hua9JwgTGEOH-+=lADhSBT0A==/publish/redis_6381_3.conf -O 6381.conf
redis-server /opt/meituan/apps/redis/6381.conf
(crontab -l 2>/dev/null |grep -Fv "6381.conf"; echo "* * * * * python /opt/meituan/apps/redis/redis-monitor.py /opt/meituan/apps/redis/6381.conf > /dev/null 2>&1 &") | crontab -
exit
exit
```
