---
title: linux—SSH（二）Rsync备份
date: 2016-09-02
tags:
---
一、Rsync软件介绍

rsync从字面上的意思就是remote sync （远程同步）的意思，是类unix系统下的远程数据镜像备份工具，可以镜像保存整个目录树和文件系统，并保持原来文件的权限、时间、软硬链接等，此外它还支持匿名传输；Rsync不仅可以远程同步数据（类似于scp）还可以本地同步数据（类似于cp），但不同于cp或scp的是rsync不像cp/scp一样会覆盖以前的数据，Rsync使用所谓的“Rsync演算法”，这个算法在第一次连通完成时，会把整份文件传输一次，下一次就只传送二个文件之间不同的部份，因此速度相当快
<!--more-->
二、Rsync的传输方式（或工作模式）

l  在本机直接运行拷贝本地文件（不使用冒号）

命令格式为：rsync[OPTION]… SRC DEST。如下

    [root@abao~]# rsync -av /etc /tmp
    
l  通过rsh或ssh的信道在server/client之间进行传输数据（使用一个冒号）

需要登录到服务器上执行任务，并且需要输入账号的密码

1、 将本地机器的内容拷贝到远程机器（目标路径中使用一个冒号）

命令格式为：rsync[OPTION]… SRC [USER@]HOST:DEST。如下


    [root@abao~]# rsync -av -e ssh /tmp axing@172.168.72.68:~

2、 将远程机器的内容拷贝到本地机器（源路径中使用一个冒号）

命令格式为：rsync[OPTION]… [USER@]HOST:SRC DEST。如下


    [root@abao~]# rsync -av -e ssh axing@172.168.72.68:~ /tmp

l  直接通过rsync的服务来传输（此时服务器端需要启动873端口，并且使用两个冒号）

这种方式在远程主机上建立一个rsync的服务器，在服务器上配置好rsync的各种应用，然后本机作为rsync的一个客户端去连接远程的rsync服务器

1、 从远程rsync服务器中拷贝文件到本地机（源路径中使用一个冒号）

命令格式为：rsync[OPTION]… [USER@]HOST::模块名 本地位置。如下


    [root@abao~]# rsync -av axing@172.168.72.68:：back /databack

2、从本地机器拷贝文件到远程rsync服务器中（目标路径中使用一个冒号）

命令格式为：rsync[OPTION]… SRC [USER@]HOST::DEST。如下

    [root@abao~]# rsync -av /databack axing@172.168.72.68:：back
    
以上3中传输方式的差异在于有没有冒号“：”，本地端传输不需要冒号，通过rsh或ssh传输时需要一个冒号，而通过rsync传输时需要两个冒号

三、Rsync的语法

    -v： 查看模式可以列出很多信息包括文件名
    
    -r： 递归复制，可以针对目录来复制
    
    -u： 仅更新，如果目标文件比较新那么则保留新文件不会覆盖
    
    -l： 复制链接文件的属性
    
    -p： 连同属性一起复制
    
    -g： 保存源文件的属组
    
    -o： 保存源文件的属主
    
    -D： 保存源文件的设备属性
    
    -t： 保存原文件的时间参数
    
    -z： 在传输时加上压缩
    
    -e： 使用的协议，例如使用ssh通道就是-e ssh
    
    -a： 相当于-rlptgoD，是最常用的参数
    
    -L： 把SRC中软连接的目标文件给拷贝到DST.
    
    –delete：如果在DST增加文件了，而SRC当中没有这些文件，同步时会删除新增的文件
    
    –exclude=filename：  指定排除不需要传输的文件，等号后面跟文件名（如*.txt）
    
    –progress：  可以看到同步的过程状态，比如文件数量、文件传输速度等

