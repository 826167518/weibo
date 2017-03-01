---
title: keepalive+nginx实现高可用
date: 2016-09-02
tags:
---
1安装相应基础服务

    yum install openssl-devel
    yum install popt-devel
2下载并安装keepalive安装包
    wget http://www.keepalived.org/software/keepalived-1.2.12.tar.gz
    tar xzf keepalived-1.2.12.tar.gz;
    cd keepalived-1.2.12
    ./configure –prefix=/usr/local/keepalived-1.2.12;
    make && make install
<!--more-->
3制作keepalive服务
    cp /usr/local/keepalived-1.2.12/etc/rc.d/init.d/keepalived /etc/init.d/
    cp /usr/local/keepalived-1.2.12/etc/sysconfig/keepalived /etc/sysconfig/
    chmod +x /etc/init.d/keepalived;
    chkconfig –add keepalived;
    mkdir -p /etc/keepalived
    ln -s /usr/local/keepalived-1.2.12/sbin/keepalived /usr/sbin/
    cp /usr/local/keepalived-1.2.12/etc/keepalived/keepalived.conf /etc/keepalived/
    ln -s /usr/local/keepalived-1.2.12/sbin/keepalived /sbin/
    service keepalived restart
4更改keepalive的配置文件
    vim /etc/keepalived/keepalived.conf
    keepalive主
    ! Configuration File for keepalived
    
    global_defs {
    
       router_id nginx_master
    
    }
    
    #监控服务.NGINX mysql等
    
    vrrp_script chk_nginx {
    
    script “/usr/local/nginx/check_nginx.sh”
    
    interval 2
    
    weight 2
    
    }
    
    vrrp_instance VI_1 {
    
    state MASTER
    
    interface eth0
    
    virtual_router_id 51   #通道
    
    priority 101#优先级，数值越高优先级越高
    
    advert_int 1
    
    authentication {
    
    auth_type PASS
    
    auth_pass 1111
    
    }
    
    virtual_ipaddress {
    
    192.168.1.254   #虚拟IP
    
    }
    
    track_script {
    
    chk_nginx  #检测脚本 上面配置的
    
    }

 

keepalive从

    ! Configuration File for keepalived
     
    global_defs {
       router_id nginx_backup
    }
    #监控服务.NGINX mysql等
    vrrp_script chk_nginx {
    script “/usr/local/nginx/check_nginx.sh”
    interval 2
    weight 2
    }
     
    vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51#通道
    priority 99#优先级，数值越高优先级越高
    advert_int 1
    authentication {
    auth_type PASS
    auth_pass 1111
    }
    virtual_ipaddress {
    192.168.1.254   #虚拟IP
    }
    track_script {
    chk_nginx  #检测脚本 上面配置的
    }
    }
5脚本/usr/local/nginx/check_nginx.sh”内容：
    #!/bin/bash
    
    if [ “$(ps -ef | grep “nginx: master process“| grep -v grep )” == “” ]
    
    then
    
    /usr/local/nginx/sbin/nginx
    
    sleep 5
    
    if [ “$(ps -ef | grep “nginx: master process“| grep -v grep )” == “” ]
    
    then
    
    killall keepalived
    
    fi
    
    fi

6编写keepalive和nginx共存脚本

    vim /data/apps/jiance.sh
    
    #!/bin/bash
    while :
    do
    nginxpid=`ps -C nginx –no-header |wc -l`
    if [ $nginxpid -eq 0 ];then
       /etc/init.d/nginx restart
      sleep 5
    nginxpid=`ps -C nginx –no-header |wc -l`
      if [ $nginxpid -eq 0 ];then
      /etc/init.d/keepalived stop
      fi
    fi
    sleep 5
    done
    }
7把该脚本制作成系统服务并且开机启动
    chmod 755 /data/apps/jiance.sh
    vim /etc/init.d/jiance
    #!/bin/bash
    # chkconfig: 2345 10 90
    # description: jiance ….
    start() {
    echo “Starting my process “
    cd /data/apps/
    ./jiance.sh
    }
    stop() {
    killall jiance.sh
    echo “Stoped”
    }
    chmod a+wrx /etc/init.d/jiance
    /etc/init.d/jiance start
    chmod +x jiance   #增加执行权限
    chkconfig –add jiance #把jiance添加到系统服务列表
    chkconfig jiance on #设定jiance的开关（on/off）
    chkconfig –list jiance   #就可以看到已经注册了jiance的服务
完成如上步骤keepalive+nginx高可用即搭建完成。