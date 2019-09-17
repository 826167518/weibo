---
title: Jira和Confluence迁移简介
date: 2019-09-17
tags: jira
categories: jira
---

# jira和confluence迁移简介
>环境说明：
>1.mysql5.6。
>2.全部使用Kubelete和docker启动。
>3.挂载方式为本地目录。

<!--more-->
## Jira迁移
### 备份老jira机器上的数据
老环境的web页面中数据进行备份，页面选择Setting>System>Backup system>
备份目录为`/var/atlassian/jira/export` 
将备份的zip包拷贝到新的机器
同时将jira目录拷贝到新的机器上，jira目录为`/var/atlassian/jira/data/attachments`     (此目录是jira上面存储的图片和附件信息)

### 启动Jira跟Mysql
mysql配置文件my.cnf
```
[client]
port            = 3306
default-character-set=utf8
socket          = /var/run/mysqld/mysqld.sock

[mysql]
default-character-set=utf8

[mysqld_safe]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
nice            = 0

[mysqld]
skip-host-cache
skip-name-resolve
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
explicit_defaults_for_timestamp
character-set-server=utf8
default-storage-engine=INNODB
max_allowed_packet=256M
innodb_log_file_size=256M


sql_mode = NO_AUTO_VALUE_ON_ZERO


symbolic-links=0

[mysqld.safe]
default-character-set=utf8
[mysql.server]
default-character-set=utf8

#
!includedir /etc/mysql/conf.d/
```
### 启动Mysql有两种方法，一种是docker run 一种是使用Kubelete启动

docker run方式
```
docker run -d --restart=always --name mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_USER=jira \
  -e MYSQL_PASSWORD=jira \
  -e MYSQL_DATABASE=jiradb \
  -v /data/mysql/config/my.cnf:/etc/mysql/my.cnf \
  -v /data/mysql/data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:5.6
```

Kubelete方式启动
```
apiVersion: app.k8s.io/v1beta1
kind: Application
metadata:
  name: mysql-jira
  namespace: production-business-jira
spec:
  assemblyPhase: Succeeded
  componentKinds:
    - group: apps
      kind: Deployment
    - group: ''
      kind: Service
  descriptor: {}
  selector:
    matchLabels:
      app.io/name: mysql-jira.production-business-jira
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app.io/name: mysql-jira.production-business-jira
  name: mysql-jira-mysql
  namespace: production-business-jira
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.io/name: mysql-jira.production-business-jira
      project.io/name: production-business
      service.io/name: deployment-mysql-jira-mysql
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: deployment-mysql-jira-mysql
        app.io/name: mysql-jira.production-business-jira
        project.io/name: production-business
        service.io/name: deployment-mysql-jira-mysql
        version: v1
    spec:
      containers:
        - env:
            - name: MYSQL_ROOT_PASSWORD
              value: root
            - name: MYSQL_USER
              value: jira
            - name: MYSQL_PASSWORD
              value: jira
            - name: MYSQL_DATABASE
              value: jiradb
          image: 'mysql:5.6'
          imagePullPolicy: IfNotPresent
          name: mysql
          resources:
            limits:
              cpu: '4'
              memory: 4Gi
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-data
            - mountPath: /etc/mysql/my.cnf
              name: mysql-cnf
              readOnly: true
              subPath: my.cnf
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/hostname: slave-4
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
        - operator: Exists
      volumes:
        - hostPath:
            path: /data/mysql/data
            type: ''
          name: mysql-data
        - configMap:
            defaultMode: 420
            items:
              - key: my.cnf
                path: my.cnf
            name: mysql
          name: mysql-cnf
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.io/name: mysql-jira.production-business-jira
  name: mysql-jira-mysql
  namespace: production-business-jira
spec:
  ports:
    - name: tcp-3306-3306
      port: 3306
      protocol: TCP
      targetPort: 3306
  selector:
    app.io/name: mysql-jira.production-business-jira
    service.io/name: deployment-mysql-jira-mysql
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  type: ClusterIP
```

### 启动Jira
Jira启动也是分为两种，一种是docker run另外一种是Kubelete方式启动
docker run方式
```
docker run -d  --restart=always --name jira -p 8080:8080 \
  -e JVM_MAXIMUM_MEMORY=6144m \
  -e JAVA_OPTS="-javaagent:/opt/atlassian-agent.jar" \
  -v /root/atlassian-agent.jar:/opt/atlassian-agent.jar \
  -v /data/jira/data:/var/atlassian/jira \
  -v /data/jira/logs:/opt/atlassian/jira/logs \
  cptactionhank/atlassian-jira
```
atlassian-agent.jar是破解包，详情请见https://gitee.com/pengzhile/atlassian-agent