四、在本机直接运行拷贝本地文件实例

    [root@abao ~]# rsync -av /etc/tmp#首次本地备份
    
    …………………………………………………………………
    
    sent 33899909 bytes  received 35626 bytes  5220851.54 bytes/sec
    
    total size is 33759697  speedup is 0.99
    
    [root@abao ~]# ll -d /tmp/etc/etc#两文件相同
    
    drwxr-xr-x. 126 root root12288 10月 10 08:23 /etc
    
    drwxr-xr-x. 126 root root12288 10月 10 08:23 /tmp/etc
    
    [root@abao ~]# rsync -av /etc/tmp#再次备份时只备份差异文件
    
    sending incremental file list
    
    sent 77528 bytes  received 293 bytes  155642.00 bytes/sec
    
    total size is 33759697  speedup is 433.81

五、通过rsh或ssh的信道在server/client之间进行传输数据实例

    [root@abao ~]# /etc/init.d/sshd restart
    
    停止 sshd：[确定]
    
    正在启动 sshd：[确定]
    
l  将本地机器的内容拷贝到远程机器

    [root@abao ~]# rsync -av -e ssh /tmp admin@172.168.72.68:~
    
    admin@172.168.72.68’s password:   #需要输入账户密码
    
    …………………………………………………………………………
    
    sent 238637037 bytes received 60045 bytes  3819153.31bytes/sec
    
    total size is 238396030 speedup is 1.00

l  将远程机器的内容拷贝到本地机器

    [root@abao ~]# rsync -av -e ssh axing@172.168.72.68:~ /tmp
    
    axing@172.168.72.68’s password:   #需要输入账户密码
    
    receiving incremental file list
    
    axing/
    
    axing/.ICEauthority
    
    …………………………………………………………………
    
    sent 1732 bytes  received758924 bytes  80069.05 bytes/sec
    
    total size is 751474 speedup is 0.99
    
    [root@abao ~]# ll -d /tmp/axing
    
    drwx——. 27 500 500 4096 10月 10 08:23 /tmp/axing

l  利用crontab通过ssh进行免密码异地备份脚本（常用）

我们可以针对用户admin制作一个免密码登陆的ssh秘钥，这样以后异地备份系统就可以使用crontab自动备份了，前提是先根据下面（六、直接通过rsync的服务来传输实例）安装并设置好rsync

1、ssh服务器端和客户端账户建立

    [root@baobao ~]# mkdir /home/back; touch /home/back/wo#创建要备份的文件
    
    [root@baobao ~]# chmod -R 755 /home/back/
    
    [root@baobao ~]# useradd admin#先在ssh服务器端ssh账号
    
    [root@baobao ~]# passwd admin
    
    更改用户 admin 的密码 。
    
    新的 密码：
    
    重新输入新的 密码：
    
    passwd： 所有的身份验证令牌已经成功更新。
    
    [root@abao ~]# useradd admin  #客户端上建立ssh账号
    
    [root@abao ~]# passwd admin
    
    更改用户 admin 的密码 。
    
    新的 密码：
    
    重新输入新的 密码：
    
    passwd： 所有的身份验证令牌已经成功更新。

2、客户端建立两把钥匙

    [root@abao ~]# su – admin
    
    [admin@abao ~]$ ssh-keygen
    
    Generating public/private rsa key pair.
    
    Enter file in which to save the key (/home/admin/.ssh/id_rsa):
    
    Created directory ‘/home/admin/.ssh’.
    
    Enter passphrase (empty for no passphrase):
    
    Enter same passphrase again:
    
    Your identification has been saved in /home/admin/.ssh/id_rsa.
    
    Your public key has been saved in /home/admin/.ssh/id_rsa.pub.
    
    The key fingerprint is:
    
    61:01:c2:9b:26:00:0a:01:c9:a0:58:7e:38:f9:85:6c admin@abao
    
    The key’s randomart image is:
    
    +–[ RSA 2048]—-+
    
    |@o… …|
    
    |B+ +.o   .   |
    
    |+.= Eo. o|
    
    |  .=+. . .   |
    
    |   o.   S   |
    
    | |
    
    | |
    
    | |
    
    | |
    
    +—————–+
    
    [admin@abao ~]$ ls -ld ~/.ssh; ls -l ~/.ssh
    
    drwx——. 2 admin admin 4096 10月 10 09:21 /home/admin/.ssh
    
    总用量 8
    
    -rw——-. 1 admin admin 1675 10月 10 09:21 id_rsa
    
    -rw-r–r–. 1 admin admin 392 10月 10 09:21 id_rsa.pub

