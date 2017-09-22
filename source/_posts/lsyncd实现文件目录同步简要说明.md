---
title: centos 7 lsyncd实时同步
date: 2017-09-22
tags:
---
## lsyncd实时同步
### lsyncd简介
<!--more-->
Lysncd 实际上是lua语言封装了 inotify 和 rsync 工具，采用了 Linux 内核（2.6.13 及以后）里的 inotify 触发机制，然后通过rsync去差异同步，达到实时的效果。我认为它最令人称道的特性是，完美解决了 inotify + rsync海量文件同步带来的文件频繁发送文件列表的问题 —— 通过时间延迟或累计触发事件次数实现。另外，它的配置方式很简单，lua本身就是一种配置语言，可读性非常强。lsyncd也有多种工作模式可以选择，本地目录cp，本地目录rsync，远程目录rsyncssh。
实现简单高效的本地目录同步备份（网络存储挂载也当作本地目录），一个命令搞定。



### 首先客户端和服务端都需要安装rsync
	yum -y install rsync

### lsyncd的安装

	rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
	yum install lsyncd

### 创建lsyncd的工作目录
	mkdir -p /usr/local/lsyncd-2.1.5/etc/
	mkdir -p /usr/local/lsyncd-2.1.5/var/

### 创建lsyncd的配置文件
	vim /usr/local/lsyncd-2.1.5/etc/lsyncd.conf
	settings {
		logfile      = "/usr/local/lsyncd-2.1.5/var/lsyncd.log",
		statusFile   = "/usr/local/lsyncd-2.1.5/var/lsyncd.status",
		inotifyMode  = "CloseWrite",
		maxProcesses = 7,
    }


	sync {
		default.rsync,  
		source    = "/var/opt/gitlab/backups",  
		target    = "root@10.1.10.101:/var/opt/gitlab/  backups-BF",  
		maxDelays = 5,  
		delay = 30,  
		rsync     = {  
			binary = "/bin/rsync",  
            archive = true,
     	    compress = true,
     	    bwlimit   = 2000
		}
	}

### lsyncd.conf配置文件说明

#### settings

里面是全局配置，-开头表示注释

	>logfile 定义日志文件
	>stausFile 定义状态文件
	>inotifyMode 指定inotify监控的事件，默认是CloseWrite，还可以是Modify或CloseWrite or Modify
	>maxProcesses 同步进程的最大个数。假如同时有20个文件需要同步，而>maxProcesses = 7，则最大能看到有7个rysnc进程

#### sync

里面是定义同步参数，可以继续使用maxDelays来重写settings的全局变量。一般第一个参数指定lsyncd以什么模式运行：rsync、rsyncssh、direct三种模式：

	>default.rsync ：本地目录间同步，使用rsync，也可以达到使用ssh形式的远程rsync效果，或daemon方式连接远程rsyncd进程；
	>default.direct ：本地目录间同步，使用cp、rm等命令完成差异文件备份；
	>default.rsyncssh ：同步到远程主机目录，rsync的ssh模式，需要使用key来认证

	source 同步的源目录，使用绝对路径。
	arget 定义目的地址
	

#### rsync
	binary 定义rsync位置，位置有可能不一样

### 做公私钥认证 
将服务的的公钥写到客户端的`root/.ssh/authorized_keys`文件中

使用ssh验证ssh访问是否正常


### 在服务端启动lsyncd服务

	lsyncd -log Exec /usr/local/lsyncd-2.1.5/etc/lsyncd.conf


### 观察客户端同步目录是否正确


### 在服务端写入定时任务备份文件
	crontab -e
	0 0 * * *  /bin/gitlab-rake gitlab:backup:create
	0 1 */2 * * find /var/opt/gitlab/backups -mtime +7 -name "*_gitlab_backup.tar"|xargs rm -rf
	
我这里写的是每周二会清理`/var/opt/gitlab/backups`文件中gitlab的备份文件，该时间可以由自己需求进行更改


