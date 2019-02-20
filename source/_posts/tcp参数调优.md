---
title: tcp参数调优
date: 2019-01-08 14:22:53
updated: 2019-01-08 14:22:53
categories: 系统
tags: linux-sysctl
---

1. /etc/sysctl.conf

```bash
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1024 65535
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_window_scaling = 0
net.ipv4.tcp_sack = 0
net.core.netdev_max_backlog = 30000
net.ipv4.tcp_no_metrics_save=1
net.core.somaxconn = 262144
net.ipv4.tcp_syncookies = 0
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2

fs.file-max = 1024000
net.nf_conntrack_max= 1024000
net.ipv4.tcp_mem=786432 2097152 3145728
```

2. ulimit -a 中open files (-n) 1024000 修改

   vi /etc/profile

   ulimit -n 1024000

   生效：source /etc/profile

   cat /proc/sys/fs/file-max

   修改：pending signals (-i) 128296

   vi /etc/profile

   ulimit -i 128296

   生效：source /etc/profile

3. error, emfile 最大用户进程需要在90-nproc.conf

   vi /etc/security/limits.conf

   soft nofile 65535

   hard nofile 65535

   然后，一般来说，修改ulimit的数值，只需要修改/etc/security/limits.conf即可，但是这个参数需要修改/etc/security/limits.d/90-nproc.conf。centos 6.*可以修改/etc/security/limits.d/90-nproc.conf，但centos 5.*并没有90-nproc.conf这个文件，我这边是通过修改/etc/security/limits.conf。

4. netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

   CLOSE_WAIT 162

   ESTABLISHED 10163

   SYN_RECV 242