Kubelete方式
```
apiVersion: app.k8s.io/v1beta1
kind: Application
metadata:
  annotations:
  labels:
  name: jira
  namespace: production-business-jira
spec:
  assemblyPhase: Succeeded
  componentKinds:
    - group: apps
      kind: Deployment
    - group: ''
      kind: Service
  descriptor: {}
  selector:
    matchLabels:
      app.io/name: jira.production-business-jira
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app.io/name: jira.production-business-jira
  name: jira-atlassian-jira
  namespace: production-business-jira
  ownerReferences:
    - apiVersion: app.k8s.io/v1beta1
      blockOwnerDeletion: true
      controller: true
      kind: Application
      name: jira
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.io/name: jira.production-business-jira
      project.io/name: production-business
      service.io/name: deployment-jira-atlassian-jira
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deployment-jira-atlassian-jira
        app.io/name: jira.production-business-jira
        project.io/name: production-business
        service.io/name: deployment-jira-atlassian-jira
        version: v1
    spec:
      hostNetwork: true
      containers:
        - env:
            - name: JAVA_OPTS
              value: '-javaagent:/opt/atlassian-agent.jar'
          image: 'cptactionhank/atlassian-jira'
          imagePullPolicy: IfNotPresent
          name: atlassian-jira
          resources:
            limits:
              cpu: '8'
              memory: 16Gi
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /var/atlassian/jira
              name: new-volume
            - mountPath: /opt/atlassian/jira/logs
              name: new-volume1
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/hostname: slave-4
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
        - operator: Exists
      volumes:
        - hostPath:
            path: /data/jira/data
            type: ''
          name: new-volume
        - hostPath:
            path: /data/jira/logs
            type: ''
          name: new-volume1
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.io/name: jira.production-business-jira
  name: jira-atlassian-jira
  namespace: production-business-jira
  ownerReferences:
    - apiVersion: app.k8s.io/v1beta1
      blockOwnerDeletion: true
      controller: true
      kind: Application
      name: jira
spec:
  ports:
    - name: tcp-8080-8080
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app.io/name: jira.production-business-jira
    service.io/name: deployment-jira-atlassian-jira
  sessionAffinity: None
  type: ClusterIP
```
#### 登录Jira开始对Jira进行初始化

1.选择I'll set it up myself     Next
2.选择My Own Database (recommended for production environments)
>Database Type:   MySQL 5.6
>Hostname: mysql-jira-mysql
>Port: 3306
>Database: jiradb
>Username: jira
>Password: jira

选择Test Connection，测试链接成功后点击  Next  然后等待数据库初始化完成
3.填写jira项目信息
>Application Title: jiar
>Mode:Private
>Base URL:http://192.168.8.27:8081

4.破解Jira
复制Server ID内容
然后进入jira容器内

 >docker exec -it jira bash
 cd /opt/
 ls
 `bash-4.4$ ls
atlassian            atlassian-agent.jar`
java -jar atlassian-agent.jar -p jira -m aaa@bbb.com -n my_name -o http://datura.top -s XXXXXX
```
====================================================
=======        Atlassian Crack Agent         =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================

>Your license code(Don't copy this line!!!):

>AAABjQ0ODAoPeJx9klFr2zAUhd/1Kwx7tmuFpXEDgm222TzsJNRpHvYyrp2bRsOWxZWcNvv1lRsX0
>jYEBEJC59xP594v6x69ErXHZ14Yzfl0ziPvZ7H2JiG/Y4+EqPad1khBLmtUBtdHjQtoUcTLokjv4
>+x7zmJCsLJTCVgUg9AP73w+Y1ckCZqapB5U4kE1spUWt15zEnjV0dtbq8385ub/XjYYyI4VIJVFB
>arG9FlLOo7VIldt5hb7JwneKNOtPFkv8qzI1mnCFn1bIS13DwbJCJ+/wV3x0tRt+9oGw8E33c4+A
>WHwyejKW6itPKCw1OO7LM/vxz9vnNtAPGHpAZr+NU+xg8YgW9IjKGlOV0MuLpYt2N6VtZ1mcaesM
>0xdQI0AgG9VVQV1157ALuOeA1zhLy2QRRo5xsSyRORZUqYLP+fT22gWRpxH0+nkXQMu9bxEOiA5+
>Y/4a+yv/mx++2m+ufV/rcrw0qh9buKqp3oPBj8O2rkY3ZSQJmnG7zlQcQF2TO2VsT3+VW5/AS2KC
> fkwLAIUYgOdQfGYSq/19VKXo54j+OohnFoCFDNIF/aherXDRLivmm2xF3bE8S47X02j7
```

