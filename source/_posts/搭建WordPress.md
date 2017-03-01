---
title: 搭建WordPress
date: 2016-09-02
tags:
---

1.搭建LAMP环境
<!--more-->
    service iptables stop
    setenforce 0
    yum install httpd mysql mysql-server php php-mysql php-gd php-xml
    service httpd start
    service mysqld start
    chkconfig httpd on //开机启动
    chkconfig –list |grep httpd
    chkconfig mysqld on
    chkconfig –list |grep mysql
    mysqladmin -u root -p password ‘123’ //为mysql设置用户和密码
    Enter password: //此处回车即可。
    mysql -u root -p
    create database wordpress; //创建wordpress数据库，为下面安装wordpress做准备。
    show databases;

2.安装WordPress

    unzip wordpress-3.9-zh_CN.zip //解压缩
    mv wordpress /var/www/html/
    cd /var/www/html/wordpress/
    vim wp-config.php