---
title: 'Troubleshooting 5minutes'
date: '2013-10-23'
description:
categories: ['Linux']
tags: ['Linux','troubleshooting']
---

##故障排除的第一个五分钟##

在处理日常运维、优化和扩展性问题的时候,经常碰到了各种不同规模的性能很差的系 统和基础设施(通常是大规模的,比如 CNN 或者世界银行的系统)。再加上修复时间紧迫、 奇葩的技术平台、缺少信息和文档,基本上这过程都会惨痛到让我们留下深刻的记忆。

造成故障的原因大多数情况是不明显的:这里有一些我们通常开始做的事情。

* * *
####获取上下文信息####

不要急于去服务器操作,需要搞清楚对服务器和这个特定故障到底了解多少。你不希望 浪费你的时间在黑暗中排除(故障)。
一些“必须了解”的问题:

+ 问题的确切表现是什么?没有反应?出现错误?
+ 问题什么时间开始被注意到?
+ 问题是否可重现?
+ 有无任何规律(比如每小时发生一次)?
+ 平台最近一次改变是什么(code, servers, stack)?
+ 问题是否会影响特定用户段(登录,登出,定位...)?
+ 有没有任何关于这个体系结构(物理或逻辑)的相关文档?
+ 是否有相关监控平台?Munin,Zabbix,Nagios, New Relic...或任何其他监控平台
+ 有没有任何(集中化)日志?Loggly, Airbrake, Graylog...

最后两点是最便利的信息来源,但是别期望太多:它们通常也是那些会失去的东西。虽 然没有这些信息,但是要做个纪录并让其恢复且继续运行。


**补充说明**:

    对于监控工具 Zabbix,Nagios 大家可能比较熟悉,简单介绍下 Munin 和 New Relic。

    + Munin 是一款网络化资源监控工具,能够帮助分析资源的趋势和刚刚发生了什么导致性能降低这些问题。它的设计特点是即插即用,有句话这样评价 Munin 因 Plugin 而亮。默认安装已经提供了大量画图。 Munin 是一个 server-nodes 结构,一个 server 对应多台 node,每台 node 都是被监控 节点,server 是统计 node 的数据并进行绘图生成 html 的节点。

    + New Relic 是一款基于 SaaS 的云端应用监测与管理平台, New Relic 提 供服务包括终端用户行为监控、应用监控、数据库监控、基础底层监控甚至 单个平台的监控,能为应用的健康提供实时的可预见性。支持用 Ruby、PHP、 Java、.NET 以及 Python 等语言写的应用以及 Android 和 IOS 平台移动 应用监控。

    三个日志工具简单介绍。

    + loggly 提供日志即服务(logging as a service)。它支持日志管理并允许用户很容易检索和导航日志。采用 Syslog 和提供 REST API 进行日志 收集,loggly 很容易安装并且提供对应用程序中心化视图,这意味着不需 要 SSH 到机器上进行 Tail 日志。loggly 也提供作为衡量应用程序性能的 日志分析。

    + Airbrake 提供错误管理即服务(error mangagement as a service), 收集其他应用程序产生的错误并进行结果聚合用以回顾。使用 Airbrake, 可以实时查看并追踪错误,从而快速修复这些问题。这种方式避免了直接查 看日志进行错误管理。同时它也支持 REST API。

    + Graylog 是开源 syslog 实现并将日志存储在 MongoDB 中,它由两部分组 成:采用 java 编写的服务端通过 TCP 或者 UDP 接收 syslog messages 并 存储在数据库中;采用 ROR 实现的 WEB 界面允许你查看日志消息。

* * *

####有哪些用户####

    $ w
    $ last

不是最关键的,但是你最好别在一台其他人测试的机器上进行故障排除,就像一个 人在厨房做饭就足够了。

**补充说明**

    w 执行这项指令可得知目前登入系统的用户有那些人,以及他们正在执行的程序。单独执行 w 命令会显示所有的用户,您也可指定用户名称,仅显示某位用户的相关信息。

