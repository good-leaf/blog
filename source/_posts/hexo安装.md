---
title: hexo 安装
date: 2018-12-14 17:23:57
updated: 2018-12-14 17:23:57
categories: hexo
tags: hexo
---

## git设置
```
git config --global user.name "good-leaf"
git config --global user.email "rwzgnyyj@xxx.com"
```

## hexo安装
```
npm install -g hexo-cli
cd
hexo init blog
cd blog
npm install
hexo server
npm install --save hexo-deployer-git
```

## hexo配置
添加git地址：使用ssh时，需要将本机ssh key添加到github上，并且选择ssh访问方式。
```
deploy:
  type: git
  repo: git@github.com:good-leaf/good-leaf.github.io.git
  branch: master
```

修改端口：vi _config.yml
```
server:
  port: 4001
  compress: true
  header: true
```

搜索支持：
```
npm install hexo-generator-searchdb --save

search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

## 更换主题：访问页面显示
```
extends partial/layout

block container
    include mixins/post
    +posts()

block pagination
    include mixins/paginator
    +home()

block copyright
    include partial/copyright
```
解决：npm install --save hexo-renderer-jade

