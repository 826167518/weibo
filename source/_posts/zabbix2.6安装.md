---
title: zabbix2.6安装
date: 2016-09-02
tags:
---


1. 安装lnmp架构
 `yum -y install gcc gcc-c++ autoconf httpd php mysql mysql-server php-mysql httpd-manual mod_ssl mod_perl mod_auth_mysql php-gd php-xml php-mbstring php-ldap php-pear php-xmlrpc php-bcmath mysql-connector-odbc mysql-devel libdbi-dbd-mysql net-snmp-devel curl-devel`
2. 启动服务
    service mysqld start
    service httpd start
<!--more-->

3. 创建zabbix用户和组
    groupadd zabbix
    useradd zabbix -g zabbix
4. 进入mysql创建数据库
    create database zabbix character set utf8;
    grant all on zabbix.* to zabbix@localhost identified by ‘jszj201501’;
5. 解压zabbix.tar包
    > tar zxf zabbix-2.4.tar.gz
    > cd zabbix-2.4.5/database/mysql/
6. 导入数据库
    mysql -uzabbix -pjszj201501 zabbix <schema.sql
    mysql -uzabbix -pjszj201501 zabbix <images.sql
7. 进行编译安装
    cd ../..
    ./configure –prefix=/usr/local/zabbix –enable-server –enable-agent –with-mysql –with-net-snmp –with-libcurl
    make&&make install
8. 添加zabbix服务对应的端口
    cat >>/etc/services<<EOF
    zabbix-agent 10050/tcp Zabbix Agent
    zabbix-agent 10050/udp Zabbix Agent
    zabbix-trapper 10051/tcp Zabbix Trapper
    zabbix-trapper 10051/udp Zabbix Trapper
    EOF
9. 修改zabbix server 配置文件
    vim /usr/local/zabbix/etc/zabbix_server.conf
    LogFile=/tmp/zabbix_server.log ##日志位置，根据需求修改；
    PidFile=/tmp/zabbix_server.pid ##PID 所在位置
    DBHost=localhost ##如果不是在本机，请修改
    DBName=zabbix ##数据库名称
    DBUser=zabbix ##数据库用户名
    DBPassword=redhat ##数据库密码
10. 安装启动脚本,添加可执行权限
    cp misc/init.d/fedora/core/zabbix_server /etc/init.d
    chmod +x /etc/init.d/zabbix_server
11. 查找zabbix_server.conf位置复制

    find / -name zabbix_server.conf
12. 修改启动脚本，启动zabbix server
    vim /etc/init.d/zabbix_server
    BASEDIR=/usr/local/zabbix ##修改这个，zabbix 的安装目录
    CONFILE=$BASEDIR/etc/zabbix_server.conf ##添加这一行，定义配置文件位置
    #搜索start,修改启动选项，默认是去/etc 下去找配置文件的
    action $”Starting $BINARY_NAME: ” $FULLPATH -c $CONFILE
    service zabbix_server start
13. 安装邮件服务
    yum install mailx
    vi /etc/mail.rc
    set from=xxx@163.com smtp=smtp.163.com
    set smtp-auth-user=xxx@163.com smtp-auth-password=123456
    set smtp-auth=login
    :wq! #保存退出
    echo “zabbix test mail” |mail -s “zabbix” yyy@163.com
    linux客户端
    mkdir /usr/local/zabbix
    tar zxf zabbix_agents_2.0.6.linux2_6.amd64.tar.gz -C /usr/local/zabbix
14. 编辑配置文件
    find / -name zabbix_agentd.conf
    vim zabbix_agentd.conf
    LogFile=/tmp/zabbix_agentd.log
    Server=202.108.1.52 ##服务器IP
    ServerActive=202.108.1.52 ##主动模式服务器IP
    Hostname=202.108.1.51 ##设定主机名
    #加入mysql配置
