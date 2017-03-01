---
title: linux—SSH1
date: 2016-09-02
tags:
---
一、远程连接简介

l  远程连接服务器

远程连接服务器通过文字或图形接口的方式来远程登陆系统，在远程的终端前面登陆linux主机并取得操作主机的接口shell，登陆后的操作就像在系统前面一样，这样可以进行系统管理的任务
<!--more-->
l  工作站

工作站就是仅提供大量的运作能力给用户而不提供因特网服务的主机

l  当前远程连接的登陆类型

1、加密的远程连接

主要是ssh，也是用到的最多最安全的，而且还可以使用rsync通过ssh协议来进行异地备份

2、图形接口的远程连接

XDMCP、VNC和XRDP，这些方式由于是传输图形所以速度慢，且安全性也不是很好

二、远程连接之SSH

Ssh是secure shell protocol（安全的壳程序协议）的简写，可以通过数据包加密技术将传输的数据包加密后再传输到网络上，当前ssh有两个版本，其中version2加上了连接检测的机制，可以避免连接期间被插入恶意攻击码

l  Ssh服务器提供的功能

默认的ssh服务器提供两个服务器功能

1、ssh服务

类似telnet的远程连接，使用shell的服务器，可以用来管理服务器

2、ftp服务

类似ftp服务的sftp-server，可以用来远程上传和下载

l  连接的加密技术

当前常见的网络数据包加密技术主要是通过非对称秘钥系统来处理的，主要是通过两把不同的公钥（public key）和私钥（private key）来进行数据的加密与解密的，且在同一个方向上的连接中这两把钥匙是成对存在的。每台主机都应该有自己的秘钥（公钥和私钥），并且公钥用于加密而私钥用于解密

1、公钥（public key）

提供给远程主机进行数据加密的行为，所有客户端都能取得它进行数据加密

2、私钥（private key）

远程主机使用公钥加密的数据在本地就需要使用私钥进行解密了，其非常重要只能在自己的主机上

l  Ssh服务器端与客户端的链接步骤

1、服务器建立公钥文件

系统安装完成时sshd会朱勇去计算出需要的公钥文件和自己需要的私钥文件，等下次再次启动sshd的时候该服务就会主动去找文件/etc/ssh/ssh_host*

2、客户端主动链接

需要使用客户端程序（如ssh）来连接

3、服务器传送公钥文件给客户端

服务器将将取得的公钥文件/etc/ssh/ssh_host*传送给客户端（由于公钥是给大家使用的，所以此时的传送是明文的）

4、客户端记录并比对该公钥数据，然后计算出自己的公钥和私钥

客户端在第一次连接该服务器后会将服务器的公钥数据记录到客户端的用户主目录下的~ /.ssh/known_hosts内，如果已经记录过该数据则客户端会去比对此次受到的公钥与之前的差异，若接受此次公钥数据那么会计算出客户端自己的公钥和私钥

5、客户端将自己的公钥传送给服务器

此时服务器端会有自己的私钥和客户端的公钥；而客户端会有服务器的公钥和自己的私钥，这时服务器与客户端的秘钥（公钥与私钥）是不一样的，所以才称之为非对称式秘钥系统

6、服务器开始进行双向加解密

1)   服务器传送数据到客户端

将用户的公钥加密后进行发送，客户端接收后用自己的私钥解密

2)   客户端传送数据到服务器

将服务器的公钥加密后进行发送，服务器接受后用自己的私钥解密

l  秘钥文件的建立

1、服务器端启动ssh服务，以生成服务器端的公钥和私钥


    [root@baobao ~]# /etc/init.d/sshd restart

停止 sshd：                                                [确定]

生成 SSH1 RSA 主机键：                                     [确定]

生成 SSH2 RSA 主机键：                                     [确定]

正在生成 SSH2 DSA 主机键：                                 [确定]

正在启动 sshd：                                            [确定]

