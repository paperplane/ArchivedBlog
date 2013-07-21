---
title: MCollective Security Config
date: '2013-04-22'
description:
categories: ['MCollective','DevOps']
tags: ['MCollective','ActiveMQ']
---
##MCollective安全配置##

***

<div>
    <ul>
        <p>作为 MCollective 重要部分就是考虑安全。默认情况下允许所有的代理和所有节点所有代理
        进行通信。问题是通过这种方法如果在某个节点有一个不受信任的用户他就能够安装一个客户
        端应用程序来从 server 配置文件中读取用户名/密码从而控制整个体系结构。本部分分析在
        MCollective 使用过程中的安全问题，并提供几种安全方案。</p>
    </ul>
    <hr>
    <ul>
        <li><a href="{{urls.media}}/pdf/mcollective-security.pdf"}}"><img src="{{urls.media}}/pdf/attach.png">请点击查看文档详细内容
        </a></li>
    </ul>
</div>
