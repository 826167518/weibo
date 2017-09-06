---
title: centos 7 部署 open-falcon 0.2.1
date: 2017-08-31
tags:
---
## 环境准备

### 更换阿里yum
<!--more-->
 步骤：  
1）下载wget  

    yum install -y wget
  
2）备份默认的yum  

    mv /etc/yum.repos.d /etc/yum.repos.d.backup
  
3）设置新的yum目录  

    mkdir /etc/yum.repos.d  

4）下载阿里yum配置到该目录中   

    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

5）重建缓存  

    yum clean all  
    yum makecache  

6）升级所有包（改变软件设置和系统设置，系统版本内核都升级，故需要几分钟耐心等待）  

    yum update -y  

###  安装vim  

    yum install -y vim  

###  安装git  

    yum install -y git  

安装结束后安全起见，确认是否满足官方要求的Git >= 1.7.5  

    git version

###  安装go语言环境（因为官方yum和阿里yum都没有go的安装包，故只能通过fedora的epel仓库来安装）

    yum install -y epel-release
    yum install golang -y

安装结束后安全起见，确认是否满足官方要求的Go >= 1.6  

    go version

###  安装redis

由于部署go时已经安装了epel，故直接执行下面的安装命令（如果没有装epel，会提示No package redis available，也就是没有安装包可用，因为官方yum和阿里yum都没有redis，故只能通过fedora的epel仓库来安装）  

    yum install redis -y

启动redis  

    systemctl start redis  

设置redis开机启动   

    systemctl enable redis

可以用下面的语句查看redis是否开启  

    systemctl status redis  

###  安装mysql

 步骤：  

1）下载repo源

    wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm  

2）安装该rpm包（安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo）  

    rpm -ivh mysql-community-release-el7-5.noarch.rpm  

3）安装mysql  

    yum install mysql-server -y  

4）启动mysql  

    systemctl start mysql  

可以用下面的语句查看mysql是否开启  

    systemctl status mysql  

###  设置环境变量GOROOT和GOPATH  

    export GOROOT=/usr/lib/golang
    export GOPATH=/home  

###  将open-falcon的源码从github上get下来  

步骤：

1）创建GOPATH下的一个本地的路径  

    mkdir -p $GOPATH/src/github.com/open-falcon  

2）进入该路径  

    cd $GOPATH/src/github.com/open-falcon  

3）将源码get到本地  

    git clone https://github.com/open-falcon/falcon-plus.git

###  初始化数据库

    cd $GOPATH/src/github.com/open-falcon/falcon-plus/scripts/mysql/db_schema/
    mysql -h 127.0.0.1 -u root -p < 1_uic-db-schema.sql
    mysql -h 127.0.0.1 -u root -p < 2_portal-db-schema.sql
    mysql -h 127.0.0.1 -u root -p < 3_dashboard-db-schema.sql
    mysql -h 127.0.0.1 -u root -p < 4_graph-db-schema.sql
    mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql

再运行“mysql -h..................”时会提示“Enter password”，如果mysql的root没有设置密码，回车即可。


###  编译源码并打包  
 
步骤：  

1）进入本地源码路径下  

    cd $GOPATH/src/github.com/open-falcon/falcon-plus/

2）使用go get获取rrdtool工具包（make过程卡壳的一个点）  
 
    go get github.com/open-falcon/rrdlite
    
这一步是官方教程没有提到的内容，如果不获取该工具包make的时候会报错  


3）编译所有模块  

    make all

 4）打包
  
    make pack  

在$GOPATH/src/github.com/open-falcon/falcon-plus/目录下就多了刚才的压缩包“open-falcon-v0.2.0.tar.gz”。

###  官方提供的安装包  

https://book.open-falcon.org/zh_0_2/quick_install/prepare.html中官方有提供编译包，如果编译过程不顺利可以直接下载编译包。

## 部署后端

###  创建工作目录

    export WORKSPACE=/home/work
    mkdir -p $WORKSPACE

###  解压二进制包（包名根据实际进行修改） 

由于我是根据本教程编译源码获得的压缩包，故需要切换到“$GOPATH/src/github.com/open-falcon/falcon-plus/”路径下。

包名由于make pack的时候就是open-falcon-v0.2.0.tar.gz，具体根据实际情况。

    cd $GOPATH/src/github.com/open-falcon/falcon-plus/
    tar -xzvf open-falcon-v0.2.0.tar.gz -C $WORKSPACE

###  修改配置文件cfg.json

猜测部分模块依赖连接数据库，因为如果不修改配置文件，aggregator模块会出现无法启动，graph、hbs、nodata、api、alarm模块会出现开启不报错但是状态为开启失败的情况。（个人认为这块的设计值得作为open-falcon优化的一个点，连接本机mysql如果失败是可以收到错误提示的，第一时间有报错提示总比什么都不显示或显示开启但实际开启失败强，如果别人服务都不知道怎么开起来，系统功能再强大有多少人硬着头皮部署下去而不是选择换个系统试试呢）    

如果需要每个模块都能正常启动，需要将上面模块的cfg.json的数据库信息进行修改。根据本教程的配置，需要修改配置文件所在的目录：   

模块	配置文件所在路径  

aggregator    	/home/work/aggregator/config/cfg.json  
graph         	/home/work/graph/config/cfg.json  
hbs	            /home/work/hbs/config/cfg.json  
nodata      	/home/work/nodata/config/cfg.json  
api	            /home/work/api/config/cfg.json  
alarm       	/home/work/alarm/config/cfg.json  

1）修改aggregator的配置文件  

    vim /home/work/aggregator/config/cfg.json


mysql的root密码为空，则去掉“password”，若不为空，则用root密码替换“password”。   