15. 安装修改启动脚本
    scp misc/init.d/fedora/core/zabbix_agentd 202.108.1.51:/etc/init.d
    vim /etc/init.d/zabbix_agentd
    BASEDIR=/usr/local/zabbix ##修改这个
    CONFILE=$BASEDIR/etc/zabbix_agentd.conf ##添加这行，搜索start 添加-c $CONFILE
    action $”Starting $BINARY_NAME: ” $FULLPATH -c $CONFILE
    service zabbix_agentd start
16. 创建用户和用户组
    groupadd zabbix
    useradd zabbix -g zabbix
    windows客户端
    cmd
    d:\zabbix_agentd.exe -i -c d:\zabbix\zabbix_agentd.conf
    services.msc
17. 禁用内部邮件服务
    service sendmail stop #关闭
    chkconfig sendmail off #禁止开机启动
    service postfix stop
    chkconfig postfix off
18. 事件触发器配置：
    名称：Action-Email
    默认接收人：故障{TRIGGER.STATUS},服务器:{HOSTNAME1}发生: {TRIGGER.NAME}故障!
    默认信息：
    告警主机:{HOSTNAME1}
    告警时间:{EVENT.DATE} {EVENT.TIME}
    告警等级:{TRIGGER.SEVERITY}
    告警信息: {TRIGGER.NAME}
    告警项目:{TRIGGER.KEY1}
    问题详情:{ITEM.NAME}:{ITEM.VALUE}
    当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
    事件ID:{EVENT.ID}
    恢复信息：打钩
    恢复主旨：恢复{TRIGGER.STATUS}, 服务器:{HOSTNAME1}: {TRIGGER.NAME}已恢复!
    恢复信息：
    告警主机:{HOSTNAME1}
    告警时间:{EVENT.DATE} {EVENT.TIME}
    告警等级:{TRIGGER.SEVERITY}
    告警信息: {TRIGGER.NAME}
    告警项目:{TRIGGER.KEY1}
    问题详情:{ITEM.NAME}:{ITEM.VALUE}
    当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
    事件ID:{EVENT.ID}
    已启用：打钩
19. 解决乱码和附件问题
    vim /usr/local/zabbix/share/zabbix/alertscripts/sendmail.sh
    #!/bin/bash
    #export.UTF-8 //解决发送的中文变成了乱码的问题
    FILE=/tmp/mailtmp.txt
    echo “$3” >$FILE
    dos2unix -k $FILE  //解决了发送的邮件内容变成附件的问题。
    /bin/mail -s “$2” $1 < $FILE
    touch /tmp/mailtmp.txt
    chown  zabbix.zabbix /tmp/mailtmp.txt
20. zabbix  mysql客户端
    find / -name userparameter_mysql.conf
    vi /usr/local/zabbixvim/etc/zabbix_agent.conf
    Include=/usr/local/zabbix/conf/zabbix_agentd/ #加入mysql配置(find查找的路径)
    vi /usr/local/zabbix/etc/zabbix_agentd.conf
    Include=/usr/local/zabbix/conf/zabbix_agentd/ #加入mysql配置
    mkdir /etc/zabbix
    touch /etc/zabbix/.my.cnf
    vim /etc/zabbix/.my.cnf
    [mysql]
    host = localhost
    user = mysqlcheck
    password = mysqlcheck
    socket = /var/lib/mysql/mysql.sock(mysql.sock的位置)
    [mysqladmin]
    host = localhost
    user = mysqlcheck
    password = mysqlchechk
    socket = /var/lib/mysql/mysql.sock(mysql.sock的位置)
    vim userparameter_mysql.conf
    UserParameter=mysql.status[*],echo “show global status where Variable_name=’$1′;” | mysql -uzabbix -pjszj201501 -N | awk ‘{print $$2}’ #取mysql状态
    UserParameter=mysql.size[*],echo “select sum($(case “$3″ in both|””) echo “data_length+index_length”;; data|index) echo “$3_length”;; free) echo “data_free”;; esac)) from information_schema.tables$([[ “$1” = “all” || ! “$1″ ]] || echo ” where table_schema=’$1′”)$([[ “$2” = “all” || ! “$2” ]] || echo “and table_name=’$2′”);” | mysql -uzabbix -pjszj201501 -N
     #取mysql操作状态
    UserParameter=mysql.ping,HOME=/etc/zabbix mysqladmin -uzabbix -ppassword | grep -c alive
    UserParameter=mysql.version,mysql -V #取mysql版本
    chmod 777 userparameter_mysql.conf
    service zabbix_agentd restart
