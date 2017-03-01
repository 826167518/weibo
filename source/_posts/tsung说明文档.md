---
title: tsung使用说明
date: 2017-01-08
tags:
---

1、tsung安装

tsung 一个非常优秀的压力测试工具，在8核32G机器上可以轻易的产生每秒10000个并发请求，且占用的资源很少，当前版本1.5.0
<!--more-->
使用erlang开发，需要先安装erlang虚拟机。安装过程略

2、tsung使用  
$ tsung -f ./tsung/tsung.xml sta

3、tusng.xml

下面是我自己的tsung.xml

        <?xml version="1.0"?>
    <!DOCTYPE tsung SYSTEM "/usr/local/share/tsung/tsung-1.0.dtd">
    <!-- dumptraffic是调试模式，如果为true，就会打印详细的请求返回信息，一般设置为false， -->
    <tsung loglevel="notice" dumptraffic="false" version="1.0">
    <!-- tsung所在的服务器，maxusers就是tsung产生的最大用户数 -->
      <clients>
    <client host="localhost" use_controller_vm="true"  maxusers="100000" />
      </clients>
    <!-- 被测服务器的ip和端口号，type一般设为tcp ->
      <servers>
    <server host="@server" port="@port" type="tcp"/>
      </servers>
    <!-- tsung产生的压力 -->
    <load>
    <!-- phase="1" 第一阶段；duration：测试持续时间；unit：单位秒 -->
    <arrivalphase phase="1" duration="@duration" unit="second">
    <!-- maxnumber：最大用户数；arrivalrate：每秒新增用户数；unit：单位秒-->
      <users maxnumber="@maxuser" arrivalrate="@user" unit="second"/>
    </arrivalphase>
      </load>
    <!-- 外部变量 -->
      <options>
    <!-- 引入一个外部文件，类型为file_server，变量名为userfile，文件路径：/tmp/users-->
    <!-- 文件是以逗号分隔的csv文件 -->
    <option name="file_server" id="userfile" value="/tmp/users"/>
      </options>
    <!-- 会话，每个用户都按照sessions中的配置发送请求 -->
      <sessions>
    <!--probability=“100”:这个session的请求概率是100%，如果要同时测多个api，可以设置请求概率；请求类型为http -->
    <session name="test" probability="100" type="ts_http">
    <!-- 请求次数，to是最大请求数，如果设为100，就是每个用户请求100次 -->
      <for from="1" to="@loop" incr="1" var="counter">
    <!-- 解析前面引入的外部文件，以逗号做为分隔符，随机读取 -->
    <setdynvars sourcetype="file" fileid="userfile" delimiter="," order="random">
      <var name="user_id" />
      <var name="passwd" />
      <var name="auth_token" />
    </setdynvars>
    <!-- 返回随机小数，变量名为decimal，code中可以写erlang函数 -->
    <setdynvars sourcetype="eval" code="fun({Pid,DynVars}) -> random:uniform() end.">
      <var name="decimal" />
    </setdynvars>
    <!-- 返回随机数，从1到10000 -->
    <setdynvars sourcetype="random_number" start="1" end="10000">
       <var name="int_1_10000" />
    </setdynvars>
    <!-- 返回随机字符串，长度为10 -->
    <setdynvars sourcetype="random_string" length="10">
       <var name="string_10" />
    </setdynvars>
    <!-- subst="true"：如果在request中使用变量，需要设置subst -->
    <request subst="true">
    <!-- url：被测试的url；method：GET、POST等；contents：POST请求的参数 -->
      <http url="@api" method="@method" contents="@contents" version="1.1">
    <!-- http header，可以添加Authorization、Cookie等，注意变量的使用格式：%%_xxx%% -->
    <http_header name="Authorization" value="111"/>
    <http_header name="Cookie" value="authToken=%%_auth_token%%; Path=/"/>
    <!-- content-Type：POST请求参数的格式，如果是json格式可以这样写 -->
    <http_header name="Content-Type" value="application/json"/>
      </http>
    </request>
    <!-- thinktime：两次请求之间的间隔时间，一般小于10s -->
    <thinktime value="1"/>
      </for>
    </session>
      </sessions>
    </tsung> 
  