2）修改graph的配置文件  

    vim /home/work/graph/config/cfg.json    


mysql的root密码为空，则去掉“password”，若不为空，则用root密码替换“password”。  

3）修改hbs的配置文件

    vim /home/work/hbs/config/cfg.json  


mysql的root密码为空，则去掉“password”，若不为空，则用root密码替换“password”。  

4）修改nodata的配置文件  

    vim /home/work/nodata/config/cfg.json  


mysql的root密码为空，则去掉“password”，若不为空，则用root密码替换“password”。  

5）修改api的配置文件  

    vim /home/work/api/config/cfg.json  


mysql的root密码为空，则去掉“password”，若不为空，则用root密码替换“password”。

6）修改alarm的配置文件  

    vim /home/work/alarm/config/cfg.json  
 

mysql的root密码为空，则去掉“password”，若不为空，则用root密码替换“password”。   

###  启动后端模块
  
    cd $WORKSPACE
    ./open-falcon start  

可以用下面的命令检查各个模块的启动情况  

    ./open-falcon check  

更多命令的用法（命令的例子是启动agent模块）

    ./open-falcon [start|stop|restart|check|monitor|reload] module
    ./open-falcon start agent
    ./open-falcon check


        falcon-graph         UP           53007
          falcon-hbs         UP           53014
        falcon-judge         UP           53020
     falcon-transfer         UP           53026
       falcon-nodata         UP           53032
    falcon-aggregator         UP           53038
        falcon-agent         UP           53044
      falcon-gateway         UP           53050
          falcon-api         UP           53056
        falcon-alarm         UP           53063

    For debugging , You can check $WorkDir/$moduleName/log/logs/xxx.log  

## 部署前端

### 创建工作目录   

    export FRONTSPACE=/home/front/open-falcon
    mkdir -p $FRONTSPACE

###  获取前端代码

    cd $FRONTSPACE
    git clone https://github.com/open-falcon/dashboard.git

###  安装依赖包

    yum install -y python-virtualenv
    yum install -y python-devel
    yum install -y openldap-devel
    yum install -y mysql-devel
    yum groupinstall "Development tools" -y

    cd $FRONTSPACE/dashboard/
    virtualenv ./env

    ./env/bin/pip install -r pip_requirements.txt

###  修改配置

根据本次记录的配置，dashboard的配置文件在/home/front/open-falcon/dashboard/rrd/config.py，需要根据实际情况对内部配置进行修改。  

由于前端后台搭在一台虚拟机里，且暂时不接入LDAP，且数据库root的密码为空，故先不修改配置文件。

###  开启8081端口  

1）防火墙添加8081端口永久开放

    firewall-cmd --add-port=8081/tcp --permanent    

2）重新载入防火墙配置 
  
    firewall-cmd --reload

###  在生产环境启动

    bash control start

由于虚拟机ip配置为192.168.3.1，故在浏览器中输入192.168.3.1:8081后跳转。

###  以开发者模式启动 

    ./env/bin/python wsgi.py

## 安装agent端

###  创建agent安装目录

    mkdir -p $GOPATH/src/github.com/open-falcon
    cd $GOPATH/src/github.com/open-falcon

###  从git上下载agent分支

    git clone https://github.com/open-falcon/agent.git

###  源码编译安装启动

    cd agent
    go get ./...
    ./control build
    ./control start
    ./control pack

最后一步会pack出一个tar.gz的安装包，拿着这个包去部署服务即可。需要注意的是在源码编译时：

1、需要主机配置GOPATH环境变量（一般可以配置为用户家家目录）；

2、需要主机可以连接外网，通过go get下载相关源码包。

3、编译pack 出的包，在其他agent主机上部署时，无需连接外网 ，pack出的包，可以类似的理解为由c源代码编译后得出的二进制文件。

### 配置说明

配置文件必须叫cfg.json，可以基于cfg.example.json修改，默认该文件并不存在，通过./control start时自动会从cfg.example.json复制一份为cfg.json 。

> {  
    "debug": true,  
    "hostname": "",  
    "ip": "",  
    "plugin": {  
        "enabled": false, # 默认不开启插件机制  
        "dir": "./plugin",  
        "git": "https://coding.net/ulricqin/plugin.git",  
        "logs": "./logs"  
    },  
    "heartbeat": {  
        "enabled": true, # 此处enabled要设置为true  
        "addr": "127.0.0.1:6030", # hbs的地址，端口是hbs的rpc端口  
        "interval": 60,  
        "timeout": 1000  
    },  
    "transfer": {  
        "enabled": true, # 此处enabled要设置为true  
        "addr": "127.0.0.1:8433", # transfer的地址，端口是transfer的rpc端口  
        "interval": 60,  
        "timeout": 1000  
    },  
    "http": {  
        "enabled": true,  
        "listen": ":1988"  
    },  
    "collector": {  
        "ifacePrefix": ["eth", "em"] # 默认配置只会采集网卡名称前缀是eth、em的网卡流量，配置为空就会采集所有的，lo的也会采集。可以从/proc/net/dev看到各个网卡的流量信息  
    },  
    "ignore": { # 默认采集了200多个metric，可以通过ignore设置为不采集  
        "cpu.busy": true,  
        "mem.swapfree": true  
    }  
}  

###  进程管理

    ./control start 启动进程
    ./control stop 停止进程
    ./control restart 重启进程
    ./control status 查看进程状态
    ./control tail 用tail -f的方式查看var/app.log

验证 

看var目录下的log是否正常，或者浏览器访问其1988端口。另外agent提供了一个--check参数，可以检查agent是否可以正常跑在当前机器上。  

    ./falcon-agent --check

打url  http://IP:1988可以查看相关监控信息。  
