---
title: tcpdump：理论、自动抓包及业务架构树的生成
date: 2017-03-02
tags:
---
###一、tcpdump基础

tcpdump是一个对网络数据包进行截获的包分析工具。

tcpdump可以将网络中传送的数据包的“头”完全截获下来提供分析。它支持针对网络层、协议、主机、端口等的过滤，并支持与、或、非逻辑语句协助过滤有效信息。
<!--more-->
命令使用规则如下：

    tcpdump version 4.1-PRE-CVS_2016_05_10
    libpcap version 1.4.0
    Usage: tcpdump [-aAdDefhIJKlLnNOpqRStuUvxX] [ -B size ] [ -c count ]
    		[ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
    		[ -i interface ] [ -j tstamptype ] [ -M secret ]
    		[ -Q|-P in|out|inout ]
    		[ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
    		[ -W filecount ] [ -y datalinktype ] [ -z command ]
    		[ -Z user ] [ expression ]
    
过滤方式有很多，可以依据所需设置过滤条件，较常用的三种：
####1、可以按host过滤，例如：

    tcpdump -i eth0 -n -X src host 172.17.198.10
####2、可以按port过滤，例如：
    cpdump -i eth0 -n -X src host 172.17.198.10 and dst port 80

####3、可以按protocol过滤，例如：

    tcpdump -i eth0 -n -X src host 172.17.198.10 and dst port 80 and tcp
下面来看一下tcpdump过滤规则的具体使用：

我们在服务器10.219.153.215上搭建了一个http服务用来作为服务端，10.19.66.62作为客户端客户端对其发起访问。我们使用前面提到的按host 10.19.66.62、port 80以及protocol tcp的组合条件来执行tcpdump。

    tcpdump -i eth0 -n  tcp port 80 and host 172.17.198.10
    [root@ssy-turn1 ~]# tcpdump -i eth0 -n  tcp port 80 and host 172.17.198.10
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
    11:39:05.355344 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [S], seq 636850073, win 65160, options [mss 1460,sackOK,TS val 2335946898 ecr 3699469428,nop,wscale 14], length 0
    11:39:05.355397 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [S.], seq 4174661387, ack 636850074, win 65160, options [mss 1460,sackOK,TS val 3699486375 ecr 2335946898,nop,wscale 14], length 0
    11:39:05.357024 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [.], ack 1, win 4, options [nop,nop,TS val 2335946899 ecr 3699486375], length 0
    11:39:05.357062 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [P.], seq 1:193, ack 1, win 4, options [nop,nop,TS val 2335946899 ecr 3699486375], length 192
    11:39:05.357071 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [.], ack 193, win 4, options [nop,nop,TS val 3699486376 ecr 2335946899], length 0
    11:39:05.357403 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [P.], seq 1:765, ack 193, win 4, options [nop,nop,TS val 3699486377 ecr 2335946899], length 764
    11:39:05.357766 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [.], ack 765, win 4, options [nop,nop,TS val 2335946901 ecr 3699486377], length 0
    11:39:05.359151 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [F.], seq 193, ack 765, win 4, options [nop,nop,TS val 2335946901 ecr 3699486377], length 0
    11:39:05.359329 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [F.], seq 765, ack 194, win 4, options [nop,nop,TS val 3699486379 ecr 2335946901], length 0
    11:39:05.359613 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [.], ack 766, win 4, options [nop,nop,TS val 2335946903 ecr 3699486379], length 0
    
不同的协议类型有不同的数据包格式显示，以tcp包为例，通常tcpdump对tcp数据包的显示格式如下:

    src > dst: flags data-seqno ack window urgent options
    src ＞ dst：表明从源地址到目的地址
    flags：TCP包中的标志信息，S 是SYN标志,，F (FIN)，P (PUSH)，R (RST)，”.” (没有标记）
    data-seqno：是数据包中的数据的顺序号
    ack：是下次期望的顺序号
    window：是接收缓存的窗口大小
    urgent：表明数据包中是否有紧急指针
    options：选项
执行抓包过程中输出的这八行数据其实包含了tcp三次握手和四次挥手的交互过程，详细分析下看看：

    11:39:05.355344 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [S], seq 636850073, win 65160, options [mss 1460,sackOK,TS val 2335946898 ecr 3699469428,nop,wscale 14], length 0
    11:39:05.355397 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [S.], seq 4174661387, ack 636850074, win 65160, options [mss 1460,sackOK,TS val 3699486375 ecr 2335946898,nop,wscale 14], length 0
    11:39:05.357024 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [.], ack 1, win 4, options [nop,nop,TS val 2335946899 ecr 3699486375], length 0
    11:39:05.357062 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [P.], seq 1:193, ack 1, win 4, options [nop,nop,TS val 2335946899 ecr 3699486375], length 192
    11:39:05.357071 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [.], ack 193, win 4, options [nop,nop,TS val 3699486376 ecr 2335946899], length 0
    11:39:05.357403 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [P.], seq 1:765, ack 193, win 4, options [nop,nop,TS val 3699486377 ecr 2335946899], length 764
    11:39:05.357766 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [.], ack 765, win 4, options [nop,nop,TS val 2335946901 ecr 3699486377], length 0
    11:39:05.359151 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [F.], seq 193, ack 765, win 4, options [nop,nop,TS val 2335946901 ecr 3699486377], length 0
    11:39:05.359329 IP 172.17.198.11.http > 172.17.198.10.solaris-audit: Flags [F.], seq 765, ack 194, win 4, options [nop,nop,TS val 3699486379 ecr 2335946901], length 0
    11:39:05.359613 IP 172.17.198.10.solaris-audit > 172.17.198.11.http: Flags [.], ack 766, win 4, options [nop,nop,TS val 2335946903 ecr 3699486379], length 0
第一至三行为建立链接的三次握手过程，包状态为：[S]、[S.]、[.]，第四至七行为传输数据的过程，包状态为[P.]、[.]；第八至十行为关闭链接的四次挥手过程（ack延迟发送未禁用，所以这里只看到三个包），包状态为[F.]、[F.]、[.]。  

####第一行：客户端10向服务器11发送了一个序号seq 636850073给服务端；

####第二行：服务端收到后将序号加一返回ack 636850074；

####第三行：客户端检查返回值正确，向服务端发ack 1，建立了链接；

####第四行和第七行：具体的数据交互，tcpdump命令-x可以显示出具体内容；

####第八行：客户端发一个序号seq 193，说明要断开链接；

####第九行：服务端在收到后序号加一返回ack 194，同意断开链接；

####第十行：客户端检查返回值正确，向服务端发ack，链接断开。



