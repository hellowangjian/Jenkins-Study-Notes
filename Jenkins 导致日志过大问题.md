## Jenkins 导致日志过大问题


每日服务器状态检查发现测试服务器1的 /u 目录磁盘使用率达到 100%
```BASH
    ~]$ sudo df -h
    Filesystem           Size  Used Avail Use% Mounted on
    /dev/vda1             40G  5.6G   32G  15% /
    tmpfs                 16G  272K   16G   1% /dev/shm
    /dev/mapper/vg1-lv1  493G  468G   19M 100% /u
    /dev/mapper/vg1-lv2   99G  7.8G   86G   9% /opt
```

查看日志文件，发现 25 号日志和 catalina.out 高达 84G：
```BASH
    logs]$ ll -h
    -rw-r----- 1 weblogic weblogic 2.5K Nov 24 14:40 catalina.2017-11-24.log
    -rw-r----- 1 weblogic weblogic  84G Nov 25 23:59 catalina.2017-11-25.log
    -rw-r----- 1 weblogic weblogic 380K Nov 26 23:59 catalina.2017-11-26.log
    -rw-r----- 1 weblogic weblogic    0 Nov 27 08:59 catalina.2017-11-27.log
    -rw-r----- 1 weblogic weblogic  84G Nov 27 08:59 catalina.out
```

查看其中的日志内容显示很多如下信息：
```BASH
    logs]$ tail catalina.out
    [2017/11/27 10:34:32]   [DNSQuestion@1853689633 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@145344221 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@1187142283 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@2093416861 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@2051349795 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@1016898456 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@1751036034 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@743542781 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
    [2017/11/27 10:34:32]   [DNSQuestion@2125407057 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
```

经过 google 搜索发现，此原因是 [Jenkins 的一个 BUG](https://issues.jenkins-ci.org/browse/JENKINS-29490)，可以在 Jenkins 中配置，也可以在启动 Jenkins 的时候给 Java 加一个参数，关闭 Java DNSMultiCast 功能：

- [在 Jenkins 中设置不记录此日志](http://new.nginxs.net/read.php/post-201612071024/)

    系统管理 -> system log -> 日志级别 -> Name: javax.jmdns,Level: off

    ![](http://new.nginxs.net/storage/77fc2487.jpg)

    ![](http://new.nginxs.net/storage/172c9f74.jpg)

- [在启动 Jenkins 时关闭 Java DNSMultiCast 功能](https://wiki.jenkins.io/display/JENKINS/Features+controlled+by+system+properties)，命令如下：
```BASH
    java -Dhudson.DNSMultiCast.disabled=true -jar jenkins.war
```

我们使用的 Jenkins 是运行在 tomcat 中的，所以要在 catalina.sh 中设置即可：
```BASH
    vim $CATTALINA_HOME/bin/catalina.sh
    JAVA_OPTS="$JAVA_OPTS -Dhudson.DNSMultiCast.disabled=true"
```




<p align="right">BY Hunter.Wang(王建) </p>
<p align="right">2017.11.28 </p>