2、客户端利用ssh连接服务器


    [root@abao ~]# ssh 172.168.72.68

    The authenticity of host ‘172.168.72.68 (172.168.72.68)’ can’tbe established.
    
    RSA key fingerprint is36:cf:9f:46:54:46:4b:b3:b0:48:4a:93:4d:14:81:56.
    
    Are you sure you want to continue connecting (yes/no)? yes
    
    Warning: Permanently added ‘172.168.72.68’ (RSA) to the list ofknown hosts.
    
    root@172.168.72.68’s password:
    
    Last login: Sun Sep 28 15:37:10 2014 from localhost
    
    [root@baobao ~]# exit
    
    logout
    
    Connection to 172.168.72.68 closed.

3、删除客户端的秘钥文件

    [root@abao ~]# rm /etc/ssh/ssh_host*
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_dsa_key”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_dsa_key.pub”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_key”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_key.pub”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_rsa_key”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_rsa_key.pub”？y

4、重新启动客户端的sshd服务以查看秘钥文件的建立过程

    [root@abao ~]# /etc/init.d/sshd restart
    
    停止 sshd：[确定]
    
    生成 SSH1 RSA 主机键： [确定]
    
    生成 SSH2 RSA 主机键： [确定]
    
    正在生成 SSH2 DSA 主机键： [确定]
    
    正在启动 sshd：[确定]
    
l  Sshd服务的启动

    [root@baobao ~]# /etc/init.d/sshd restart
    
    停止 sshd：[确定]
    
    正在启动 sshd：[确定]
    
    [root@baobao ~]# netstat -tlnp | grep ssh #注意ssh服务是tcp端口22
    
    tcp 0  0 0.0.0.0:22  0.0.0.0:*LISTEN 9262/sshd
    
    tcp 0  0:::22:::* LISTEN 9262/sshd

在linux系统中默认就有ssh所需要的软件了，包括可以产生密码等协议的OpenSSL软件和OpenSSH软件，而且在当前的linux系统中都是默认启动ssh的。这个sshd可以同时提供shell与ftp，而且都是在tcp端口22

l  Linux用户ssh客户端的连接程序

Linux客户端默认情况下是可以正常使用ssh的而不必安装额外的软件，而且其默认是启动的

1、 直接登录远程主机的指令ssh（可用作服务器管理）

Ssh命令格式为：ssh [-f][-o参数项目][-p非标准端口][账号@]IP地址[命令]

1)   Ssh命令参数介绍

-f：     需要配合后面的[命令]，不登陆远程主机直接发送一个命令过去而已

-o：     主要的参数有：ConnectTimeout=秒数：连接等到的秒数，减少等待的时间

StrictHostKeyChecking=yes/no/ask：默认是ask，如果想要public key主动加入known_host这里设置为no

-p：     如果sshd服务启动在非标准的端口需使用该项目[命令]，不登陆远程主机直接发送一个命令过去，但与-f意义不太相同

2)   ssh使用范例

A：直接登录到远程主机

    [root@abao ~]# ssh 172.168.72.68
    
    The authenticity of host ‘172.168.72.68 (172.168.72.68)’ can’tbe established.
    
    RSA key fingerprint is36:cf:9f:46:54:46:4b:b3:b0:48:4a:93:4d:14:81:56.   #远程服务器的公钥指纹码
    
    Are you sure you want to continue connecting (yes/no)? yes   #将上述指纹码写入服务器公钥记录文件~ /.ssh/known_hosts，等再次登录时就不会出现该指纹码提示了。一定要yes而不是y
    
    Warning: Permanently added ‘172.168.72.68’ (RSA) to the list ofknown hosts.
    
    root@172.168.72.68’s password:#远程主机的root密码
    
    Last login: Sun Sep 28 15:37:10 2014 from localhost
    
    [root@baobao ~]# exit #退出远程连接
    
    logout
    
    Connection to 172.168.72.68 closed.

一般我们使用“ssh 账号 主机IP地址”的登录方式，如果不写账号的话那么会以本地计算机的当前账号来尝试登录远程主机

B：再次登录远程主机

    [root@abao ~]# ssh 172.168.72.68
    
    root@172.168.72.68’s password:
    
    Last login: Sun Sep 28 16:42:05 2014 from aca84448.ipt.aol.com

