---
title: Kubernetes如何使用kube-dns实现服务发现
date: 2018-04-10
tags:
---
# Kubernetes中如何发现服务
<!--more-->
##  发现Pod提供的服务
首先使用nginx-deployment.yaml文件创建一个Nginx Deployment，文件内容如图所示：
首先创建两个运行Nginx服务的Pod：
![http://blog.tenxcloud.com/wp-content/uploads/2016/10/1.png](http://blog.tenxcloud.com/wp-content/uploads/2016/10/1.png)

使用kubectl create -f nginx-deployment.yaml指令创建，这样便可以得到两个运行nginx服务的Pod。待Pod运行之后查看一下它们的IP，并在k8s集群内通过podIP和containerPort来访问Nginx服务：

### 获取Pod IP：
```
kubectl get pod -o yaml -l run=myy-nginx|grep podIP
```
![http://blog.tenxcloud.com/wp-content/uploads/2016/10/2.png](http://blog.tenxcloud.com/wp-content/uploads/2016/10/2.png)

### 在集群内访问Nginx服务：
![http://blog.tenxcloud.com/wp-content/uploads/2016/10/3.png](http://blog.tenxcloud.com/wp-content/uploads/2016/10/3.png)

看到这里相信很多人会有以下疑问：
>1.  每次收到获取podIP太扯了，总不能每次都要手动改程序或者配置才能访问服务吧，要怎么提前知道podIP呢？
>2.  Pod在运行中可能会重建，IP变了怎么解？
>3.  如何在多个Pod中实现负载均衡嘞？

这些问题使用k8s Service就可以解决。

## 使用Service发现服务
下面为两个Nginx Pod创建一个Service。使用nginx-service.yaml文件进行创建，文件内容如下：
![http://blog.tenxcloud.com/wp-content/uploads/2016/10/4.png](http://blog.tenxcloud.com/wp-content/uploads/2016/10/4.png)
创建之后，仍需要获取Service的Cluster-IP，再结合Port访问Nginx服务。

Service可以将pod  IP封装起来，即使Pod发生重建，依然可以通过Service来访问Pod提供的服务。此外，Service还解决了负载均衡的问题，大家可以多访问几次Service，然后通过kubectl logs 来查看两个Nginx Pod的访问日志来确认。

### 获取IP：
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/5.png)

### 在集群内访问Service：
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/6.png)

虽然Service解决了Pod的服务发现和负载均衡问题，但存在着类似的问题：不提前知道Service的IP，还是需要改程序或配置啊。看到这里有没有感觉身体被掏空？

接下来聊聊kube-dns是如何解决上面这个问题的。

## 使用kube-dns发现服务

kube-dns可以解决Service的发现问题，k8s将Service的名称当做域名注册到kube-dns中，通过Service的名称就可以访问其提供的服务。

可能有人会问如果集群中没有部署kube-dns怎么办？没关系，实际上kube-dns插件只是运行在kube-system命名空间下的Pod，完全可以手动创建它。可以在k8s源码（v1.2）的cluster/addons/dns目录下找到两个模板（skydns-rc.yaml.in和skydns-svc.yaml.in）来创建，为大家准备的完整示例文件会在分享结束后提供获取方式，PPT中只截取了部分内容。

通过skydns-rc.yaml文件创建kube-dns Pod，其中包含了四个containers，这里开始简单过一下文件的主要部分，稍后做详细介绍。

第一部分可以看到kube-dns使用了RC来管理Pod，可以提供最基本的故障重启功能。

创建kube-dns Pod，其中包含了4个containers
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/7.png)

接下来是第一个容器  etcd  ，它的用途是保存DNS规则。
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/8.png)

第二个容器  kube2sky ，作用是写入DNS规则。
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/9.png)

第三个容器是  skydns ，提供DNS解析服务。
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/10.png)

最后一个容器是  healthz ，提供健康检查功能。
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/11.png)

有了Pod之后，还需要创建一个Service以便集群中的其他Pod访问DNS查询服务。通过skydns-svc.yaml创建Service，内容如下：
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/12.png)

创建完kube-dns Pod和Service，并且Pod运行后，便可以访问kube-dns服务。

下面创建一个Pod，并在该Pod中访问Nginx服务：

创建之后等待kube-dns处于运行状态
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/13.png)

再新建一个Pod，通过其访问Nginx服务
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/14.png)

在curl-util Pod中通过Service名称访问my-nginx Service：
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/15.png)

只要知道需要的服务名称就可以访问，使用kube-dns发现服务就是那么简单。

虽然领略了使用kube-dns发现服务的便利性，但相信有很多人也是一头雾水：kube-dns到底怎么工作的？在集群中启用了kube-dns插件，怎么就能通过名称访问Service了呢？