Server ID：XXXXXXX
Your License Key:
```
AAABjQ0ODAoPeJx9klFr2zAUhd/1Kwx7tmuFpXEDgm222TzsJNRpHvYyrp2bRsOWxZWcNvv1lRsX0
jYEBEJC59xP594v6x69ErXHZ14Yzfl0ziPvZ7H2JiG/Y4+EqPad1khBLmtUBtdHjQtoUcTLokjv4
+x7zmJCsLJTCVgUg9AP73w+Y1ckCZqapB5U4kE1spUWt15zEnjV0dtbq8385ub/XjYYyI4VIJVFB
arG9FlLOo7VIldt5hb7JwneKNOtPFkv8qzI1mnCFn1bIS13DwbJCJ+/wV3x0tRt+9oGw8E33c4+A
WHwyejKW6itPKCw1OO7LM/vxz9vnNtAPGHpAZr+NU+xg8YgW9IjKGlOV0MuLpYt2N6VtZ1mcaesM
0xdQI0AgG9VVQV1157ALuOeA1zhLy2QRRo5xsSyRORZUqYLP+fT22gWRpxH0+nkXQMu9bxEOiA5+
Y/4a+yv/mx++2m+ufV/rcrw0qh9buKqp3oPBj8O2rkY3ZSQJmnG7zlQcQF2TO2VsT3+VW5/AS2KC
fkwLAIUYgOdQfGYSq/19VKXo54j+OohnFoCFDNIF/aherXDRLivmm2xF3bE8S47X02j7
```
等待初始化Jira

5.创建Jira用户信息
>Full name ：jiraadmin
Email Address :jiraadmin@123.com
Username: jiraadmin
Password: jiraadmin
Confirm Password : jiraadmin
选择 Next
Configure Email Notifications ：Later
选择 Finish
选择语言
之后创建Jira项目

6.恢复Jira数据
将备份的zip包拷贝到新的Jira目录中，目录位置为 
`/var/atlassian/jira/import`
返回jira页面，Setting>System>Restore system
将备份包信息填写到相应位置，License输入上面的key信息
然后点击Restore
等待数据恢复，数据恢复后回到命令行
将老Jira`/var/atlassian/jira/data/attachments`  的目录拷贝到本机的/data/jira/data/data/attachments目录中
拷贝完成后重启Jira容器
重启完成后，此时将可以使用老Jira的用户登录新Jira，到此Jira迁移完成。


## Confluence 迁移简介

### 备份老Confluence数据
登录Confluence，Setting>General Configuration>Backup & Restore
选择Include attachments然后导出或者直接使用Confluence每日自动备份的zip包进行恢复，我这里使用的是每日自动备份的zip包进行恢复的
包路径为`$CONFLUENCE_HOME/backups`
将zip包拷贝到新的Confluence机器上

### 启动Confluence和Mysql
mysql配置文件my.cnf
```
[client]
port            = 3306
default-character-set=utf8
socket          = /var/run/mysqld/mysqld.sock

[mysql]
default-character-set=utf8

[mysqld_safe]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
nice            = 0

[mysqld]
skip-host-cache
skip-name-resolve
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
explicit_defaults_for_timestamp
character-set-server=utf8
default-storage-engine=INNODB
max_allowed_packet=256M
innodb_log_file_size=256M


sql_mode = NO_AUTO_VALUE_ON_ZERO


symbolic-links=0

[mysqld.safe]
default-character-set=utf8
[mysql.server]
default-character-set=utf8

#
!includedir /etc/mysql/conf.d/
```
### 启动mysql有两种方法，一种是docker run 一种是使用Kubelete启动

