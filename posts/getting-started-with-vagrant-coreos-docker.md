---
title: 'Getting Started With Vagrant/Coreos/Docker'
date: '2014-05-17'
description:
categories: ['DevOps']
tags: [Vagrant, CoreOS, Docker]
---
这是一篇关于Vagrant, CoreOS, Docker三个工具的入门介绍性文章，通过描述"使用Vagrant安装CoreOS", "在CoreOS中使用Docker"来介绍三个工具的入门使用。当然要首先介绍”在Mac OS X上运行Vagrant”。

###Part I: Running Vagrant On Mac OS X

***

####Vagrant简介

Vagrant是一个创建和配置轻量级，可复制，可移植开发环境的工具，让配置开发环境更容易。一个简单的使用案例就是通过vagrant封装一个Linux开发环境，分发给团队成员。而每个人都可以在自己喜欢的桌面系统(Mac/Win/Linux)上开发程序，代码却能在封装好的环境里运行。得益于下面几个特点，Vagrant将极大改变我们的工作流程。
 
+ 快速安装

    下载和安装Vagrant非常快，无论是在MacOS X， Windows或者其他流行的Linux发行版，仅需数分钟时间。没有复杂的安装过程，仅仅是简单的使用系统标准安装工具。快速而且跨平台。
   
+ 极简配置

    Vagrant项目唯一配置文件Vagrantfile描述需要的机器类型，需要安装的软件以及如何访问该机器等。支持Puppet , Chef, Shell等配置管理工具，清单文件可通过同步目录实现。基础设施即代码。
	 
+ 工作简单
	
    运行命令”vagrant up"即可启动机器，vagrant会将所有配置组合完成完整开发环境配置。”代码在我机子上运行没有问题”这种说辞将成为历史，因为vagrant为团队中每个人创建了统一开发环境。

***

####Vagrant安装

Vagrant的强大很大程度源于她站在巨人的肩膀之上，她并不提供虚拟化的具体实现，而是借助Providers来实现，最初的Provider是VirtualBox，后来支持了VMware， AWS等。在这之上，通过Puppet, Chef,  Shell来实现软件安装与配置。Vagrant就是虚机管理工具 。