C：使用账号axing登录

    [root@baobao ~]# ssh axing@172.168.72.68
    
    The authenticity of host ‘172.168.72.68 (172.168.72.68)’ can’tbe established.
    
    RSA key fingerprint is 36:cf:9f:46:54:46:4b:b3:b0:48:4a:93:4d:14:81:56.
    
    Are you sure you want to continue connecting (yes/no)? yes
    
    Warning: Permanently added ‘172.168.72.68’ (RSA) to the list ofknown hosts.
    
    axing@172.168.72.68’s password:
    
    [axing@baobao ~]$   #远程登陆后身份变为axing
    
D：远程登陆执行命令后立刻离开

    [root@abao ~]# ssh axing@172.168.72.68 find / -name passwd  #既后面直接加命令
    
    axing@172.168.72.68’s password:
    
卡在这里等待命令的执行完毕

E：让远程主机自动运行命令而立刻回到本地端继续工作
    
    [root@abao ~]# ssh -faxing@172.168.72.68 shutdown -h now
    
    axing@172.168.72.68’s password:

F：自动加上公钥记录而不再询问

    [root@abao ~]# rm ~/.ssh/known_hosts
    
    rm：是否删除普通文件 “/root/.ssh/known_hosts”？y
    
    [root@abao ~]# ssh -o StrictHostKeyChecking=no root@172.168.72.68#不在要求输入yes或no了
    
    Warning: Permanently added ‘172.168.72.68’ (RSA) to the list ofknown hosts.
    
    root@172.168.72.68’s password:
    
    Last login: Mon Sep 29 14:00:28 2014 from aca84448.ipt.aol.com
    
2、服务器公钥记录文件~ /.ssh/known_hosts

当远程登陆服务器时本机会主动将从服务器收到的公钥服务器公钥记录文件~ /.ssh/known_hosts进行比对，如果服务器的公钥文件还没有记录那么就会主动询问是否记录（登陆时候的yes或no行为）；如果收到的公钥已经记录那么会比对记录是否相同，如果相同则继续登陆，如果不同就会离开登陆而返回。但是如果是服务器重新安装那么服务器的公钥就会经常变化，这样的话我们就无法正常远程登陆了

A：模拟服务器重新安装后ssh登陆

    [root@baobao ~]# rm /etc/ssh/ssh_host*
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_dsa_key”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_dsa_key.pub”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_key”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_key.pub”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_rsa_key”？y
    
    rm：是否删除普通文件 “/etc/ssh/ssh_host_rsa_key.pub”？y
    
    [root@baobao ~]# /etc/init.d/sshd restart
    
    停止 sshd：[确定]
    
    生成 SSH1 RSA 主机键： [确定]
    
    生成 SSH2 RSA 主机键： [确定]
    
    正在生成 SSH2 DSA 主机键： [确定]
    
    正在启动 sshd：[确定]
    
    [root@baobao ~]# ssh 172.168.72.68
    
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    
    @WARNING: REMOTE HOSTIDENTIFICATION HAS CHANGED! @
    
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    
    IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
    
    Someone could be eavesdropping on you right now(man-in-the-middle attack)!
    
    It is also possible that the RSA host key has just been changed.
    
    The fingerprint for the RSA key sent by the remote host is
    
    17:7e:8e:e9:fd:df:1c:e5:c9:9d:cd:30:31:5e:a6:45.
    
    Please contact your system administrator.
    
    Add correct host key in /root/.ssh/known_hosts to get rid ofthis message.
    
    Offending key in /root/.ssh/known_hosts:2#有问题的数据行号
    
    RSA host key for 172.168.72.68 has changed and you haverequested strict checking.
    
    Host key verification failed.
    
B：上述现象解决方法
    
    [root@baobao ~]# vim /root/.ssh/known_hosts  #清空该文件
    
    [root@baobao ~]# ssh 172.168.72.68
    
    The authenticity of host ‘172.168.72.68 (172.168.72.68)’ can’tbe established.
    
    RSA key fingerprint is17:7e:8e:e9:fd:df:1c:e5:c9:9d:cd:30:31:5e:a6:45.
    
    Are you sure you want to continue connecting (yes/no)? yes#记录公钥
    
    Warning: Permanently added ‘172.168.72.68’ (RSA) to the list ofknown hosts.
    
    root@172.168.72.68’s password:
    
    Last login: Mon Sep 29 14:28:46 2014 from aca84448.ipt.aol.com