21. Zabbix配置email报警
一、              使用msmtp这个命令行MUA
    (1)./configure –prefix=/usr/local/msmtp
    (2)make
    (3)make install
    (4)mkdir /usr/local/msmtp/etc
    (5)touch /usr/local/msmtp/etc/msmtprc
    (6)在/usr/local/msmtp/etc/msmtprc中写入如下内容：
    defaults
    account michael_zhou
    host mail.chinadba.com
    domain chinadba.com
    from michael_zhou@chinadba.com
    auth login
    user michael_zhou@chinadba.com
        password your_password
    account default:michael_zhou
    logfile /var/log/maillog
    (7)测试一下：/usr/local/msmtp/bin/msmtp i@chinadba.com，输入内容后按ctrl+D发出。
二、    在实际测试中发现直接使用msmtp命令发出去的邮件会看不到发件人和主题，只能看到邮件内容，所以我使用mutt挂接在msmtp上，mutt默认会安装，如果没有安装请yum install mutt*
    (1)修改mutt的配置文件/etc/Muttrc, 不是/etc/muttrc  ，M要大写
    1．set sendmail=”/usr/local/msmtp/bin/msmtp”
    2．set use_from=yes
    3．set realname=michael_zhou@chinadba.com  #发件人邮箱地址
    4．set editor=”vi”
    5．保存退出
    (2)测试一下：echo “邮件报警测试” | mutt -s “测试” i@chinadba.com  #收件人地址
三、    创建 zabbix用于发送邮件的脚本,脚本放在什么位置随便，但是要保证zabbix能找到！
(1)vim /usr/bin/baojing,并写入如下内容：
#!/bin/bash
echo “$3” | mutt -s “$2” $1       # $3表示邮件内容、$2表示邮件标题、$1表示收件人
(2)chmod a+x /usr/bin/baojing
四、    zabbix配置
(1)创建meida types
1．登录到zabbix，进入“Administration” >> ”Media types”，点击右上角“Create Media Type”。 Description填”mediatype-baojing”或其它名称，Type选择”Script”，Script填”baojing”。
2．点击save保存
(2)创建actions
1.登录到zabbix，进入”Configation” >> “Actions”，点击右上角”Create Actions”。输入Name “action-baojing” ，其它都默认点击右侧“Action Operations”下的”New”按钮，”Operation Type”选择”Send message”，”Send Message to”选择一个或多个要发送消息的用户组，”Send only to”选择我们之前新增的mediatype-baojing。
2.点击save保存
(3) zabbix用户配置
登录到zabbix, 进入”Adimistration” >> “Users”，在之前选定要发送消息的组里的Members栏位里选择一个用户，例如选择Admin用户。
在用户信息修改界面最下方的”Media”处点击”Add”按钮。
Type选择”mediatype-baojing”，Send to填入收件人地址，点击Add添加。
点击”Save”保存配置。
至此配置完成，测试！
不光是zabbix,nagios等监控平台的邮件报警都可以这样配置。当然转到139邮箱的话可以收到短信的，会更加及时的收到报警。
zabbix企业应用之服务器硬件信息监控
http://dl528888.blog.51cto.com/2382721/1403893
zabbix企业应用之Mysql主从监控
http://dl528888.blog.51cto.com/2382721/1434263
Zabbix监控MySQL数据库状态
http://www.linuxidc.com/Linux/2015-04/116304.htm
Zabbix使用微信接口实现微信报警功能
http://lcbk.net/zabbix/2022.html
http://www.cnyunwei.com/thread-29593-1-1.html