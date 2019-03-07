---
title: erlang19.3
date: 2019-03-06 17:11:20
updated: 2019-03-06 17:11:20
categories: 开发语言
tags: erlang

---

### erlang19.3安装

- ./configure  --without-javac

  ```bash
  configure: error: No curses library functions found
  yum install ncurses-devel
  
  No usable OpenSSL found
  yum install openssl-devel
  ```

- 根据提示安装

  yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel unixODBC-devel

- 安装路径

  ```bash
  [root@iZ2ze9z5o9dc900zyvux6aZ otp_src_19.3]# find / -name erlang
  /root/otp_src_19.3/lib/jinterface/java_src/com/ericsson/otp/erlang
  /root/blog/public/tags/erlang
  /root/blog/.deploy_git/tags/erlang
  /usr/local/lib/erlang
  /var/www/html/public/tags/erlang
  ```