3、模拟FTP的文件传输之SFTP

如果想要从远程服务器下载或上传文件就不能使用ssh了，而必须使用sftp或scp，这两个指令也是使用ssh的端口22，只是模拟成ftp与复制的操作而已

1)   SFTP使用的命令

Sftp使用的命令与ftp是一样的

A：针对远程服务器的命令

跟linux服务器命令相同

B：针对本机的命令

在基本命令前面加“l”即代表是针对本机的操作，例如sftp> lcd /tmp进入本机的该目录

C：针对资料的上传或下载的命令

put [本地目录或文件][远程]或put [本地目录或文件]（这样会存储到远程主机的目录下）

get [远程目录或文件][本机]或get [远程目录或文件]（这样会存储到当前本机所在目录）

2)   Sftp的使用范例

A：sftp的登陆于退出

    [root@baobao ~]# sftp 172.168.72.68
    
    Connecting to 172.168.72.68…
    
    root@172.168.72.68’s password:
    
    sftp>
    
    或
    
    [root@baobao ~]# sftp axing@172.168.72.68
    
    Connecting to 172.168.72.68…
    
    axing@172.168.72.68’s password:
    
    sftp> exit

B：上传与下载

    sftp> pwd#查看当前在服务器的目录
    
    Remote working directory: /home/axing
    
    sftp> lls /etc/hosts #查看本机是否有该文件
    
    /etc/hosts
    
    sftp> put /etc/hosts #上传该文件到远程服务器
    
    Uploading /etc/hosts to /home/axing/hosts#上传到服务器的默认目录
    
    /etc/hosts 100%  1580.2KB/s   00:00
    
    sftp> ls
    
    hosts
    
    sftp> ls –a  #查看服务器该目录下的隐藏文件
    
    .   ..  .bash_history   .bash_logout.bash_profile   .bashrc .emacs
    
    .gnome2.mozillahosts
    
    sftp> lcd /tmp#切换到本地的目录/tmp下
    
    sftp> get .bashrc #从服务器下载该文件
    
    Fetching /home/axing/.bashrc to .bashrc
    
    /home/axing/.bashrc   100%  124 0.1KB/s  00:00
    
    sftp> lls –a #确认是否下载成功
    
    ……………………………………………
    
    baoaj  .bashrc  guoal   guobe  guobx guocq  guodj  pulse-Fhzh5o9BSGRy
    
    sftp> exit

4、文件异地直接复制SCP

当已经知道服务器上的文件名时可以使用该命令，该命令的上传和下载使用格式如下：

上传：scp [-pr] [-l 速率] file [账号@]主机：目录名（：后没有空格）

下载：scp [-pr] [-l 速率] [账号@]主机：file 目录名（file后有空格）

1)   命令参数介绍

-p：     保留文件的原有权限

-r：     复制整个目录

-l：     传输速率

2)   Scp使用范例

A：上传本地文件到远程服务器

`[root@baobao ~]# scp /etc/hosts* axing@172.168.72.68:~     ` #上传到服务器的用户主目录下

axing@172.168.72.68’s password:

hosts                100% 158     0.2KB/s   00:00

hosts.allow          100% 370     0.4KB/s   00:00

hosts.deny           100% 460     0.5KB/s   00:00

B：从服务器下载文件到本地

    [root@baobao ~]# scp axing@172.168.72.68:/etc/bashrc /tmp
    
    axing@172.168.72.68’s password:
    
    bashrc   100%2681 2.6KB/s   00:00
    
l  windows用户ssh客户端的连接程序

默认的windows并没有ssh的客户端程序，所以需要下载第三方软件才行，常见的有pietty、psftp和filezilla

1、直接连接的pietty（可用作服务器管理）

下载后安装即可使用，不过由于编码的问题中文会显示乱码需要设置该软件才行option—more options—features（右第二个打钩打开键盘数字）—connection—-ssh（右2only选择版本）

option—font（脚本gb2312调整字符集支持中文）

2、psftp

