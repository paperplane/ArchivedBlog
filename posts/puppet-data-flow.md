---
title: Puppet Data Flow
tagline: Puppet数据流程解析
date: '2013-09-17'
description:
categories: ['Puppet']
tags: ['Puppet','DevOps']
---

##Puppet数据流程##

前面的部分对Puppet进行了概述，并对Puppet内部实现原理和认证过程进行了介绍。本部分将主要围绕puppet数据流程进行介绍，详细描述puppet是如何管理单个节点的数据流程。这部分在之前的概述和内部原理都有所涉及，单独选出来作为一部分介绍希望对数据流程和流程涉及的组件进行更深的理解。

***

首先回顾Puppet是如何工作：

![puppet数据流]({{urls.media}}/puppet/internals.png)

+ 定义：采用Puppet的描述性语言，以可重用的模块形式来定义资源及资源间关系的图。这些模块定义基础设施的预期状态。

+ 模拟 : 有了这些资源及其关系图，Puppet能够模拟部署环境，让你在不影响基础设施的前提下测试你的改变。使用noop参数或者多环境等实现。

+ 执行 : puppet比较你的系统和定义的预期状态，然后自动地使你的系统符合定义的预期状态。这个过程是相对透明的过程。

+ 报告 : Puppet Dashboard展示收集到的组件关系和所有的改变，并提供安全和授权措施查看。此外Puppet也提供有开放API以结合第三方监控工具使用。


由此我们可以看出 Puppet定义并维护一个节点的预期角色，同时Puppet能够管理配置改变。理解前者很重要，良好的Puppet 模块代表Agent 角色的分类，角色信息和facter信息等组成了此节点更为丰富的属性信息。

***

看完这个过程可能最大的疑惑就是哪些是在客户端执行，哪些又在服务端执行，客户端和服务端是如何交互的。

下面以以数据流程图为例详细描述Puppet如何管理单个节点数据流程。这幅图在之前已经描述过。

![puppet数据流]({{urls.media}}/puppet/dataflow.png)

+ Agent 进程收集关于运行此进程的主机信息(facts，也包括自定义的facts)，然后将这些信息传递给Master

+ Master 使用这些系统信息和本地Puppet模块清单来编译针对特定主机的配置(Catalog),并将结果返回给Agent

+ Agent 会比较配置与本机状态，并应用此配置，对于状态不同部分做出改变，并将结果(report)报告给服务器

+ Puppet的开放API同样可以将这些数据(report)发送给第三方工具来进行分析，例如同Nagios这样的监控工具结合


***

再看每个流程涉及到的组件：

####facts####

    自动维护资产清单 这些清单包括但不限于：domain,ip,cpu,mem等信息。

![puppet数据流]({{urls.media}}/puppet/facts.png)

当然Puppet还可以是自定义Facts：

$modulename/lib/facter/certname.rb
![puppet数据流]({{urls.media}}/puppet/fact-custom.png)

Facts很方便定义这些静态属性，对于动态属性，Puppet生态系统同样提供了相应工具---Hiera是其中的新星。


####catalog####

    可以理解为资源的列表解析

采用命令puppet catalog可以查看，puppet catalog find --render-as yaml offsum001.web.djt而且支持多种展现方式比如yaml，json等。

![puppet数据流]({{urls.media}}/puppet/catalog.png)

####reporting####

    每一个资源相关连的每一次改变

![puppet数据流]({{urls.media}}/puppet/reporting.png)

目前支持四种预定义的报告处理器和自定义报告处理器

    * HTTP/HTTPS
    * Log
    * Store
    * TagMail
    * Custom Processors
        * irc
        * twitter
        * jabber
        * growl 

还有一种值得关注的报告处理器就是PuppetDB，目前正在快速发展。简单来说，PuppetDB是一个puppet数据仓库，它能够检索平台生成的所有数据并管理这些数据的存储。它能够存储节点目录，facts和reports等。后面会继续介绍。

***

下面换一种角度来看数据流程

![puppet数据流]({{urls.media}}/puppet/data-flow-technical.png)

+ 节点信息存储在外部节点分类器或LDAP等，从Dashboard中可以获取节点信息

+ Master在执行manifeasts前会同步插件（这里的插件包括facts ,types, provides 如果你写过这些自定义插件那么执行前会显式出来 ）以及 Fact Code

![puppet数据流]({{urls.media}}/puppet/plugins.png)

+ 客户端会在本地计算这些Facts 值 ，然后将这些信息发送给Master并要求Catalog

+ Master根据facts和Manifests生成符合特定节点的配置，这个配置是一种中间编译结果成为Catalog，然后Master将CataLog发送给客户端

+ 客户端根据CataLog定义的状态，对比自身状态，对于不同的部分作出改变。在应用配置期间，可能会需要请求文件等操作，这些还需要同Master进行交互，等到配置应用完毕，将执行结果以报告形式发送给Master。