# kube-dns原理

## Kube-dns组成

之前已经了解到kube-dns是由四个容器组成的，它们扮演的角色可以通过下面这张图来理解。
![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/kube-dns%E6%9E%B6%E6%9E%84.png)

其中：

●  SkyDNS是用于服务发现的开源框架，构建于etcd之上。作用是为k8s集群中的Pod提供DNS查询接口。项目托管于https://github.com/skynetservices/skydns

●  etcd是一种开源的分布式key-value存储，其功能与ZooKeeper类似。在kube-dns中的作用为存储SkyDNS需要的各种数据，写入方为kube2sky，读取方为SkyDNS。项目托管于https://github.com/coreos/etcd。

●   kube2sky是k8s实现的一个适配程序，它通过名为kubernetes的Service（通过kubectl get svc可以查看到该Service，由集群自动创建）调用k8s的list和watch API来监听k8s Service资源的变更，从而修改etcd中的SkyDNS记录。代码可以在k8s源码（v1.2）的cluster/addons/dns/kube2sky/目录中找到。

●   exec-healthz是k8s提供的一种辅助容器，多用于side car模式中。它的原理是定期执行指定的Linux指令，从而判断当前Pod中关键容器的健康状态。在kube-dns中的作用就是通过nslookup指令检查DNS查询服务的健康状态，k8s livenessProbe通过访问exec-healthz提供的Http API了解健康状态，并在出现故障时重启容器。其源码位于https://github.com/kubernetes/contrib/tree/master/exec-healthz。

●  从图中可以发现，Pod查询DNS是通过ServiceName.Namespace子域名来查询的，但在之前的示例中只用了Service名称，什么原理呢？其实当我们只使用Service名称时会默认Namespace为default，而上面示例中的my-nginx Service就是在default Namespace中，因此是可以正常运行的。关于这一点，后续再深入介绍。

●  skydns-rc.yaml中可以发现livenessProbe是设置在kube2sky容器中的，其意图应该是希望通过重启kube2sky来重新写入DNS规则

## 域名格式

接下来了解一下kube-dns支持的域名格式，具体为：..svc.。

其中cluster_domain可以使用kubelet的--cluster-domain=SomeDomain参数进行设置，同时也要保证kube2sky容器的启动参数中--domain参数设置了相同的值。通常设置为cluster.local。那么之前示例中的my-nginx Service对应的完整域名就是my-nginx.default.svc.cluster.local。看到这里，相信很多人会有疑问，既然完整域名是这样的，那为什么在Pod中只通过Service名称和Namespace就能访问Service呢？下面来解释其中原因。

# 配置

## 域名解析配置

为了在Pod中调用其他Service，kubelet会自动在容器中创建域名解析配置（/etc/resolv.conf），内容为：

![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/16.png)

感兴趣的可以在网上查找一些resolv.conf的资料来了解具体的含义。之所以能够通过Service名称和Namespace就能访问Service，就是因为search配置的规则。在解析域名时会自动拼接成完整域名去查询DNS。

刚才提到的kubelet --cluster-domain参数与search的具体配置是相对应的。而kube2sky容器的--domain参数影响的是写入到etcd中的域名，kube2sky会获取Service的名称和Namespace，并使用--domain参数拼接完整域名。这也就是让两个参数保持一致的原因。

## NS 相关配置

kube-dns可以让Pod发现其他Service，那Pod又是如何自动发现kube-dns的呢？在上一节中的/etc/resolv.conf中可以看到nameserver，这个配置就会告诉Pod去哪访问域名解析服务器。

![](http://blog.tenxcloud.com/wp-content/uploads/2016/10/17.png)

相应的，可以在之前提到的skydns-svc.yaml中看到spec.clusterIP配置了相同的值。通常来说创建一个Service并不需要指定clusterIP，k8s会自动为其分配，但kube-dns比较特殊，需要指定clusterIP使其与/etc/resolv.conf中的nameserver保持一致。

修改nameserver配置同样需要修改两个地方，一个是kubelet的--cluster-dns参数，另一个就是kube-dns Service的clusterIP。


# 总结
接下来重新梳理一下本文的主要内容：
>●    在k8s集群中，服务是运行在Pod中的，Pod的发现和副本间负载均衡是我们面临的问题。
>●    通过Service可以解决这两个问题，但访问Service也需要对应的IP，因此又引入了Service发现的问题。
>●    得益于kube-dns插件，我们可以通过域名来访问集群内的Service，解决了Service发现的问题。
>●    为了让Pod中的容器可以使用kube-dns来解析域名，k8s会修改容器的/etc/resolv.conf配置。

有了以上机制的保证，就可以在Pod中通过Service名称和namespace非常方便地访问对应的服务了。