下载后安装并启动，输入open172.168.72.68后连接即可

3、filezilla

下载后安装运行即可，是普通的中文界面ftp软件

l  windows用户远程登陆管理服务器工具xshell（当前最好用的工具）

1、xshell界面

它是windows下当前最好用的远程管理软件，只需要你下载后安装即可使用，一般还有中文版，非常好用，以下是链接后的画面

    Connecting to 172.168.72.68:22…
    
    Connection established.
    
    To escape to local shell, press ‘Ctrl+Alt+]’.  #回到本地shell
    
    Last login: Tue Sep 30 08:55:23 2014 from aca80058.ipt.aol.com
    
    [root@baobao ~]# ls
    
    123baoae  cheng  last.list  lsrootaf   图片
    
    anaconda-ks.cfg  baoagguo.txt  lsrootaa regular_express.txt.1  下载
    
    baoaa baoah  homelsrootab   xinzi.txt  音乐
    
    baoab   baoai  homefile  lsrootac   公共的 桌面
    
    ………………………………………………………
    
    [root@baobao ~]#

2、xshell中文乱码解决法案

Xshell是个非常不错的工具。但很多时候中文显示为乱码的问题，解决方法其实很简单的，即把xshell编码方式改成UTF-8即可：[文件]–>[打开]–>在打开的session中选择连接的那个然后右键点击[属性] -> [终端]，编码选择为：Unicode(UTF-8)，然后重新连接服务器即可

3、ssh远程登陆日志（重要）

    [root@baobao ~]# cat /var/log/secure
    
    thenticationAgent, locale zh_CN.UTF-8)
    
    Sep 30 08:55:23 baobao sshd[2916]: Accepted password for rootfrom 172.168.0.88 port 57853 ssh2
    
    Sep 30 08:55:23 baobao sshd[2916]: pam_unix(sshd:session):session opened for user root by (uid=0)
    
    Sep 30 09:21:21 baobao sshd[3331]: Accepted password for rootfrom 172.168.0.88 port 52386 ssh2
    
    Sep 30 09:21:21 baobao sshd[3331]: pam_unix(sshd:session):session opened for user root by (uid=0)
    
    Sep 30 09:21:55 baobao sshd[2916]: pam_unix(sshd:session):session closed for user root

l  sshd服务器配置

sshd服务器的详细配置都放在/etc/ssh/sshd_config配置文件里，只要是没有被注释的就是默认值

        [root@baobao ~]# vim /etc/ssh/sshd_config
    
    ……………………………………………………
    
    #Port 22   #也可以设置多个端口只要添加一行然后重启即可（不建议）
    
    #ListenAddress 0.0.0.0 #默认监听所有网卡的接口，如果想指定后面直接写ip即可
    
    Protocol 2 #ssh的协议版本
    
    # HostKey for protocol version 1   #下面是不同协议版本的秘钥文件host key
    
    #HostKey /etc/ssh/ssh_host_key
    
    # HostKeys for protocol version 2
    
    #HostKey /etc/ssh/ssh_host_rsa_key
    
    #HostKey /etc/ssh/ssh_host_dsa_key
    
    SyslogFacility AUTHPRIV #ssh登陆记录，默认是/var/log/secure
    
    #LoginGraceTime 2m  #登陆超时设置
    
    #PermitRootLogin yes#是否允许root登陆，默认是允许的，建议设置为no
    
    #StrictModes yes#是否让sshd检查相关权限以免用户将某些权限设置错误
    
    PasswordAuthentication yes  #密码验证，当然需要了
    
    #PermitEmptyPasswords no#是否允许空密码登陆，当然是no了
    
    #IgnoreUserKnownHosts no#是否忽略用户主文件记录，当然是no了
    
    #IgnoreRhosts yes   #是否取消~/.ssh/.rhosts认证，当然yes了
    
    ChallengeResponseAuthentication no  #该认证不安全，设置为no即可
    
    UsePAM yes#最好使用该认证模块记录与管理，所以yes
    
    #PrintLastLog yes #显示上次登陆的信息
    
    #TCPKeepAlive yes #网络不稳定时为了连接不中断可以设置为no
    
    #UsePrivilegeSeparation yes   #使用权限较低的程序来给用户操作
    
    DenyUsers #拒绝登陆的用户
    
    DenyGroups#拒绝登陆的组

