---
title: linux 命令rsync+crontab实现自动同步
date: 2016-10-05
tags: rsync
categories: rsync
---
linux 命令rsync+crontab实现自动同步,这个技术现在已经用得很广泛了,比起第三方的软件要可靠好使,所以得到系统管理员的广泛应用;在此,我给大伙来分享一下;请指教.

首先,我们来了解一下这个命令:

rsync命令格式:rsync [option] 源路径 目标路径;
<!--more-->
其中:  

[option]:  

    a:使用archive模式,等于-rlptgoD,即保持原有的文件权限;
    
    z:表示传输时压缩数据;
    
    v:显示到屏幕中;
    
    e:使用远程shell程序(可以使用rsh或ssh;
    
    --delete:精确保存副本,源主机删除的文件,目标主机也会同步删除;
    
    --include=PATTERN:不排除符合PATTERN的文件或目录;
    
    --exclude=PATTERN:排除所有符合PATTERN的文件或目录;
    
    --password-file:指定用于rsync服务器的用户验证密码;
    
源路径和目标路径可以使用如下格式:


    rsync://[USER@]Host[:Port]/Path #--rsync服务器路径;
    
    [USER@]Host::Path   #--rsync服务器的另一种表示形式;
    
    [USER@]Host:Path#--远程路径;
    
    LocalPath   #--本地路径;
    
知道上述命令的基本格式了吗?

下面我们来讲安装rsyn命令;


    [root@dbserver ~]#yum list rsync*
    
    Loaded plugins: fastestmirror, refresh-packagekit, security
    
    Loading mirror speeds from cached hostfile
    
     * rpmforge: mirrors.neusoft.edu.cn
    
    Installed Packages
    
    rsync.i686  3.0.6-9.el6   @anaconda-CentOS-201207051201.i386/6.3
    
    [root@dbserver ~]#yum -y install rsync*
    
前面是查看rsync RPM包,后面是安装rsync这个命令;

安装完后,我们便可以来配置rsync服务器与客服端了;

实例:

A服务器:192.168.1.213

B客户端:192.168.1.210

首先人们配置服务器,look,

在配置服务器之前要先生成密钥,ssh-keygen -t rsa,生成密钥如下:


    [root@masternagios .ssh]# ls
    
    id_rsa  id_rsa.pub
    
    [root@masternagios .ssh]#  scp id_rsa_pub root@192.168.1.210:/root/.ssh/authorized_keys

在客户端也要如下操作:


    [root@masternagios .ssh]# ssh-keygen -t rsa
    
    [root@masternagios .ssh]# ls
    
    id_rsa  id_rsa.pub  authorized_keys(213的公钥)
    
    [root@masternagios.ssh]#
    
    scp id_rsa_pub root@192.168.1.213:/root/.ssh/authorized_keys

这样两台机可以无密码SSH登陆,以便后面我们同步方便;当然,不要上述的操作也能实现;那么如下操作:

服务端:

    vi /etc/sery.pass  权限:600(chmod 600 /etc/sery.pass)
    
    root:123456

客服端:
    
    vi /etc/sery_client.pass  权限:600(chmod 600 /etc/sery_client.pass)
    
    123456

生成的这两件文件后面有用处的;

然后新建配置文件vi /etc/rsyncd.conf,内容如下图示:
![](http://hiphotos.baidu.com/exp/pic/item/d872d695d143ad4b038f881c83025aafa50f060e.jpg)
解析如下:

    uid = root           #root用户访问(我这里用ROOT用户,也可以用其他新建的用户)

    gid = root           #root组用户访问

    use chroot = no      #不能使用chroot

    max connections = 10  #最大连接数

    list = yes           #允许列出文件清单

    pid file = /var/run/rsyncd.pid

    lock file = /var/run/rsyncd.lock

    log file = /var/log/rsyncd.log

    hosts allow  = 192.168.1.2      #只允许这个主机访问

   [data]                    #发布项(注意这个命名)

    path = /webapps/IDManage         #发布的路径

    ignore errors

    read only = yes            #只读

    auth users = root                #认证用户为root

    secrets file = /etc/sery.pass    #密码文件

然后我们来启动:

[root@masternagios ~]# rsync --daemon --config=/etc/rsyncd.conf

    [root@masternagios ~]# ps -ef |grep rsync

    root     21359     1  0 Aug24 ?        00:00:00 rsync --daemon --     config=/etc/rsyncd.conf

    root     24018 23885  0 10:38 pts/0    00:00:00 grep rsync

    [root@masternagios ~]#lsof -i:873

    COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME

    rsync   21359 root    4u  IPv4 1558266      0t0  TCP *:rsync (LISTEN)

    rsync   21359 root    5u  IPv6 1558267      0t0  TCP *:rsync (LISTEN)

然后在客户端测试:
    
    [root@dbserver ~]# telnet 192.168.1.213 873
    
    Trying 192.168.1.213...
    
    Connected to 192.168.1.213.
    
    Escape character is '^]'.
    
    @RSYNCD: 30.0
    
    ^]
    
    telnet> q
    
    Connection closed.

说明网络端口开放,没有问题;通常在这配置时会发现一些问题,比如报错(111)--说明服务器端口未开启,就检查一下rsync服务有没有开启;

报错(1503)(1536)--说明无 [data] #发布项(注意这个命名),这里命令一定要对应上同步::[data];

我们再来把服务端rsync加自动启动;

    echo "/usr/bin/rsync --daemon --config=/etc/rsyncd.conf" >>/etc/rc.local

配置客户端;

客户端只要安装rsync这个命令便可以实现,所以,我们来测试同步实现;

    [root@dbserver ~]#rsync -aSvH /webapps/IDManage/ root@192.168.1.213::data --password-file=/etc/sery_client.pass
    
可以看到:
![](http://hiphotos.baidu.com/exp/pic/item/346bd85c10385343bd094ffd9213b07ec88088ed.jpg)
命令执行成功;说明服务端与客户端都没有问题;

如何自实rsync客户端自动与rsync服务器端同步呢?这里我们用到计划任务命令:crontab;

首先,我们来做一个shell脚本,

    [root@dbserver ~]#vi /tmp/rsyncd.sh
    
    #!/bin/bash
    
    rsync -aSvH /webapps/IDManage/ root@192.168.1.213::data --password-file=/etc/sery_client.pass
    
    wq!   ##保存退出
    
    [root@dbserver ~]#crontab -e
    
    */5 * * * * sh /tmp/rsyncd.sh #第5分钟执行一次同步;
    
    wq!   ##保存退出

看了,到此分享linux 命令rsync+crontab实现自动同步,已经结束;总结一点:rsync命令格式一定要知道:rsync [option] 源路径目标路径,目标路径的格式有几种,大家只要记得一两种便可以了;
