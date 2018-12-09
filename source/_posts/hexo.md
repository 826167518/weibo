---
title: 搭建Hexo
date: 2016-09-02
tags:
---
1、安装编译npm基础包
    rpm -Uvh http://download-i2.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
<!--more-->

    yum install nodejs npm --enablerepo=epel
部署 Hexo --- 安装
    npm install -g hexo
部署 Hexo --- 初始化
    mkdir /home/wwwroot && hexo init /home/wwwroot