基本上ssh的默认设置已经就很安全了，不过还是建议将root的登陆权限取消，并将ssh的版本设置为2，而且通常这个文件不需要修改，如果修改了需要重启sshd

l  制作不用密码接口登陆的ssh用户

将客户端产生的key复制到服务器中，以后客户端再次登录的时候由于两者在ssh要连接的信号传递中已经比对过key了，所以不再需要输入密码了

1、步骤一，客户端建立两把钥匙

    [root@abao ~]# useradd abao
    
    [root@abao ~]# passwd abao
    
    更改用户 abao 的密码 。
    
    新的 密码：
    
    重新输入新的 密码：
    
    passwd： 所有的身份验证令牌已经成功更新。
    
    服务器上也做上面相同的用户配置
    
    [root@abao ~]# su – abao
    
    [abao@abao ~]$ ssh-keygen#默认以RSA建立两把钥匙
    
    Generating public/private rsa key pair.
    
    Enter file in which to save the key (/home/abao/.ssh/id_rsa): 回车
    
    Created directory ‘/home/abao/.ssh’. #建立主目录
    
    Enter passphrase (empty for no passphrase): 回车
    
    Enter same passphrase again: 回车
    
    Your identification has been saved in /home/abao/.ssh/id_rsa.   #私钥文件
    
    Your public key has been saved in /home/abao/.ssh/id_rsa.pub.   #公钥文件
    
    The key fingerprint is:
    
    89:1e:87:c9:a8:68:69:df:bd:75:a4:df:54:37:70:f1 abao@abao
    
    The key’s randomart image is:
    
    +–[ RSA 2048]—-+
    
    |   . |
    
    |o|
    
    | . .E|
    
    | o + .o  |
    
    |. * S  .   o.|
    
    | … . o  o   . o|
    
    |.+.   .  o . .  |
    
    |o . . . . o o|
    
    |   . . o.  . .   |
    
    +—————–+
    
    [abao@abao ~]$ ls -ld ~/.ssh; ls -l ~/.ssh
    
    drwx——. 2 abao abao 4096 9月  30 11:56 /home/abao/.ssh
    
    总用量 8
    
    -rw——-. 1 abao abao1675 9月  30 11:56 id_rsa
    
    -rw-r–r–. 1 abao abao  391 9月  3011:56 id_rsa.pub

默认情况下建立私钥后权限和文件名放置位置都是正确的。身份必须是abao，当执行ssh-keygen的时候才会在用户主目录下生成两把钥匙，需要注意的是~/.ssh/目录必有700的权限，而且私钥文件的权限必须是-rw——-且属于abao才行，否则在秘钥比对中会被误判为危险而无法成功的以公私钥成对文件的机制实现连接

2、步骤二，将公钥文件数据上传到服务器

    [root@baobao ~]# useradd abao  #先在ssh服务器端建立上传文件账户abao
    [root@baobao ~]# passwd abao
    更改用户 abao 的密码 。
    新的 密码：
    重新输入新的 密码：
    passwd： 所有的身份验证令牌已经成功更新。
    [abao@abao ~]$ scp ~/.ssh/id_rsa.pub abao@172.168.72.68:~
    
    abao@172.168.72.68’s password:
    
    id_rsa.pub100%  3910.4KB/s   00:00

3、步骤三，蒋公钥放置到服务器端的正确目录与文件名

1)   服务器上建立文件~/.ssh

    [root@baobao ~]# su – abao
    
    [abao@baobao ~]$ ls -ld .ssh
    
    ls: 无法访问.ssh: 没有那个文件或目录
    
    [abao@baobao ~]$ mkdir .ssh; chmod 700 .ssh  #注意其权限必须是700
    
    [abao@baobao ~]$ ls -ld .ssh
    
    drwx——. 2 abao abao 4096 9月  3012:30 .ssh