+ 安装VirtualBox与Vagrant 
	
	选择开源VirtualBox作为Provider，小巧免费，下载安装很简单。[传送门](https://www.virtualbox.org/wiki/Downloads)
	
	同样Vagrant安装也很简单，支持Mac, Win和常见Linux发行版。[传送门](http://www.vagrantup.com/downloads.html)

+ 镜像下载与添加

	需要下载基础镜像文件。可以直接下载，也可以通过命令box add下载，在[这里](http://www.vagrantbox.es/)有很多镜像。
		
		vagrant box add centos https://github.com/2creatives/vagrant-centos/releases/download/v6.5.1/centos65-x86_64-20131205.box
		
		centos 是box名字 链接是box镜像下载地址，如果已经下载，可直接指定文件路径

+ 环境初始化

	初始化环境。
	
		cd ~/vagrant
		vagrant init centos
	启动环境
	
		vagrant up
	登录环境
	
		vagrant ssh
	注 ：初始化完成会提示在当前目录生成文件Vagrantfile, 这是整个项目唯一配置文件。
	
	注：启动过程大致阶段：检查配置 — 设置网卡 — 启动系统 — 配置网络 — 其他配置— 共享目录 其实本质是对VirtualBox配置和启动过程的封装

	到此为止，开发环境就已经准备就绪，并且已经登录完成。可以进行开发工作和后续配置。

+ 相关设置项
	
	这里通过一个简单的例子来描述几个常见配置项。情景举例：在搭建好的环境安装Nginx开发环境，并且确保通过LocalHost和开发机IP都可访问，能够在本机提供Index页面覆盖Nginx默认Index页面。

	+ SSH登上开发机，使用Yum安装Nginx, 然后起服务。
		
			vagrant ssh
			sudo yum install -y nginx
			sudo service nginx start
	
	+ Vagrant默认使用NAT方式，将虚机端口映射到本地端口。也可以使用HOST_ONLY方式，修改配置项：config.vm.network "private_network", ip: “192.168.33.11" 这样开发机就有了IP
	
	+ 修改端口映射配置项：config.vm.network "forwarded_port", guest: 80, host: 8000 当然可以设置更多的端口映射
	
	+ 修改同步文件配置项：config.vm.synced_folder "html", "/usr/share/nginx/html", owner: "root", group: "root", create: true 这样可以在html目录下放置Nginx需要的文件
	
	+ 重启机器使配置生效:
            	
            vagrant reload
            echo testing >  html/index.html
            curl http://192.168.33.11/ /curl http://localhost:8000/ 

***

####Vagrant使用

+ Vagrant机器生命周期管理：
			
		init - up - ssh - suspend - resume - reload -  halt - destroy
	![图1]({{urls.media}}/docker/vagrant1.png)

+ Vagrant Cloud与Git工作流对照
	
		login - init - package - share
	![图2]({{urls.media}}/docker/vagrant2.png)
+ 其他命令
	+ init/box: 
			
			vagrant init lsomeoe/centos = vagrant box add centos image_link + vagrant init centos
	Vagrant Cloud允许所有人分享自己的Vagrantfile, 类似GitHub式托管配置文件。这样可以直接init在VagrantCloud上托管的Vagrantfile。init命令会帮助下载镜像并初始化环境。
	+ connect
			
			当使用vagrant share共享环境的时候，可通过connect命令连接至共享环境。
	
	+ provision
			
			vagrant集成了多种配置管理工具: Shell, Chef, Puppet等常见方式均支持。

	+ plugin
			
			用以管理插件：支持如下子命令：install. uninstall, update, list, lisence

***

####遇到问题
+ 编码问题
	
		vagrant ssh
		Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.8.0-39-generic x86_64)
		Documentation:  https://help.ubuntu.com/
		Last login: Sat May  3 09:02:01 2014 from 10.0.2.2
		-bash: warning: setlocale: LC_ALL: cannot change locale (zh_CN.UTF-8)
		-bash: warning: setlocale: LC_ALL: cannot change locale (zh_CN.UTF-8)
		-bash: warning: setlocale: LC_ALL: cannot change locale (zh_CN.UTF-8)
	原因是本机器编码设置为zh_CN.UTF-8 SSH登录过去
+ 异常错误

		Bringing machine 'default' up with 'virtualbox' provider...
		[default] Importing base box 'Cupcake'...
		[default] Matching MAC address for NAT networking...
		[default] Setting the name of the VM...
		[default] Clearing any previously set forwarded ports...
		[default] Creating shared folders metadata...
		[default] Clearing any previously set network interfaces...
		There was an error while executing `VBoxManage`, a CLI used by Vagrant for controlling VirtualBox. The command and stderr is shown below.
		Command: ["hostonlyif", "create"]
		Stderr: 0%...
		Progress state: NS_ERROR_FAILURE
		VBoxManage: error: Failed to create the host-only adapter

	错误信息如上：解决办法：
	
		sudo /Library/StartupItems/VirtualBox/VirtualBox restart(mac os x 10.9)
		sudo modprobe vboxnetadp(linux)
	vagrant在重启virtualbox虚机或启动虚机时有时候会遇到这种异常退出。基本重启virtaulbox服务即可。

+ SSH别名

	既然配置了IP, 也设置了端口那么能不能直接通过原生的SSH命令登录，而不是Vagrant SSH登录呢。必然是可以的，其实只需要确定好主机，端口，证书就行。 可以通过vagrant ssh-config来查看这些信息
		
		vagrant ssh-config
		ssh $(vagrant ssh-config | awk ‘{print “ -o “$1”=“$2""}’) localhost
	
		#!/bin/sh 
		PORT=$(vagrant ssh-config | grep Port | grep -o '[0-9]\+')
		ssh -q \    
			-o UserKnownHostsFile=/dev/null \    
			-o StrictHostKeyChecking=no \    
			-i ~/.vagrant.d/insecure_private_key \    
			vagrant@localhost \    
			-p $PORT  $@