docker run方式
```
docker run -d --restart=always --name mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_USER=confluence \
  -e MYSQL_PASSWORD=confluence \
  -e MYSQL_DATABASE=confluencedb \  
  -v /data/mysql/data:/var/lib/mysql \
  -v /data/mysql/config/my.cnf:/etc/mysql/my.cnf \
  -p 3306:3306 \
  mysql:5.6
```
Kubelete方式启动
```
apiVersion: app.k8s.io/v1beta1
kind: Application
metadata:
  name: mysql-confluence
  namespace: production-business-confluence
spec:
  assemblyPhase: Succeeded
  componentKinds:
    - group: apps
      kind: Deployment
    - group: ''
      kind: Service
  descriptor: {}
  selector:
    matchLabels:
      app.io/name: mysql-confluence.production-business-confluence
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  generation: 8
  labels:
    app.io/name: mysql-confluence.production-business-confluence
    app.io/uuid: b0353aca-d537-11e9-ab02-5254002bc0d6
  name: mysql-confluence-mysql-confluence
  namespace: production-business-confluence
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.io/name: mysql-confluence.production-business-confluence
      project.io/name: production-business
      service.io/name: deployment-mysql-confluence-mysql-confluence
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deployment-mysql-confluence-mysql-confluence
        app.io/name: mysql-confluence.production-business-confluence
        project.io/name: production-business
        service.io/name: deployment-mysql-confluence-mysql-confluence
        version: v1
    spec:
      containers:
        - env:
            - name: MYSQL_USER
              value: confluence
            - name: MYSQL_PASSWORD
              value: confluence
            - name: MYSQL_DATABASE
              value: confluencedb
            - name: MYSQL_ROOT_PASSWORD
              value: root
          image: 'mysql:5.6'
          imagePullPolicy: IfNotPresent
          name: mysql-confluence
          resources:
            limits:
              cpu: '2'
              memory: 4Gi
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-data
            - mountPath: /etc/mysql/my.cnf
              name: new-volume
              readOnly: true
              subPath: my.cnf
      dnsPolicy: ClusterFirst
      hostNetwork: true
      nodeSelector:
        kubernetes.io/hostname: slave-2
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
        - operator: Exists
      volumes:
        - hostPath:
            path: /data/mysql/data
            type: ''
          name: mysql-data
        - configMap:
            defaultMode: 420
            items:
              - key: my.cnf
                path: my.cnf
            name: confluence-mysql
          name: new-volume
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-confluence
  namespace: production-business-confluence
spec:
  ports:
    - name: tcp-3306-3306
      port: 3306
      protocol: TCP
      targetPort: 3306
  selector:
    app.io/name: mysql-confluence.production-business-confluence
    service.io/name: deployment-mysql-confluence-mysql-confluence
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  type: ClusterIP
```

### 启动Confluence
Confluence启动也是分为两种，一种是docker run另外一种是Kubelete方式启动
docker run方式
```
docker run -d --restart=always --name confluence -p 8090:8090 \
  -e JVM_MAXIMUM_MEMORY=8196m \
  -e JAVA_OPTS="-javaagent:/opt/atlassian-agent.jar" \
  -v /data/confluence/data:/var/atlassian/application-data/confluence \
  -v /root/atlassian-agent.jar:/opt/atlassian-agent.jar \
  -v /data/confluence/logs:/opt/atlassian/confluence/logs \
docker.io/atlassian/confluence-server
```

Kubelete方式启动
```
apiVersion: app.k8s.io/v1beta1
kind: Application
metadata:
  name: confluence
  namespace: production-business-confluence
spec:
  assemblyPhase: Succeeded
  componentKinds:
    - group: apps
      kind: Deployment
    - group: ''
      kind: Service
  descriptor: {}
  selector:
    matchLabels:
      app.io/name: confluence.production-business-confluence
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app.io/name: confluence.production-business-confluence
  name: confluence-confluence
  namespace: production-business-confluence
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.io/name: confluence.production-business-confluence
      project.io/name: production-business
      service.io/name: deployment-confluence-confluence
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deployment-confluence-confluence
        app.io/name: confluence.production-business-confluence
        project.io/name: production-business
        service.io/name: deployment-confluence-confluence
        version: v1
    spec:
      hostNetwork: true
      containers:
        - env:
            - name: JAVA_OPTS
              value: '-javaagent:/opt/atlassian-agent.jar'
          image: 'docker.io/atlassian/confluence-server'
          imagePullPolicy: IfNotPresent
          name: confluence
          resources:
            limits:
              cpu: '4'
              memory: 16Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /var/atlassian/application-data/confluence
              name: confluence-data
            - mountPath: /opt/atlassian/confluence/logs
              name: confluence-log
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/hostname: slave-2
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
        - operator: Exists
      volumes:
        - hostPath:
            path: /data/confluence/data
            type: ''
          name: confluence-data
        - hostPath:
            path: /data/confluence/logs
            type: ''
          name: confluence-log
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.io/name: confluence.production-business-confluence
  name: confluence-confluence
  namespace: production-business-confluence
spec:
  ports:
    - name: tcp-8090-8090
      port: 8090
      protocol: TCP
      targetPort: 8090
  selector:
    app.io/name: confluence.production-business-confluence
    service.io/name: deployment-confluence-confluence
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  type: ClusterIP
```

