---
title: 
date: '2013-06-15'
tagline: 解决Puppet调度问题
description:
categories: ['MCollective','DevOps']
tags: MCollective
---

##Puppet Commander简介##

Puppet Commander解决了即使在使用splay时，有时候也会有大量的checkins来hitting你的master并可能overwhelming它。

基本的理论就是通过使用控制puppetd代理的执行情况。Puppetd Agent代理能够知道有多少正在在运行的机器并且能够调度安排这些运行，因此能够确保在任何指定的时间可以调度运行当前给定数量的机器，这在一定程度上 会提高计划master的性能。但是这在执行puppetd(test run或者其他规定的执行)很繁忙时会有副作用。而puppet commander这时就会不去调度这些机器来运行，控制并发执行数量，使你master的资源空闲以供别的使用。

这是一个调用puppetd代理的工具。在puppetd的基础上，它主要实现了两个功能：-i/--interval,设置mcollective server的puppetd执行间隔 -m 允许最大并发执行机器数量。当然它也支持MCO工具所有功能，比如批量执行/批量睡眠、过滤机制等这些功能。该工具命令使用前需要停止所有mcolletive server的puppet daemon，并start puppetmaster。

***

####安装####

有两种安装方式，作为MCollective客户端使用 和作为Daemon使用。官网上介绍是作为Daemon(puppetcommanderd)使用，我尝试作为客户端程序使用没有问题，需要进一步封装。

前提：agentpuppetd安装并正常运行

要求：停止在每个机器上运行的puppet服务

源代码有三个文件，分别做如下处理：

    将puppetcommanderd脚本放在 /usr/sbin/目录下
    将puppetcommander.init重命名为puppetcommander放在/etc/ini.d/目录下，并启用(enable)该服务
    将模板配置文件puppetcommander.cfg（可修改）放在/etc/目录下
    另外创建文件/etc/sysconfig/puppetcommanderd可以设置一些设置选项比如安全插件或者MCOLLECTIVE_EXTRA_OPTS等

***

####配置####

提供的参考配置文件模板如下（puppetcommander.cfg）

    ---
    :filter: "country=/de|uk|za/"
    :interval: 30
    :concurrency: 2
    :randomize: true
    :logfile: /var/log/puppetcommander.log
    :daemonize: true

它会在de,uk和za节点上每30分钟运行一次，每次同时运行的节点不超过两个。它会在发现的节点上轮询并将日志写入相应文件。

这个配置文件的filter很受限制，所以现在关于过滤的配置已经在文件/etc/sysconfig/puppetcommanderd里，如下所示，并且可以多样化（应用之前提到的所有过滤机制）。

    export MCOLLECTIVE_EXTRA_OPTS="-W country=/de|uk|za/"

然后可以设置：filter为空，如下所示

    :filter: ""