如果被测接口需要登录跳转，可以指明跳转条件：   
 

    <!-- if need login, http 302 will be return -->
    <request subst="true">
      <dyn_variable name="redirect1" re="Location: ((http|https)://.*)\r"/>
      <http url="@api" method="@method" contents="@contents" version="1.1"></http>
    </request>
    
    <if var="redirect1" neq="">
      <request subst="true">
    <!-- 这里可以使用xpath提取页面中的元素值 -->
    <dyn_variable name="lt" xpath="//input[@name='lt']/@value"/>
    <dyn_variable name="s_uuid" xpath="//input[@name='s_uuid']/@value"/>
    <dyn_variable name="eventId" xpath="//input[@name='_eventId']/@value"/>
    <http url="%%_redirect1%%" method="GET"></http>
      </request>
    
      <request subst="true">
    <dyn_variable name="redirect2" re="Location: (http://.*)\r"/>
    <http url="%%_redirect1%%" method="POST" contents="username=%%_username%%&amp;password=%%_password%%&amp;lt=%%_lt%%&amp;s_uuid=%%_s_uuid%%&amp;_eventId=%%_eventId%%&amp;j_captcha_response=''"></http>
      </request>
    
      <request subst="true">
    <dyn_variable name="redirect3" re="Location: (http://.*)\r"/>
    <http url="%%_redirect2%%" method="GET"></http>
      </request>
    
      <request subst="true">
    <http url="%%_redirect3%%" method="@method" contents="@contents"></http>
      </request>
    </if>
    

4、通过shell控制tsung