![w command]({{urls.media}}/linux/tu1.png)

    last和lastb都是列出目前和过去登录的用户信息,这两个命令对于追踪一般账 号者的使用行为很有帮助。
    + last 查找文件/var/log/wtmp 并列出自从文件创建时登录和登出的用户列 表,列表显示用户名或 tty 名称。文件/var/log/wtmp 记录正确登录系统 者的用户信息,包括每个用户登录、注销及系统的启动、停机的时间。
    + lastb 命令与 last 类似,只是默认它显示的文件/var/log/btmp,该文件 记录错误登录时所使用账户信息。必须指出无论是读 btmp 还是 wtmp。都只 是默认行为。last(lastb)可以通过-f 参数指定文件。比如 last -f var/log/btmp 行为与 lastb 一样。

    与用户登录相关的日志文件(这些文件都可以作为 last[lastb]的参数)
    + /var/log/btmp – 记录所有失败登录信息
    + /var/log/lastlog — 记录所有用户的最近信息
    + /var/log/wtmp 或/var/log/utmp — 包含登录信息。使用 wtmp 可以找出谁正在登陆进入系统,谁使用命令显示这个文件或信息等。
    + /var/log/faillog – 包含用户登录失败信息。此外,错误登录命令也会记录在本文件中。

* * *

####刚刚执行了什么####

    $ history

查看历史通常是很好的;结合上一步知道的哪些用户在这台机器的信息。 简单思考过后,你可能会想更新环境变量 HISTTIMEFORMAT 来追踪这些命令执行的时间。没有什么比调查一段过时的命令列表让人更沮丧...

**补充说明**

    教训:个人有两次误删数据都是从历史命令中查出问题的

    两个与 history 相关的环境变量:$HISTSIZE $HISTTIMEFORMAT,设置后者可以解决默认历史命令不显示时间的问题

* * *

####正在运行什么####

    $ pstree -a
    $ ps aux

虽然 ps aux 的信息可能会很冗长,pstree -aup 却给你很好的浓缩的关于正在运行 的进程和调用进程的用户信息,可以纵览整个进程树结构。

**补充说明**

    PS 查看进程常见命令

    内存使用排序 ps -e -o "%C : %p : %z : %a"|sort -k5 -nr
    CPU 使用排序 ps-e -o"%C :%p:%z:%a"|sort -nr
    清除僵死进程 ps -eal | awk '{ if ($2 == "Z") {print $4}}' | kill -9

* * *

####监听的服务####

    $ netstat -ntlp
    $ netstat -nulp
    $ netstat -nxlp

虽然 netstat -nalp 能做所有的事情,我倾向于单独执行它们,主要是因为不想同时 查看所有的服务。

识别正在运行的服务并确定它们是否是期望运行的。查看不同的监听端口,你可以始终 将 PID 与 ps aux 的输出进行匹配;这会非常有用特别是当你杀掉同时在运行的多个 Java 或 Erlang 进程。

我们通常偏向于在每台机器上运行着较少数量的服务,必要时可以通过增加机器。如果 你看到很多监听端口你或许应该去思考并做进一步调查,看看有哪些可以清理或者重组。


**补充说明**

    netstat 常见统计

    netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a,S[a]}'
    netstat -an -t | grep ":80" | grep ESTABLISHED | awk '{printf"%s %s\n",$5,$6}' | sort

    ss 和 ip 取代 netstat【想要知道原因的请查看 http://roclinux.cn/?p=2420】

    在 man netstat 时会看到这句话 This program is obsolete. 替代 netstat 的是 ss 命令。替代 netstat -r 的是 ip route。替代 netstat -i 的是 ip -s link。替代 netstat -g 的是 ip maddr。具体对应关系如下 图所示。

![ss ip command]({{urls.media}}/linux/tu2.png)

* * *

####CPU和RAM####

    $ free -m
    $ uptime
    $ top
    $ htop

这应该能回答一些问题:

+ 是否有任何可用内存?是交换内存吗?
+ 是否有 CPU 剩余可用?服务器上有多少 CPU 核可用?是否有 CPU 核过载?
+ 什么导致最多负载?平均负载是多少?

**补充说明**

    load average 可以理解为每秒钟 CPU 等待运行的进程个数. 在 Linux 系统中, sar -q、uptime、w、top 等命令都会有系统平均负载 load average 的输出,那 么什么是系统平均负载呢? 系统平均负载被定义为在特定时间间隔内运行队列中 的平均任务数。如果一个进程满足以下条件则其就会位于运行队列中:

    + 它没有在等待 I/O 操作的结果
    + 它没有主动进入等待状态(也就是没有调用'wait') 
    + 没有被停止(例如:等待终止)

    htop 是一个交互式动态进程查看器。与 Linux 传统的 top 相比,htop 更加人性化。 它可让用户交互式操作,支持颜色主题,可横向或纵向滚动浏览进程列表,并支持 鼠标操作,杀进程也不需要进程号

![htop command]({{urls.media}}/linux/tu3.png)

* * *

####硬件####

    $ lspci
    $ dmidecode
    $ ethtool


依然有很多裸机服务器,这可以帮你回答以下问题:

+ 识别 RAID 卡(以及是否有 BBU 备用电池?),CPU,以及可用的内存槽位。这可能给 你一些在潜在的问题追踪或性能提升方面的提示。
+ 网卡是否正确设置?是否运行在半双工模式?网速是 10MBps?任何发送/接收错 误?


**补充说明**

    dmidecode 这款软件允许你在 Linux 系统下获取有关硬件方面的信息。 dmidecode 遵循 SMBIOS/DMI 标准,其输出的信息包括 BIOS、系统、主板、处 理器、内存、缓存等等。CMDB/监控系统获取机器硬件等基础信息比如 SN 号使用。

    ethtool 用于查询及设置网卡参数的命令
    + ethtool ethX //查询 ethX 网口基本设置
    + ethtool –h //显示 ethtool 的命令帮助(help)
    + ethtool –i ethX //查询 ethX 网口的相关信息
    + ethtool –d ethX //查询 ethX 网口注册性信息
    + ethtool –r ethX //重置 ethX 网口到自适应模式
    + ethtool –S ethX //查询 ethX 网口收发包统计
    + ethtool –sethX[speed10|100|1000]//设置网口速率10/100/1000M

* * *

####IO 性能####

    $ iostat -kx 2
    $ vmstat 2 10
    $ mpstat 2 10
    $ dstat --top-io --top-bio

这些命令对于分析后端性能非常有用。

+ 查看磁盘使用:盒子是否有文件系统/磁盘的磁盘使用率达到 100%
+ 交换分区目前是否在使用(si/so)?
+ 什么最占用 CPU:system?user?stolen(vm)?
+ dstat 是我的最爱。什么进程在使用 IO?是 MySQL 在吸收资源呢还是 PHP 进程?

**补充说明**

    dstat 使用简便记法:dstat 命令很强大,但是命令选项比较多很难记。但是对于做运维的却很好记忆只需要记住【-lamps】即可。

![dstat command]({{urls.media}}/linux/tu4.png)

    vmstat/iostat/mpstat
    + vmstat监视内存使用情况:vmstat是VirtualMeomoryStatistics(虚拟内存统计)的缩写,可对操作系统的虚拟内存、进程、CPU 活动进行监视。它是对系统的整体情况进行统计,不足之处是无法对某个进程进行深入分析。

    + iostat 监视 I/O 子系统情况:iostat 是 I/O statistics(输入/输出统 计)的缩写,iostat 工具将对系统的磁盘操作活动进行监视。它的特点是汇报磁盘活动统计情况,同时也会汇报出 CPU 使用情况。同 vmstat 一样,iostat 也有一个弱点,就是它不能对某个进程进行深入分析,仅对系统的整体情况进行分析。

    + mpstat 是 Multiprocessor Statistics 的缩写,是实时系统监控工具。其报告与 CPU 的一些统计信息,这些信息存放在/proc/stat 文件中。在多 CPUs 系统里,其不但能查看所有 CPU 的平均状况信息,而且能够查看特定 CPU 的信息。使用 mpstat -P ALL 可查看所有核。

* * *

####挂载点和文件系统####

    $ mount
    $ cat /etc/fstab
    $ vgs
    $ pvs
    $ lvs
    $ df -h
    $ lsof +D / /* beware not to kill your box */

+ 有多少文件系统被挂载?

+ 是否存在某些服务有专用挂载点(可能是 MySQL...)?

+ 文件系统的挂载选项:noatime?default?是否有文件系统被重新挂载为只读?

+ 磁盘是否有剩余空间?

+ 是否有任何大文件(删除文件)没有被刷新?

+ 如果磁盘空间成为问题是否有剩余空间用以扩展分区?

**补充说明**

    常见设备 mount 操作

    + USB 设备:mkdir /mnt/usb;mount -t vfat /dev/sda /mnt/usb
    + 光盘设备:mkdir /mnt/cdrom;mount -t iso9660 /dev/cdrom /mnt/cdrom
    + ISO 文件:mount -o loop pes6.iso /mnt/cdrom

    LVM 逻辑卷的创建,使用,删除
    + 创建逻辑卷
        + pvcreate 将物理硬盘格式化成 PV(物理卷)
        + vgcreate 创建 VG(卷组),并将 PV 加入到卷组
        + lvcreate 基于 VG(卷组)创建 LV(逻辑卷)
    + 使用逻辑卷: 格式化【如 mkfs.ext4】后再 mount
    + 删除逻辑卷: umount -> lvremove -> vgremove -> pvremove

* * *

