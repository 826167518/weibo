---
title: yum仓库搭建之RPM包制作
date: 2016-11-05
tags:
---
 常见的软件安装方式有以下几种  
1.yum安装，可自动解决依赖，但不能自定义软件安装位置  
2.编译安装，可指定安装路径，指定装模块，但编译参数冗长，且耗时较长，不能解决依赖问题。  
<!--more-->
3.rpm安装，安装速度较速，但不能自动解决依赖，尤其是遇到需要的依赖包较多时，特别费时。  
本文主要介绍利用fpm工具制作个性化的rpm包，后期可放到yum仓库中，直接用yum安装。  
【fpm介绍】  
项目地址：https://github.com/jordansissel/fpm  
作者把这个fpm称作Effing Package Management，翻译过来就是该死的包管理器，粗暴一点就是去他妈的包管理器。Ubuntu及CentOS的包管理及安装方式完全不同，要想同时掌握这两种平台下的软件包安装方法是很困难的，为了不再遭受这痛苦，fpm便应运而生了。fpm是由jordansissel于2011年开发的一套打包工具，可快速度地将你安装好的程序目录或包打包为rpm及deb等结尾软件包。与传统的打包工具(rpmbuild、dh_make)相比，制作起来更加简单、方便、快捷。  
【fpm安装】  
1.安装ruby及gcc    

    yum install ruby-devel gcc  
2.安装fpm    

    gem install fpm  
3.fpm打包   
语法格式  

    fpm -s <source type> -t <target type> [options]  
其中源类型主要有：dir、gem、rpm、python等，目标类型主要有rpm,deb,puppet,solaris等。  
-s指定输入的包类型  
-t指定输出包的类型  
-n, --name指定输出的包名  
-v, --version指定版本号，默认为1.0  
-d, --depends指定依赖包，可重复多次出现，通常以"-d 'name' or -d 'name > version'"的形式展现。  
-f, --force强制输出，会覆盖掉旧包  
-p, --package OUTPUT 指定输出目录  
【打包实例】  
定制cron初始化rpm包    

    $fpm -s dir -t rpm -a noarch -p /root/ -n cron-init-script -v 1.0 /var/spool/cron/  
    no value for epoch is set, defaulting to nil {:level=>:warn}  
    no value for epoch is set, defaulting to nil {:level=>:warn}  
    Created package {:path=>"/root/cron-init-script-1.0-1.noarch.rpm"}  
    $ll /root/cron-init-script-1.0-1.noarch.rpm   
    -rw-r--r-- 1 root root 1693 Nov  2 22:24 /root/cron-init-script-1.0-1.noarch.rpm  
在客户端yum安装cron-init-script  

    yum install cron-init-scipt

【升级RPM包】  
1.编辑cron任务  

    $crontab -l  
    */5 * * * * /usr/sbin/ntpdate pool.ntp.org >/dev/null 2>&1  
    */10 * * * * /usr/sbin/ntpdate 1.pool.ntp.org >/dev/null 2>&1  

2.重新生成包
    
    fpm -s dir -t rpm -a noarch -p /tools/fpm/ -n cron-init-script -v 1.1 /var/spool/cron/

  

yum仓库搭建之RPM包制作  
3.传到yum仓库  

    $cp cron-init-script-1.1-1.noarch.rpm /application/yum/centos6.6/x86_64/ 

4.更新yum仓库索引  

    $createrepo --update /application/yum/centos6.6/x86_64/
    Spawning worker 0 with 1 pkgs  
    Workers Finished  
    Gathering worker results  
    Saving Primary metadata  
    Saving file lists metadata  
    Saving other metadata  
    Generating sqlite DBs  
    Sqlite DBs complete  
5.客户端清空yum缓存    

    ###yum clean all  
    Loaded plugins: fastestmirror, security  
    Cleaning repos: oldboy  
    Cleaning up Everything   
    Cleaning up list of fastest mirrors  
6.查找cron包  

    # yum list |grep cron-init  
    cron-init-script.noarch 1.0-1  @oldboy#前面的@表示已经安装过，保留下来的信息   
    cron-init-script.noarch 1.1-1  oldboy   
7.更新cron包  
    
    # crontab -l  
    */5 * * * * /usr/sbin/ntpdate pool.ntp.org >/dev/null 2>&1  
    # yum update cron-init-script  
    Is this ok [y/N]: y  
    Running Transaction  
      Updating  : cron-init-script-1.1-1.noarch1/2   
      Cleanup: cron-init-script-1.0-1.noarch2/2   
      Verifying  : cron-init-script-1.1-1.noarch1/2   
      Verifying  : cron-init-script-1.0-1.noarch2/2  
    Updated:  
      cron-init-script.noarch 0:1.1-1 
    Complete!  
    # crontab -l  
    */5 * * * * /usr/sbin/ntpdate pool.ntp.org >/dev/null 2>&1  
    */10 * * * * /usr/sbin/ntpdate 1.pool.ntp.org >/dev/null 2>&1  
cron任务已更新。  
    
    
    fpm -f -s dir -t rpm -n easemob-sersync-ssy -v 2.5.4_64bit  --iteration 1.el6.centos -a native \  
    -C /tmp/easemob-serync \  
    --vendor 'sam@easemob.com' \  
    --description 'Ejabberd packager by easemob.com' \  
    --url 'https://github.com/easemob/serync/' \  
    --rpm-user easemob \  
    --rpm-group easemob \  
    --verbose \  
    --epoch 20160616  
    
    
    
    createrepo /data/apps/data/nginx/yum/ssy/ssy201606/x86_64/6/easemob-ssy/packages/
    