---
layout: post
title: Hadoop2.8.3伪分布式搭建
date: 2018-3-07
categories: blog
tags: 学习篇
description: 大数据这么火，如何搭建一个最简单的集群！
---

# 1.准备Linux环境 #
## 1.1 配置虚拟机 ##
- 点击VMware快捷方式，右键打开文件所在位置 -> 双击vmnetcfg.exe -> VMnet1 host-only ->修改subnet ip 设置网段：192.168.8.0 子网掩码：255.255.255.0 -> apply -> ok


- 回到windows --> 打开网络和共享中心 -> 更改适配器设置 -> 右键VMnet1 -> 属性 -> 双击IPv4 -> 设置windows的IP：192.168.8.100 子网掩码：255.255.255.0 -> 点击确定


- 在虚拟软件上 --My Computer -> 选中虚拟机 -> 右键 -> settings -> network adapter -> host only -> ok
## 1.2 修改主机名
	vim /etc/sysconfig/network
## 1.3 修改IP ##
两种方式：

- 第一种：通过Linux图形界面进行修改（强烈推荐）
进入Linux图形界面 -> 右键点击右上方的两个小电脑 -> 点击Edit connections -> 选中当前网络System eth0 -> 点击edit按钮 -> 选择IPv4 -> method选择为manual -> 点击add按钮 -> 添加IP：192.168.8.118 子网掩码：255.255.255.0 网关：192.168.1.1 -> apply

- 第二种：修改配置文件方式
	
	`vim /etc/sysconfig/network-scripts/ifcfg-eth0`

## 1.4 修改主机名和IP的映射关系 ##

    vim /etc/hosts

## 1.5 关闭防火墙 ##

#### 查看防火墙状态 ####

	service iptables status

####关闭防火墙 ####

	service iptables stop

####查看防火墙开机启动状态 ####

	chkconfig iptables --list

####关闭防火墙开机启动 ####

	chkconfig iptables off

## 1.6 重启Linux ##
	reboot

# 2 安装JDK

####创建文件夹 ####

	mkdir /usr/java

####解压

	tar -zxvf jdk-8u162-linux-x64.tar.gz -C /usr/java/

####将java添加到环境变量中 ####

	vim /etc/profile

####在文件的最后添加 ####

	export JAVA_HOME=/usr/java/jdk1.8.0_162
	export PATH=$PATH:$JAVA_HOME/bin

####刷新配置 ####

	source /etc/profile

#3 安装hadoop2.8.3 #

##3.1 配置hadoop ##

####配置hadoop-env.sh ####

	vim hadoop-env.sh

####修改JAVA_HOME ####

	export JAVA_HOME=/usr/java/jdk1.7.0_65

####配置core-site.xml ####

	<!-- 制定HDFS的（NameNode）的地址 -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://itcast01:9000</value>
	</property>
	<!-- 指定hadoop运行时产生文件的存储目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/itcast/hadoop-2.4.1/tmp</value>
	</property>

####配置hdfs-site.xml ####

	<!-- 指定HDFS副本的数量 -->
	<property>
		<name>dfs.replication</name>
		<value>1</value>
    </property>

####配置yarn-site.xml ####

	<!-- 指定YARN的老大（ResourceManager）的地址 -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>itcast01</value>
    </property>
	<!-- reducer获取数据的方式 -->
    <property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
    </property>

##3.2 将hadoop添加到环境变量 ##

	vim /etc/proflie
	export JAVA_HOME=/usr/java/jdk1.8.0_162
	export HADOOP_HOME=/itcast/hadoop-2.8.3
	export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

	source /etc/profile

##3.3 格式化namenode ##

	hdfs namenode -format (hadoop namenode -format)

##3.4 启动hadoop ##

	先启动HDFS
	sbin/start-dfs.sh
	
	再启动YARN
	sbin/start-yarn.sh

##3.5 验证是否启动成功 ##

	使用jps命令验证
	27408 NameNode
	28218 Jps
	27643 SecondaryNameNode
	28066 NodeManager
	27803 ResourceManager
	27512 DataNode
	
####用浏览器打开以下页面测试搭建是否成功 ####

	http://192.168.8.118:50070 （HDFS管理界面）
	http://192.168.8.118:8088 （MR管理界面）

#4 配置ssh免登录 #

	#生成ssh免登陆密钥
	#进入到我的home目录

	cd ~/.ssh

	ssh-keygen -t rsa （按四下回车）
	执行完这个命令后，会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）
	将公钥拷贝到要免登陆的机器上
	ssh-copy-id localhost
	