3、将公钥文件数据上传到服务器

    [admin@abao ~]$ scp ~/.ssh/id_rsa.pub admin@172.168.72.68:~#客户端上传公钥文件
    
    admin@172.168.72.68’s password:
    
    id_rsa.pub100%  3920.4KB/s   00:00

4、将公钥放置到服务器端的正确目录与文件名（服务器上操作）

    [root@baobao ~]# su – admin
    
    [admin@baobao ~]$ ls -ld .ssh
    
    ls: 无法访问.ssh: 没有那个文件或目录
    
    [admin@baobao ~]$ mkdir .ssh; chmod 700 .ssh #服务器上建立文件~/.ssh
    
    [admin@baobao ~]$ ls -ld .ssh
    
    drwx—— 2 admin admin 4096 10月 10 09:36 .ssh
    
    [admin@baobao ~]$ ls -l *pub
    
    -rw-r–r– 1 admin admin 392 10月 10 09:27 id_rsa.pub
    
    [admin@baobao ~]$ cat id_rsa.pub >> .ssh/authorized_keys
    
    [admin@baobao ~]$ chmod 644 .ssh/authorized_keys
    
    [admin@baobao ~]$ ls -l .ssh
    
    总用量 4
    
    -rw-r–r– 1 admin admin 392 10月 10 09:38 authorized_keys

5、在客户端建立异地备份脚本

    [root@abao ~]# mkdir /backups
    
    [root@abao ~]# chmod -R 755 /backups/
    
    [root@abao ~]# su – admin
    
    [admin@abao ~]$ mkdir ~/bin; vim ~/bin/backup.sh
    
    #!/bin/bash
    
    localdir=/backups
    
    remotedir=/home/back/
    
    remoteip=”172.168.72.68″
    
    [ -d ${localdir} ] || mkdir ${localdir}   #-d是判断是否有这个目录；符号“||”是逻辑或意思，左边为假时执行右边命令；小括号一般用作执行命令，而自定义变量一般用大括号括起来
    
    for dir in ${remotedir}
    
    do
    
    rsync -av -essh admin@${remoteip}:${dir} ${localdir}
    
    done
    
    [admin@abao ~]$ chmod 755 ~/bin/backup.sh
    
    [admin@abao ~]$ ls -ld /home/admin/bin/backup.sh
    
    -rwxrwxr-x. 1 admin admin 224 10月 11 08:49 /home/admin/bin/backup.sh
    
    [admin@abao ~]$ ~/bin/backup.sh   #执行脚本
    
    receiving incremental filelist
    
    sent 11 bytes  received 43 bytes  5.68 bytes/sec
    
    total size is 1010  speedup is 18.70
    
6、制作crontab计划任务

    [root@abao ~]# crontab –e #每天的凌晨02:00异地备份服务器上的/etc /root /home目录到本地的/backups/下
    0 2 * * * /home/admin/bin/backup.sh

六、直接通过rsync的服务来传输实例

l  创建用户账号和rsync配置文件

    [root@baobao ~]# useraddadmin   #创建链接用户
    
    [root@baobao ~]# passwd admin
    
    更改用户 admin 的密码。
    
    新的 密码：
    
    重新输入新的 密码：
    
    passwd： 所有的身份验证令牌已经成功更新。
    
    [root@baobao ~]# mkdir /home/back   #创建要进行备份的目录或文件
    
    [root@baobao ~]# touch/home/back/guo
    
    [root@baobao ~]# vim/home/back/guo
    
    [root@baobao ~]# chmod -R 755/home/back/#设定要备份的目录或文件权限
    
    [root@baobao ~]# yum -yinstall xinetd
    
    [root@baobao ~]# yum -yinstall rsync#安装rsync
    
    [root@baobao ~]# yum -yinstall rsync#客户端也安装rsync
    
    [root@baobao ~]# touch/etc/rsyncd.conf  #默认该文件是没有的
    
    [root@baobao ~]# chmod 600/etc/rsyncd.conf  #修改配置文件权限