2)   将公钥文件内的数据使用cat转存到authorized_keys内

    [abao@baobao ~]$ ls -l *pub
    
    -rw-r–r–. 1 abao abao 391 9月  3012:26 id_rsa.pub
    
    [abao@baobao ~]$ cat id_rsa.pub >> .ssh/authorized_keys
    
    [abao@baobao ~]$ chmod 644 .ssh/authorized_keys
    
    [abao@baobao ~]$ ls -l .ssh
    
    总用量 4
    
    -rw-r–r–. 1 abao abao 391 9月  3012:39 authorized_keys

总结：客户端必须制作出两把钥匙，其中私钥必须放到~/.ssh内；而公钥必须上传到服务器端并且放置到用户主目录下的~/.ssh/authorized_keys，同时目录的权限必须是700,而文件权限必须是644

4、步骤四，验证

    [root@abao ~]# ssh abao@172.168.72.68
    
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    
    @WARNING: REMOTE HOSTIDENTIFICATION HAS CHANGED! @
    
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    
    IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
    
    Someone could be eavesdropping on you right now(man-in-the-middle attack)!
    
    It is also possible that the RSA host key has just been changed.
    
    The fingerprint for the RSA key sent by the remote host is
    
    17:7e:8e:e9:fd:df:1c:e5:c9:9d:cd:30:31:5e:a6:45.
    
    Please contact your system administrator.
    
    Add correct host key in /root/.ssh/known_hosts to get rid ofthis message.
    
    Offending key in /root/.ssh/known_hosts:1
    
    RSA host key for 172.168.72.68 has changed and you haverequested strict checking.
    
    Host key verification failed.
    
    [root@abao ~]##看不再需要密码了
    
    [root@abao ~]# ifconfig   #查看服务器IP地址
    
    eth0  Linkencap:Ethernet  HWaddr00:0C:29:59:D9:E6
    
    inet addr:172.168.68.72  Bcast:172.168.255.255  Mask:255.255.0.0
    
    inet6 addr:fe80::20c:29ff:fe59:d9e6/64 Scope:Link
    
    UP BROADCASTRUNNING MULTICAST  MTU:1500  Metric:1
    
    RXpackets:239668 errors:0 dropped:0 overruns:0 frame:0
    
    TX packets:855errors:0 dropped:0 overruns:0 carrier:0
    
    collisions:0txqueuelen:1000
    
    RXbytes:18091942 (17.2 MiB)  TX bytes:71093(69.4 KiB)

l  Ssh的安全设置

Sshd所谓的安全其实指的是它的数据加密功能，而对于sshd本身这个服务来说是很不安全的，所以如果不是特别需要请尽量限制在小范围内的几个ip或主机名即可

1、 服务器本身的设置强化/etc/ssh/sshd——config


    [root@baobao ~]# vim /etc/ssh/sshd_config

1)   禁止root账号使用sshd服务

    PermitRootLogin no                         #去掉注释并修改为no

2)   禁止nossh这个组的用户使用sshd服务

    DenyGroups nosh
    
3)   禁止用户testssh使用sshd服务

    DenyUsers testssh
    
    [root@abao ~]# /etc/init.d/sshd restart
    
    停止 sshd：[确定]
    
    正在启动 sshd：[确定]
    
    [root@abao ~]# cat /var/log/secure #验证上述用户不能登陆后查看其日志
    
2、TCP Wrapper的使用

    [root@baobao ~]# vim /etc/host.allow  #只允许内网和本机可以远程登陆
    
    sshd: 127.0.0.1 192.168.1.0/255.255.255.0192.168.10.0/255.255.255.0
    
    [root@baobao ~]# vim /etc/host.deny
    
    sshd: all

3、iptables数据包过滤防火墙

    [root@baobao ~]# iptables -A INPUT -i eth0 -s 192.168.1.0/24 -p tcp –dport 22 -j ACCEPT
    
    [root@baobao ~]# iptables -A INPUT -i eth0 -s 192.168.10.0/24 -p tcp –dport 22 -j ACCEPT
    
    [root@baobao ~]# /etc/init.d/iptables save
    
    [root@baobao ~]# /etc/init.d/iptables restart

注意不要开放ssh的登陆权限给Internet上面的所有用户或主机，只开放给适当的部分用户或主机即可，否则会很不安全