####内核,中断和网络####

    $ sysctl -a | grep ...
    $ cat /proc/interrupts
    $ cat /proc/net/ip_conntrack /* may take some time on busy servers */ 
    $ netstat
    $ ss -s

+ 中断请求是否在 CPU 之间均衡负载?或者是否某个核因为网络中断,RAID 卡等造成 过载?

+ SWAP 设置多少?对于工作站 60 就够了,但是对于服务器这通常是一个坏主意:你 不希望你的服务器总是在交换。否则当数据在读写在进行磁盘读写时你的交换进程 将会被锁住。

+ conntrack_max 是否设置得足够高以处理网络流量?

+ TCP 连接不同状态花费多长时间(TIME_WAIT,...)?

+ netstat 在显示所有存在得连接时可能很慢,你可能会使用 ss 来获取一个概况。

查看[Linux TCP Tuning](http://www.lognormal.com/blog/2012/09/27/linux-tcpip-tuning/)以获取更多 信息作为如何优化网络栈指南。

关于中断亲和力以及线程化我之前有篇[博客](http://junqili.com/linux/深入理解 linux 内核中断及其特性-更新/)谈到过。

* * *

####系统日志和内核消息####

    $ dmesg
    $ less /var/log/messages
    $ less /var/log/secure
    $ less /var/log/auth

+ 查看任何错误或警告消息;它是否分解了关于你的 conntrack 连接数设置过高的问 题?

+ 是否看见硬件错误或者文件系统错误?

+ 能否关联这些事件的事件和之前获取到的信息?

**补充说明**

    /var/log/messages 包括整体系统信息,其中也包含系统启动期间的日志。此外, mail,cron,daemon,kern 和 auth 等内容也记录在 var/log/messages 日志中。 几乎系统发生的所有错误信息(或者重要信息)都会记录在这个文件里,这几乎是 系统发生莫名错误必查的文件。
    /var/log/dmesg — 包含内核缓冲信息(kernel ring buffer)。在系统启 动时,会在屏幕上显示许多与硬件有关的信息。可以用 dmesg 查看它们。
    /var/log/secure 包含验证和授权方面信息。例如,sshd 会将所有信息记录(其 中包括失败登录)在这里。基本上,只要涉及到需要输入账号密码的软件,那么当 登录时(无论正确错误)都会记录到这个文件。包括系统的 login 程序,图形界面 登录所使用的 gdm 程序,su, sudo 等程序还有网络连接的 ssh,telnet 等程序。
    /var/log/auth 包含系统授权信息,包括用户登录和使用的权限机制等。

* * *

####定时任务####

    $ ls /etc/cron* + cat
    $ for user in $(cat /etc/passwd | cut -f1 -d:); do crontab -l –u $user; done


+ 是否有定时任务执行太过频繁?

+ 是否存在某些用户的定时任务被隐藏?

+ 在出现问题时是否有一些备份在运行?

**补充说明**

    /var/log/cron — 每当 cron 进程开始一个工作时,就会将相关信息记录在这个文 件中。
    在/etc/cron.d/ 目录下建立的 cron 文件如果一个文件里有多条 cron,且最后 通过 2>&1 重定向输出,换行必须是 unix 格式的,如果是 dos 格式换行,bash 会 报错,cron 是无法正常执行的。
    [Cron 串行化](http://lilinux.tk/blog/2013/05/cron-sequential/)

* * *

####应用程序日志####

这里有很多可以分析的地方,但是它可能却是你开始不肯花时间的地方,你关注的是前面提 到的那些。就 LAMP 为例:

+ Apache & Nginx;追溯 access log 和 error log,查找 5XX 错误和可能的 limit_zone 错误。

+ MySQL; 查看 mysql.log 中的错误,追踪损坏的数据库表,同时 innodb 启动恢复 进程。

+ PHP-FPM;如果启用 php-slow log,深挖日志并尽量找到错误 (php,mysql,memcache...)。如果没有,那就启用。

+ Varnish; 在 varnishlog 和 varnishstat 中,检查 hit/miss 比率。是否有配置 中的规则未命中而导致最终用户回源。

+ HA-Proxy;后端的状态是什么?健康检查是否成功?是否达到前端或者后端最大队 列大小?

* * *

####总结####

经过这开始的 5 分钟你应该能够更好的了解到:

+ 正在运行什么

+ 问题是否与 IO/硬件/网络或配置(不好的代码,内核优化...)相关

+ 是否注意到某个现象:例如滥用数据库索引,或者太多 apache 工作进程

你甚至可能发现问题确切的最根本的原因。如果没有,在你了解了上面提到的这些信息之后, 也应该具有作为继续深挖的条件。