l  Rsync服务器端配置文件设置

配置文件时即时生效的，不用重启服务

1、/etc/rsyncd.conf配置

1)  全局参数配置

    [root@baobao ~]# manrsyncd.conf   #查看说明文档看下面部分参数是yes/no还是true/false
    
    [root@baobao ~]# vim/etc/rsyncd.conf
    
    uid=root  #运行RSYNC守护进程的用户
    
    gid=root  #运行RSYNC守护进程的组
    
    use chroot=false  #不使用chroot
    
    max connections=8 #最大连接数是4
    
    pid file=/var/run/rsyncd.pid  #pid文件默认存放位置
    
    lock file=/var/run/rsync.lock #锁文件默认存放位置（锁住rsync正在操作的文件不让其他的程序对其进行写操作）
    
    log file=/var/log/rsyncd.log  #日志文件默认存放位置
    
    strict modes=true #是否检查口令文件的权限
    port=873  #默认端口873
    
2)  模块参数配置(多台客户端需要设置多个模块)

    [backup]   #认证的模块名，在client端需要指定
    
    path=/etc  #需要做备份的目录
    
    comment=This is backup #这个模块的注释信息
    
    list=true  #当用户查询该服务器上的可用模块时，该模块是被列出（true）还是被隐藏（false）
    
    max connections=6  #客户端最大连接数(默认0没限制)，模块里可以不设置
    ignore errors  #可以忽略一些无关的IO错误
    read only=false#“yes”只读客户端不能上传；“no”客户端可以上传
    write only=false   #“yes”客户端不能下载；“no”客户端可以下载
    
    uid=root   #指定该模块传输文件时守护进程应该具有的uid
    gid=root   #指定该模块传输文件时守护进程应该具有的gid
    hosts allow=172.168.0.0/16 #允许连接的主机（“*”充许任何主机连接），多个主机用“，”分开；多个网段用空格隔开
    hosts deny=192.168.10.0/32 #禁止连接的主机或网段
    
    auth users=admin   #登陆系统使用的用户名（系统必须存在的用户），没有默认为匿名
    secrets file= /etc/rsyncd.secrets #登陆用户的密码文件（需要自己生成）

2、rsync server启动文件配置

    [root@baobao ~]# vim /etc/xinetd.d/rsync#只修改disable = no即可
    
    # default: off
    
    # description: The rsyncserver is a good addition to an ftp server, as it \
    
    #   allows crc checksumming etc.
    
    service rsync
    
    {
    
    disable = no
    
    flags   = IPv6
    
    socket_type = stream
    
    wait= no
    
    user= root
    
    server  = /usr/bin/rsync
    
    server_args = –daemon
    
    log_on_failure  += USERID
    
    }
    
    [root@baobao ~]# chkconfigrsync on
    
l  创建密码文件、欢迎信息

1、生成rsync密码文件并设置该文件相应权限
    [root@baobao ~]# touch /etc/rsyncd.secrets
    
    [root@baobao ~]# vim /etc/rsyncd.secrets
    
    admin:guobaobao!1314#格式为“账号：密码”且一行一个
    [root@baobao ~]# chown root.root /etc/rsyncd.secrets
    
    [root@baobao ~]# chmod 600 /etc/rsyncd.secrets
    
    因为rsyncd.secrets存储了rsync服务的用户名和密码，所以要将rsyncd.secrets设置为root拥有, 且权限为600

