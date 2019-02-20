---
title: java安装
date: 2019-02-13 17:23:39
categories: 开发语言
tags: java
---

1. java 1.8 安装

   yum install yum install java-1.8.0-openjdk-devel.x86_64

2. java 环境变量

   CLASSPATH中的tools.jar主要包含一些工具，如javac（将.java编译为.class）、javadoc（根据java源文件以html格式生成API文档）、javap（反汇编.class文件）等；

   ```bash
   PATH=$PATH:$HOME/bin
   JAVA_HOME=/usr/lib/jvm/java-1.8.0
   CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
   PATH=$JAVA_HOME/bin:$HOME/bin:$HOME/.local/bin:$PATH
   ```

   <!--more-->

3. java 1.8 安装

   yum install yum install java-1.8.0-openjdk-devel.x86_64

4. java 环境变量

   CLASSPATH中的tools.jar主要包含一些工具，如javac（将.java编译为.class）、javadoc（根据java源文件以html格式生成API文档）、javap（反汇编.class文件）等；

   ```bash
   PATH=$PATH:$HOME/bin
   JAVA_HOME=/usr/lib/jvm/java-1.8.0
   CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
   PATH=$JAVA_HOME/bin:$HOME/bin:$HOME/.local/bin:$PATH
   ```

5. java 安装目录信息

   dt.jar中包含了关于swing的控件对应的图标和BeanInfo.class

   ```bash
   /usr/lib/jvm-exports/java
   /usr/lib/java
   /usr/lib/jvm/java
   /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/jre/bin/java
   /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/bin/java
   /usr/share/java
   /usr/bin/java
   /var/lib/alternatives/java
   /etc/pki/java
   /etc/pki/ca-trust/extracted/java
   /etc/java
   /etc/alternatives/java
   ```