如果每次使用tsung -f tsung.xml start运行tsung，那么每次修改测试接口或者压力改变都需要修改xml，非常麻烦，我写了一个shell脚本，替换上面的tsung.xml中以@开头的变量

    #!/bin/bash
    defaultTestFile="$HOME/tsung_test.xml"
    defaultUser=20
    defaultDuration=100
    # s
    defaultThinktime=1
    defaultServer="tomcat1"
    defaultPort=9000
    defaultApi="/test"
    defaultMethod="POST"
    defaultLoopCount=50
    defaultMaxuser=5000
    
    while [ $# -gt 0 ]; do
      case "$1" in
    -f|--testFile)
    testFile=$2
    shift 
    shift ;;
    -u|--user)
    user=$2
    shift 
    shift ;;
    -d|--duration)
    duration=$2
    shift 
    shift ;;
    -t|--thinktime)
    thinktime=$2
    shift 
    shift ;;
    -s|--server)
    server=$2
    shift 
    shift ;;
    -p|--port)
    port=$2
    shift 
    shift ;;
    -a|--api)
    api=$2
    shift 
    shift ;;
    -m|--method)
    method=$2
    shift 
    shift ;;
    -l|--loopCount)
    loopCount=$2
    shift 
    shift ;;
    -x|--maxuser)
    maxuser=$2
    shift 
    shift ;;
    -h|--help)
    echo "-f | --testFile: tsung test file xml,default $defaultTestFile"
    echo "-u | --user: user number per second, default $defaultUser"
    echo "-x | --maxuser: max user number, default $defaultMaxuser"
    echo "-d | --duration: times used to generate user,default $defaultDuration s"
    echo "-t | --thinktime: the inteval time between two request,default $defaultThinktime s"
    echo "-l | --loopCount: Each user's request number,default $defaultLoopCount"
    echo "-s | --server: play server,default $defaultServer"
    echo "-p | --port: play server http port,default $defaultPort"
    echo "-a | --api: api, default $defaultApi"
    echo "-m | --method: POST/GET,default $defaultMethod"
    echo "-h | --help: print this help"
    shift
    exit 1
    ;;
    --)
      shift
      break
      ;;
    *)
      echo "wrong input:$1,use -h or --help see how to use" 1>&2
      exit 1
      ;;
      esac
    done
    
    processName="tsung"
    pid=`ps aux | grep $processName | grep -v grep | awk '{print $2}'`
    #convert from string to array
    pid=($pid)
    if [ ${#pid[*]} -gt 3 ]; then
      echo "warning!!! a $processName process is running,please wait"
      exit 1
    fi
    
    #env
    #set default parameters
    testFile=${testFile:=$defaultTestFile}
    user=${user:=$defaultUser}
    duration=${duration:=$defaultDuration}
    thinktime=${thinktime:=$defaultThinktime}
    server=${server:=$defaultServer}
    port=${port:=$defaultPort}
    api=${api:=$defaultApi}
    method=${method:=$defaultMethod}
    loopCount=${loopCount:=$defaultLoopCount}
    maxuser=${maxuser:=$defaultMaxuser}
    
    #key of params is nodname in tusng_test.xml file
    declare -A params
    params=( \
      ["user"]=$user \
      ["maxuser"]=$maxuser \
      ["duration"]=$duration \
      ["thinktime"]=$thinktime \
      ["server"]=$server \
      ["port"]=$port \
      ["api"]=$api \
      ["method"]=$method \
      ["loopCount"]=$loopCount \
      )
    reportPath="$HOME/.tsung/log"
    currentTest=`date +%Y%m%d-%H%M`
    reportPath="$reportPath/$currentTest"
    mkdir -p $reportPath
    #deal with jmx file
    cp $testFile $reportPath
    currentTestFile="$reportPath/tsung_test.xml"
    function replace(){
      echo "$1:$2" | tee -a "$reportPath/test.env"
      #change / to \/ for sed 
      local val=${2//\//\\\/}
      sed -i "s/@${1}/${val}/" $currentTestFile
    }
    for key in ${!params[*]}
    do
      replace $key ${params[$key]}
    done
    #start tsung 
    tsung -f $currentTestFile start &
    wait %1
    cd $reportPath
    /usr/local/lib/tsung/bin/tsung_stats.pl


5、tsung结果分析

tsung生成的测试报告都放在$HOME/.tsung/log下，以日期加时间的方式命名，如：`.tsung/log/20150407-1951`，其中最重要的几张图是

- tsung产生的用户数曲线图 .tsung/log/20150407-1951/images/graphes-Users-simultaneous.png
 ![](http://static.oschina.net/uploads/space/2015/0706/135050_sSxO_780347.png)

Y轴代表每秒用户数，tsung每秒会产生一批用户，这个统计结果是每十秒统计一次，所有的图的起始位置显示的是0，其实是第一个10秒


- http接口响应数曲线图（TPS） .tsung/log/20150407-1951/images/graphes-HTTP_CODE-rate.png
 ![](http://static.oschina.net/uploads/space/2015/0706/135106_dKTc_780347.png)
 
 Y轴是每秒响应数，右上角的200是http状态码，如果有多个状态码，会有多条不同颜色的曲线。

- http接口响应时间曲线图 .tsung/log/20150407-1951/images/graphes-Perfs-mean.png

![](http://static.oschina.net/uploads/space/2015/0706/135120_sPqf_780347.png)

  Y轴是接口响应时间，单位是毫秒，request的线代表请求响应总耗时，connect的线代表tcp链接建立的时间。

6、主要统计信息  
Tsung统计数据是平均每十秒重置一次，所以这里的响应时间（连接、请求、页面、会话）是指每十秒的平均响应时间；  
connect： 表示 每个连接持续时间；  
Hightest 10sec mean	连接最长持续时间  
Lowest 10sec mean	连接最短持续时间  
Highest rate 	每秒最高建立连接速率  
Mean	平均每个连接持续时间  
Count	总连接数  
page： 表示 每个请求集合的响应时间,（一个页面表示一组没有被thinktime间隔的请求）  
request： 表示 每个请求的响应时间；  
Hightest 10sec mean	请求最长响应时间  
Lowest 10sec mean	请求最短响应时间  
Highest rate 	请求最快发送速率  
Mean	平均每个请求响应时间  
Count	总请求数  
session： 表示 每个用户会话持续时间；  
Hightest 10sec mean	会话最长持续时间  
Lowest 10sec mean	会话最短持续时间  
Highest rate 	每秒最高进行会话速率  
Mean	平均每个会话持续时间  
Count	总会话数  
 
7、数据流量统计  
size_rcv: 表示 响应请求数据量  
size_sent:表示 发送请求数据量  
Hightest rate	每秒最高 响应/发送 请求数据量  
Total 	响应/发送 请求总数据量  
 
8、计数统计  
connected	表示会话开始且尚未结束，并且已建立连接的最大用户数  
finished_user_count	表示已经完成会话的最大用户数  
users	表示会话开始且尚未结束的最大用户数  
users_count	表示Tsung总共生成的用户总数  
 
9、错误统计  
Error_abort_max_conn_retries	重新尝试连接错误  
Error_connect_timeout	连接超时错误  
Error_connect_nxdomain	不存在的域错误  
Error_unknown	位置错误  
 
Highest rate 	发生错误最高速率  
Total number	发生该错误总个数  
   
10、http返回状态码统计  
200：表示客户端请求已成功响应  
Highest rate	状态码返回最高速率  
Total number	返回状态码的总个数  
