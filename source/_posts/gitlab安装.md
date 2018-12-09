---
title: centos 7 部署 gitlab
date: 2017-09-08
tags: git
categories: git
---
## gitlab的安装搭建
<!--more-->

### 安装基础环境包

    yum -y install curl policycoreutils openssh-server openssh-clients

### 启动sshd

    systemctl enable sshd
    systemctl start sshd

### 安装postfix

    yum -y install postfix
    systemctl enable postfix
    systemctl start postfix

### 添加防火墙规则

    firewall-cmd --permanent --add-service=http
    systemctl reload firewalld
or
    yum install firewalld
    systemctl unmadk firewalld

### 下载并安装软件包

    curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
    yum install gitlab-ce

如遇到time out，请更换成国内源
    vim /etc/yum.repos.d/gitlab-ce.repo
> [gitlab-ce]
> name=gitlab-ce
> baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
> repo_gpgcheck=0
> gp0gcheck=0
> enabled=1
> gpgkey=https://packages.gitlab.com/gpg.key

    yum makecache
    yum install gitlab-ce -y


or 下载相应版本gitlab的rpm包

    https://packages.gitlab.com/gitlab/gitlab-ce/    

安装完毕后

    gitlab-ctl reconfigure 
> gitlab: GitLab should be reachable at http://iZ2851te7e5Z  
gitlab: Otherwise configure GitLab for your system by editing /etc/gitlab/gitlab.rb file  
gitlab: And running reconfigure again.  
gitlab:   
gitlab: For a comprehensive list of configuration options please see the Omnibus GitLab readme  
gitlab: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md  
gitlab:   
It looks like GitLab has not been configured yet; skipping the upgrade script.  
  验证中      : gitlab-ce-8.7.6-ce.0.el7.x86_64                                                                                             1/1     
已安装:  
  gitlab-ce.x86_64 0:8.7.6-ce.0.el7                                                                                                         
完毕！

### 配置并启动gitlab

如果遇到
>安装GitLab出现ruby_block[supervise_redis_sleep] action run

那么需要运行

	systemctl restart gitlab-runsvdir
    gitlab-ctl reconfigure

### 默认账号密码是

    Username: root
    Password: 5iveL!fe
测试地址（默认80端口）
    http://127.0.0.1/

## gitlab的备份

### 备份命令
    gitlab-rake gitlab:backup:create
默认的备份目录为： /var/opt/gitlab/backups  备份文件名类似： 时间戳_gitlab_backup.tar

如要修改备份目录：
    vim /etc/gitlab/gitlab.rd
    gitlab_rails['backup_path'] = '/mnt/gitlab_backups'

### gitlab数据的恢复或还原
提示：gitlab数据的恢复或者迁移成功的前提--两台服务器的gitlab的版本必须相同，若不同侧可能迁移或者恢复失败  

>将备份文件放在gitlab的默认备份目录  
比如/var/opt/gitlab/backups下的1504693308_gitlab_backup.tar     
  
设置自动备份

    0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create

恢复或者还原  

停服务

    gitlab-ctl stop unicorn
	gitlab-ctl stop sidekiq

恢复数据

    gitlab-rake gitlab:backup:restore BACKUP=1504693308

BACKUP后面跟的是备份文件的时间戳，比如恢复备份文件1504693308_gitlab_backup.tar

恢复完启动服务

    gitlab-ctl start

## gitlab nginx的修改

配置文件 /var/opt/gitlab/nginx/conf/gitlab-http.conf。这个文件是gitlab内置的nginx的配置文件，里面可以影响到nginx真实监听端口。

    server {
        listen *:80;
		server_name ip
		server_tokens off; ##Don't show the nginx version number, a security best practice 		
    }

修改完成后，重启下就可以了
    gitlab-ctl restart

## 汉化

### 查看自己gitlab的版本号
    cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
    9.3.5

当前版本为v9.3.5,并确认汉化版本库是否包含该版本的汉化标签(-zh结尾),也就是是否包含 v9.3.5-zh

### 下载汉化包并汉化

克隆汉化版本库，此处用了好久的时间，拉取这个分支，没有更好的办法，可以自行百度一下Git慢的解决方式

    git clone https://gitlab.com/xhang/gitlab.git

如果已经克隆过，则进行更新

    git fetch

比较汉化标签和原标签，导出 patch 用的 diff 文件.进入刚才的目录git clone 的目录

    cd gitlab
	git diff v9.3.5 v9.3.5-zh > ../9.3.5-zh.diff

