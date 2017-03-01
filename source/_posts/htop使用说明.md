---
title: htop使用说明
date: 2016-09-06
tags:
---
一、htop 简介

This is htop, an interactive process viewer for Linux. It is a text-mode application (for console or X terminals) and requires ncurses.

Comparison between htop and top
<!--more-->
In 'htop' you can scroll the list vertically and horizontally to see all processes and complete command lines.
In 'top' you are subject to a delay for each unassigned key you press (especially annoying when multi-key escape sequences are triggered by accident).
'htop' starts faster ('top' seems to collect data for a while before displaying anything).
In 'htop' you don't need to type the process number to kill a process, in 'top' you do.
In 'htop' you don't need to type the process number or the priority value to renice a process, in 'top' you do.
'htop' supports mouse operation, 'top' doesn't
'top' is older, hence, more used and tested.
htop 是Linux系统中的一个互动的进程查看器，一个文本模式的应用程序(在控制台或者X终端中)，需要ncurses。

与Linux传统的top相比，htop更加人性化。它可让用户交互式操作，支持颜色主题，可横向或纵向滚动浏览进程列表，并支持鼠标操作。

与top相比，htop有以下优点：

可以横向或纵向滚动浏览进程列表，以便看到所有的进程和完整的命令行。
在启动上，比top 更快。
杀进程时不需要输入进程号。
htop 支持鼠标操作。
top 已经很老了。
htop 官网：http://htop.sourceforge.net/

二、htop 安装

a. 源码包安装

    # tar zxvf htop-1.0.2.tar.gz
    
    # cd htop-1.0.2
    
    # ./configure
    