#### 登录Confluence开始对Confluence进行初始化
选择Confluence Team Calendars
复制Server ID
登录Confluence机器中的Confluence容器内

>docker exec -it confluence bash
 cd /opt/
 ls
 `bash-4.4$ ls
atlassian            atlassian-agent.jar`
java -jar atlassian-agent.jar -p conf -m aaa@bbb.com -n my_name -o http://datura.top -s XXXXXX
```
====================================================
=======        Atlassian Crack Agent         =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================
Your license code(Don't copy this line!!!):
AAABQQ0ODAoPeJxtUNFOgzAUfe9XNPEZXDc3xpImTkCdgWFkm76ZC7tzTaCQtizOr7cDfDFLmjQ9p
/fcc87NpkWaYUOZR0f+YjJfTDz6lGzoeMR8EigEI2oZgkF+QZyR7zCPRCco247hByg1khB1oUTTI
VtZikoY3NNSFCg10vxMj8Y0enF7+3MUJbqiJqn6Ail0L3JhLbkH0ypwTd2QopYHFwojTsiNapEEt
TT2HSUgSg4A93meu0Vd9T8zA8qgGtx0UNwv35wbXEOFPEiTJHoLVsuYWA1pUIIsMPpuhDoP+eY2n
2cPGWZXIY9XYRatnZhNZ3OPTRjzpnczkqE6obL0w/vOc7wPljqP/vLFmYXPyd/wdeXXVhVH0Pi/0
aGqHSp9KWTcZ1i3VY4qPWy1xbnDiPXCr/gZyulyVudPae9f57CZMDAsAhRpli3TWhHdAL0fOkAeh
38ZCSuf+QIUfBreVMIH4k/6isdn0HptYXvU608=X02g0
```

Server ID： XXXXX
Confluence：
```
AAABQQ0ODAoPeJxtUNFOgzAUfe9XNPEZXDc3xpImTkCdgWFkm76ZC7tzTaCQtizOr7cDfDFLmjQ9p
/fcc87NpkWaYUOZR0f+YjJfTDz6lGzoeMR8EigEI2oZgkF+QZyR7zCPRCco247hByg1khB1oUTTI
VtZikoY3NNSFCg10vxMj8Y0enF7+3MUJbqiJqn6Ail0L3JhLbkH0ypwTd2QopYHFwojTsiNapEEt
TT2HSUgSg4A93meu0Vd9T8zA8qgGtx0UNwv35wbXEOFPEiTJHoLVsuYWA1pUIIsMPpuhDoP+eY2n
2cPGWZXIY9XYRatnZhNZ3OPTRjzpnczkqE6obL0w/vOc7wPljqP/vLFmYXPyd/wdeXXVhVH0Pi/0
aGqHSp9KWTcZ1i3VY4qPWy1xbnDiPXCr/gZyulyVudPae9f57CZMDAsAhRpli3TWhHdAL0fOkAeh
38ZCSuf+QIUfBreVMIH4k/6isdn0HptYXvU608=X02g0
```

选择My own database
>Database type: MySQL
>Setup type: Simple
>Hostname: mysql-confluence
>PortThis: 3306
>Database name: confluencedb
>Username: confluence
>Password: confluence

点击Test connection 后可能会出现
`Collation error
The database collation 'utf8_general_ci' is not supported by Confluence. You need to use 'utf8_bin'.`
错误
进入mysql容器
>docker exec -it mysql bash
>mysql -uroot -proot
>`alter database confluencedb character set utf8 COLLATE utf8_bin;
>SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;`

然后在点击Test connection，提示Success! Database connected successfully.
然后点击Next 等待数据库初始化
然后选择Restore From Backup
将备份的包拷贝到本机的 `/data/confluence/data/restore/` 目录下
此时页面上就会有相应的包，选择之后点击import后，等待导入完数据，此时Confluence迁移完成