上传9.3.5-zh.diff文件到服务器停止 gitlab

    gitlab-ctl stop
	patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < ../9.3.6-zh.diff
>这里path 如果也出现 command not found 说明path安装包没有安装，然后在运行前边的代码就可以了  
yum -y install patch 


重启gitlab即可.

    gitlab-ctl start

执行重新配置命令

    gitlab-ctl reconfigure



## 升级

### 关闭gitlab服务

	gitlab-ctl stop unicorn
	gitlab-ctl stop sidekiq
	gitlab-ctl stop nginx


### 备份gitlab

	gitlab-rake gitlab:backup:create

### 下载gitlab的RPM包并进行升级

	curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
	yum update gitlab-ce
或者直接上官网下载相应gitlab的版本所对应的软件包（[https://packages.gitlab.com/gitlab/gitlab-ce/](https://packages.gitlab.com/gitlab/gitlab-ce/)）

	yum install gitlab-ce-8.8.3-ce.0.el7.x86_64

### 启动并查看gitlab的版本信息

	gitlab-ctl reconfigure
	gitlab-ctl restart
	head -1 /opt/gitlab/version-manifest.txt
	cat /opt/gitlab/embedded/service/gitlab-rails/VERSION

### 可以更换gitlab自带的nginx，使用自行编译好的nginx来管理gitlab服务

	vim /etc/gitlab/gitlab.rd

>设置nginx为dalse，关闭自带nginx  
>nginx['enable'] = false  
>检查默认nginx配置文件，并迁移至新nginx服务。  
>/var/opt/gitlab/nginx/conf/nginx.conf  #nginx配置文件，包含gitlab-http.conf文件  
>/var/ope/gitlab/nginx/conf/gitlab-http.conf #gitlab核心nginx配置文件
 
重启nginx、gitlab服务

	gitlab-ctl reconfigure
	systemctl restart nginx

访问报502。原因是nginx用户无法访问gitlab用户的socket文件。重启gitlab需要重新授权

	chmod -R o+x /var/opt/gitlab/gitlab-rails
	gitlab-ctl restart


##  卸载

前提：必须在Gitlab运行状态下才能卸载

    gitlab-ctl uninstall
    rpm -e gitlab-ce

在卸载gitlab然后再次安装执行gitlab-ctl reconfigure的时候往往会出现:ruby_block[supervise_redis_sleep] action run,会一直卡无法往下进行！   
解决方案：  

按住CTRL+C强制结束  

运行:  

    systemctl restart gitlab-runsvdir
    gitlab-ctl reconfigure


## 报错解决
### 迁移后页面500错误
如果遇到迁移项目后web页面点击项目报500错误，查看相关日志如下

	==> /var/log/gitlab/gitlab-rails/production.log <==  
	Started GET "/ops/install_php" for 127.0.0.1 at   2017-09-13 10:32:45 +0800  
	Processing by ProjectsController#show as HTML    
	 Parameters: {"namespace_id"=>"ops",  "id"=>"install_php"}  
	Completed 500 Internal Server Error in 36ms (ActiveRecord: 2.2ms)  

	OpenSSL::Cipher::CipherError (bad decrypt):  
	  app/models/project.rb:383:in `import_url'
	  app/models/project.rb:413:in `external_import?'
	  app/models/project.rb:405:in `import?'
	  app/models/project.rb:421:in `import_in_progress?'
	  app/controllers/projects_controller.rb:93:in `show'
	  lib/gitlab/middleware/go.rb:16:in `call'

可得知是OpenSSL解密出现了问题，经调查后发现
这个是gitlab数据迁移的时候一个缺陷


### 解决方案
1.覆盖原来gitlab的db_key_base到新的gitlab

db_key_base位置在/etc/gitlab/gitlab-secrets.json

2.EE版本执行

	sudo gitlab-rails runner "Project.where(mirror: false).where.not(import_url: nil).each { |p| p.import_data.destroy if p.import_data }"

3.CE版本执行

	sudo gitlab-rails runner "Project.where.not(import_url: nil).each { |p| p.import_data.destroy if p.import_data }"



### 备机web界面项目地址显示主机名问题

	vim /var/opt/gitlab/gitlab-rails/etc/gitlab.yml

修改host项

>由host: localhost改成  
>host: ******

gitlab重启即可
	
	gitlab-ctl restart


