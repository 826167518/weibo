---
title: mysql主从配置文件内容
date: 2016-09-05
tags:
---

mysql简要配置文件---master
<!--more-->
    [client]  
    port = 3306  
    default-character-set=utf8  
    [mysqld]  
    server-id=10  
    log-bin=mysql-master-bin  
    binlog_format = mixed  
    expire_logs_days=15  
    max_connections=10000  
    innodb_flush_log_at_trx_commit=1  
    sync_binlog=1  
    binlog-ignore-db=mysql,test,information_schema  
    skip-name-resolve  
    port = 3306  
    key_buffer_size = 16M  
    max_allowed_packet = 16M  
    join_buffer_size = 512M  
    sort_buffer_size = 256M  
    read_rnd_buffer_size = 128M  
    innodb_buffer_pool_size = 4096M  
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES  
    [mysqld_safe]  
    default-character-set=utf8mb4  
    


mysql简要配置文件---slave
    [client]   
    port = 3306  
    default-character-set=utf8  
    [mysqld]  
    server-id=100  
    #log-bin=mysql-slave-bin  
    relay-log=mysqld-relay-bin  
    max_binlog_size = 1000M  
    binlog_format = mixed  
    expire_logs_days=7  
    innodb_flush_log_at_trx_commit=1  
    sync_binlog=1  
    read_only=1  
    binlog-ignore-db=mysql,test,information_schema  
    skip-name-resolve  
    max_connections=10000  
    max_user_connections=490  
    max_connect_errors=2  
    key_buffer_size = 16M  
    max_allowed_packet = 16M  
    join_buffer_size = 512M  
    sort_buffer_size = 256M  
    read_rnd_buffer_size = 128M  
    innodb_buffer_pool_size = 4096M  
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES  
    [mysqld_safe]  
    default-character-set=utf8mb4  