2、配置欢迎信息rsyncd.motd（可有可无）
    [root@baobao ~]# vim /etc/rsyncd.motd   #rsyncd.motd记录了rsync服务的欢迎信息
    
    Welcome to use the rsyncservices!
    [root@baobao ~]# service xinetd restart
    
    停止 xinetd：  [确定]
    
    正在启动 xinetd：  [确定]
    
l  Rsync的启动与开机启动

1、Rsync服务端启动

1)  载入配置

    [root@baobao ~]# rsync –daemon–config=/etc/rsyncd.conf   #载入配置并启动
    
    或
    
    [root@baobao ~]# /etc/rc.d/init.d/xinetdreload
    
    重新载入配置： [确定]

2)  重启rsync

    [root@baobao ~]# /usr/bin/rsync –daemon
    
    failed to create pid file/var/run/rsyncd.pid: File exists
    
    或
    
    [root@baobao ~]# /etc/rc.d/init.d/xinetdrestart#重新启动
    
    停止 xinetd：  [确定]
    
    正在启动 xinetd：  [确定]

3)  检查rsync是否启动

    [root@baobao ~]# netstat -lnp | grep 873#由超级进程启动
    
    tcp   0  0 0.0.0.0:8730.0.0.0:*LISTEN  3133/rsync
    
    tcp   0  0 :::873 :::*LISTEN 3133/rsync
    
    或
    
    [root@baobao ~]# netstat -a |grep rsync
    
    tcp0  0*:rsync *:* LISTEN
    
    或
    
    [root@baobao ~]# lsof -i:873 #端口没有被占用
    
    COMMAND  PID USER  FD   TYPE DEVICE SIZE/OFF NODENAME
    
    rsync   3133 root   4u  IPv4  22973 0t0  TCP *:rsync (LISTEN)
    
    rsync   3133 root   5u  IPv6  22974 0t0  TCP *:rsync (LISTEN)

4)  查看rsync日志

    [root@baobao ~]# cat /var/log/rsyncd.log #查看rsync日志
    
    2014/10/14 15:29:13 [35681] rsyncdversion 3.0.6 starting, listening on port 873
    
2、将rsync加入开机启动

    [root@baobao ~]# echo”/usr/bin/rsync –daemon –config=/etc/rsyncd.conf”>>/etc/rc.local
    
    [root@baobao ~]# cat/etc/rc.local
    
    #!/bin/sh
    
    #
    
    # This script will beexecuted *after* all the other init scripts.
    
    # You can put your owninitialization stuff in here if you don’t
    
    # want to do the full Sys Vstyle init stuff.
    
    touch /var/lock/subsys/local
    
    /usr/bin/rsync –daemon–config=/etc/rsyncd.conf

l  Rsync服务器端配置防火墙

1、防火墙设置

    [root@baobao ~]# iptables -F
    
    [root@baobao ~]# iptables -X
    
    [root@baobao ~]# iptables -Z
    
    [root@baobao ~]# iptables -AINPUT -i eth0 -s 172.168.0.0/16 -p tcp –dport 22 -j ACCEPT
    
    [root@baobao ~]# iptables -L
    
    Chain INPUT (policy ACCEPT)
    
    target prot opt source   destination
    
    ACCEPT tcp —  ACA80000.ipt.aol.com/16  anywheretcp dpt:ssh
    
    Chain FORWARD (policy ACCEPT)
    
    target prot opt source   destination
    
    Chain OUTPUT (policy ACCEPT)
    
    target prot opt source   destination
    
    [root@baobao ~]#/etc/init.d/iptables save
    
    iptables：将防火墙规则保存到 /etc/sysconfig/iptables： [确定]
    
    [root@baobao ~]#/etc/init.d/iptables restart
    
    iptables：清除防火墙规则：[确定]
    
    iptables：将链设置为政策 ACCEPT：filter[确定]
    
    iptables：正在卸载模块：  [确定]
    
    iptables：应用防火墙规则：[确定]

2、selinux设置


    [root@baobao ~]# setenforce 0