![](http://images.cnitblog.com/blog/370046/201301/12224043-10fbaaa4ef2d49ba844de4ff1df2cea2.jpg)

    # make && make install

![](http://images.cnitblog.com/blog/370046/201301/12224044-8e70dd81c3594f14bf334dbf7ae12cda.jpg)

若出现错误：


    configure: error: You may want to use --disable-unicode or install libncursesw.

则需安装 ncurses-devel


    # yum install ncurses-devel

b. RHEL/CentOS 安装

可以通过 yum install htop 来安装它，但前提是要添加epel 的yum源，具体请参考 CentOS yum 源的配置与使用。

    # rpm -ivh http://download.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm 
    # rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL//导入key 
    # yum install htop
    
三、htop 参数

键入htop 命令，打开htop。


    # htop


 ![](http://images.cnitblog.com/blog/370046/201301/12224047-b6fd5270cea14cee87544ff3ca193b34.jpg)

上面左上角显示CPU、内存、交换区的使用情况，右边显示任务、负载、开机时间，下面就是进程实时状况。

下面是 F1~F10 的功能和对应的字母快捷键。

----------

Shortcut Key　　　　　　  　　 Function Ke　　　　　　  　　 Description　　　　　　 　　　　　　  　　  中文说明  

h,?　　　　　　  　　　　　　　  　　   F1	　　　　　　  　　 Invoke htop Help　　　　　　  　　 	　　　　　　查看htop使用说明  
S	　　　　　　  　　 　　　　　　  　　F2　　　　　　  　　 	Htop Setup Menu	htop　　　　　　  　　 　　　 设定  
/	　　　　　　  　　 　　　　　　   　　  F3	　　　　　　  　　 Search for a Process	　　　　　　  　　　　　　 搜索进程  
\	　　　　　　  　　 　　　　　　  　　 F4	　　　　　　  　　 Incremental process filtering	　　　　　　  　　 增量进程过滤器  
t	　　　　　　  　　 　　　　　　  　　 F5　　　　　　  　　 	Tree View	　　　　　　  　　 　　　　　　  　　　 显示树形结构  
<, >	　　　　　　  　　 　　　　　　  　F6	　　　　　　  　　 Sort by a column	　　　　　　  　　　　　　　　   选择排序方式  
[	　　　　　　  　　 　　　　　　  　　 F7	　　　　　　  　　 Nice - (change priority)	　　　可减少nice值，这样就可以提高对应进程的优先级  
]	　　　　　　  　　 　　　　　　  　　 F8	　　　　　　  　　 Nice + (change priority)	　　　可增加nice值，这样就可以降低对应进程的优先级  
k	　　　　　　  　　 　　　　　　  　　 F9	　　　　　　  　　 Kill a Process	　　　　　　  　　 　　　　　　  　可对进程传递信号  
q	　　　　　　  　　 　　　　　　  　　 F10	　　　　　　  　　 Quit htop	　　　　　　  　　 　　　　　　  　　 结束htop  

----------

命令行选项（COMMAND-LINE OPTIONS）

-C --no-color　　　　 　　 使用一个单色的配色方案

-d --delay=DELAY　　　　 设置延迟更新时间，单位秒

-h --help　　　　　　  　　 显示htop 命令帮助信息

-u --user=USERNAME　　  只显示一个给定的用户的过程

-p --pid=PID,PID…　　　    只显示给定的PIDs

-s --sort-key COLUMN　    依此列来排序

-v –version　　　　　　　   显示版本信息

交互式命令（INTERACTIVE COMMANDS）

----------


上下键或PgUP, PgDn 选定想要的进程，左右键或Home, End 移动字段，当然也可以直接用鼠标选定进程；

Space    标记/取消标记一个进程。命令可以作用于多个进程，例如 "kill"，将应用于所有已标记的进程

U    取消标记所有进程

s    选择某一进程，按s:用strace追踪进程的系统调用

l    显示进程打开的文件: 如果安装了lsof，按此键可以显示进程所打开的文件

I    倒转排序顺序，如果排序是正序的，则反转成倒序的，反之亦然

+, -    When in tree view mode, expand or collapse subtree. When a subtree is collapsed a "+" sign shows to the left of the process name.

a (在有多处理器的机器上)    设置 CPU affinity: 标记一个进程允许使用哪些CPU

u    显示特定用户进程

M    按Memory 使用排序

P    按CPU 使用排序

T    按Time+ 使用排序

F    跟踪进程: 如果排序顺序引起选定的进程在列表上到处移动，让选定条跟随该进程。这对监视一个进程非常有用：通过这种方式，你可以让一个进程在屏幕上一直可见。使用方向键会停止该功能。

K    显示/隐藏内核线程

H    显示/隐藏用户线程

Ctrl-L    刷新

Numbers    PID 查找: 输入PID，光标将移动到相应的进程上

----------


四、htop 使用

4.1. 显示自带帮助

鼠标点击Help或者按F1 显示自带帮助
![](http://images.cnitblog.com/blog/370046/201301/12224049-575f66ab3d964c5cb419959aee9032b1.jpg)


4.2. htop 设定

鼠标点击Setup或者按下F2 之后进入htop 设定的页面，Meters 页面设定了顶端的一些信息显示，顶端的显示又分为左右两侧，到底能显示些什么可以在最右侧那栏新增，要新增到上方左侧（F5）或是右侧（F6）都可以，这就是个人设定的范围了。这里多加了一个时钟。

![](http://images.cnitblog.com/blog/370046/201301/12224051-9b4d728776d1499190c9d21ab3615f5a.jpg)

上方左右两栏的显示方式分为Text Bar Graph Led 四种，下图我就把 cpu memory swap 改成文本模式显示，然后右栏的改成Bar 显示，clock 用LED方式显示。数据显示都差不多，只是这样看有点不习惯了。

![](http://images.cnitblog.com/blog/370046/201301/12224052-befc1e283e4c4576a59f89e6f72a4337.jpg)

关于Display options 的设定，可要根据管理者自己的需要来设定。

![](http://images.cnitblog.com/blog/370046/201301/12224054-b8d7722ac20b421299bd0c72b3394198.jpg)

颜色选择，除了基本的颜色显示之外，htop 还提供了换面板的功能，其实也只是改变一些色彩显示的设定，虽然说不能自定义到细部的颜色显示，但是至少提供了几种风格可以选择。

![](http://images.cnitblog.com/blog/370046/201301/12224055-b6a13eb2abaa45fab167e61a8db12356.jpg)

最后一项的设定是调整 Columns 的显示，就是在一般htop 指令进来希望可以看到的什么样的数据及信息，字段的调整可以在这边做个人化的设定，一般使用系统默认值就好了。

![](http://images.cnitblog.com/blog/370046/201301/12224058-1ef4f726ccd64dfbaab68142dc9a95be.jpg)

4.3. 搜索进程

鼠标点击Search 或者按下F3 或者输入"/"， 输入进程名进行搜索，例如搜索ssh

![](http://images.cnitblog.com/blog/370046/201301/12224100-dab0040192b24614b44d1e28d1b4badb.jpg)

4.4. 过滤器

按下F4，进入过滤器，相当于关键字搜索，不区分大小写，例如过滤dev

![](http://images.cnitblog.com/blog/370046/201301/12224102-6339cac4da0b482097e1a5b218b523c4.jpg)

4.5. 显示树形结构

输入"t"或按下F5，显示树形结构，意思跟pstree 差不多，能看到所有程序树状执行的结构，这对于系统管理来说相当方便，理清程序是如何产生的，当然树状结构的浏览也可以依照其他数据来排序。

![](http://images.cnitblog.com/blog/370046/201301/12224104-f5d4a8c905cd4a83b262ad2e5944f3fe.jpg)

4.6. 选择排序方式

按下F6 就可以选择依照什么来排序，最常排序的内容就是cpu 和memory 吧！

![](http://images.cnitblog.com/blog/370046/201301/12224107-6a091d2523cd4829b243f635b1c32a0a.jpg)

4.7 操作进程

F7、F8分别对应nice-和nice+，F9对应kill给进程发信号，选好信号回车就OK了

![](http://images.cnitblog.com/blog/370046/201301/12224112-9dc1706a8dbd4972b78a78af34f75ec7.jpg)

4.8. 显示某个用户的进程，在左侧选择用户

输入"u"，在左侧选择用户

![](http://images.cnitblog.com/blog/370046/201301/12224115-289b4cf95c344dada4aa048c25f821ce.jpg)

五、Alias top

也许你用惯了top，我们也可以用top来打开htop。

编辑/root/.bashrc文件，添加如下代码

    if [ -f /usr/local/bin/htop ]; then
    alias top=’/usr/local/bin/htop’
    fi
    # source /root/.bashrc