l  客户端测试

    [root@abao ~]# rsync -avadmin@172.168.72.68::backup /backups/
    
    Password:
    
    receiving incremental filelist
    
    created directory /backups
    
    ./
    
    guo
    
    sent 79 bytes  received 1162 bytes  275.78 bytes/sec
    
    total size is 1010  speedup is 0.81

可以看到这里还是需要输入密码，这样同样也不能写入脚本中自动执行

l  常见问题处理

1、错误一

@ERROR: chroot failed

rsync error: error startingclient-server protocol (code 5) at main.c(1522) [receiver=3.0.3]

原因：服务器端的目录不存在或无权限

2、错误二

@ERROR: auth failed on moduletee

rsync error: error startingclient-server protocol (code 5) at main.c(1522) [receiver=3.0.3]

原因：服务器端该模块（tee）需要验证用户名密码，但客户端没有提供正确的用户名密码，认证失败。

3、错误三

@ERROR: Unknown module‘tee_nonexists’

rsync error: error startingclient-server protocol (code 5) at main.c(1522) [receiver=3.0.3]

原因：服务器不存在指定模块

4、错误四

password file must not beother-accessible

continuing without passwordfile

Password:

原因：这是因为rsyncd.pwdrsyncd.secrets的权限不对，应该设置为600

5、错误五

rsync: failed to connect to218.107.243.2: No route to host (113)

rsync error: error in socketIO (code 10) at clientserver.c(104) [receiver=2.6.9]

原因：对方没开机、防火墙阻挡、通过的网络上有防火墙阻挡，都有可能

6、错误六

rsync error: error startingclient-server protocol (code 5) at main.c(1524) [Receiver=3.0.7]

原因：/etc/rsyncd.conf配置文件内容有错误

7、错误七

rsync: chown “”failed: Invalid argument (22)

原因：权限无法复制，去掉同步权限的参数即可(这种情况多见于Linux向Windows的时候)

8、错误八

@ERROR: daemon security issue– contact admin
rsync error: error starting client-server protocol (code 5) at main.c(1530)[sender=3.0.6]

原因：同步的目录里面有软连接文件，需要服务器端的/etc/rsyncd.conf打开use chroot = yes掠过软连接文件。

l  建立不需输入密码的链接

这样就可以将其写入脚本和任务计划自动运行了

1、第一种方案：指定密码文件

    [root@abao ~]# touch/etc/pass#客户端上建立密码文件
    
    [root@abao ~]# vim /etc/pass #将账户“admin”密码写入
    
    [root@abao ~]# cat /etc/pass
    
    guobaobao！1314
    
    [root@abao ~]# chmod 600/etc/pass
    
    [root@abao ~]# rsync -avadmin@172.168.72.68::backup /backups/
    
    Password:
    
    @ERROR: auth failed on modulebackup
    
    rsync error: error startingclient-server protocol (code 5) at main.c(1503) [receiver=3.0.6]
    
    [root@abao ~]#
    
    [root@abao ~]# rsync -avadmin@172.168.72.68::backup /backups/ –password-file=/etc/pass #注意黑体部分指定密码文件
    
    receiving incremental filelist
    
    sent 57 bytes  received 106 bytes  326.00 bytes/sec
    
    total size is 1010  speedup is 6.20
    
2、第二种方案：在rsync服务器端不指定用户

    [root@baobao ~]# vim/etc/rsyncd.conf   #服务器端配置文件注释掉下面认证两行
    
    #auth users=admin
    
    #secretsfile=/etc/rsyncd.secrets
    
    [root@abao ~]# rsync -avadmin@172.168.72.68::backup /backups/
    
    receiving incremental filelist
    
    sent 28 bytes  received 65 bytes  37.20 bytes/sec
    
    total size is 1010  speedup is 10.86
    
    或
    
    [root@abao ~]# rsync -av172.168.72.68::backup /backups/#不加账户默认是root
    
    receiving incremental filelist
    
    sent 28 bytes  received 65 bytes  186.00 bytes/sec
    
    total size is 1010  speedup is 10.86