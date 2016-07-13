---
layout: post
title:  "Linux自动备份cisco设备配置文件 "
categories: Linux笔记
excerpt: Linux
---

* 系统 Centos 6.7
* cisoc 2960 4506

##1.安装tftp及telnet
	
	yum install tftp-server telnet -y
	
###配置tfyp
	
	vim /etc/xinetd.d/tftp
	
	    service tftp
    {
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /bak 					#指定上传目录
        disable                 = no						#改为no 默认为yes不开启
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
     }

 	 重启服务器 /etc/init.d/xinetd restart

###设置开机启动tftp
	
	vim /etc/rc.d/rc.local
	#增加一行
	/etc/init.d/xinetd restart

###编写自动备份脚本

	#!/bin/bash
	#bakup the switch files to tftp server
	#programming by yzjia
	# 2016/07/12
	#####################################
	time=`date +"%Y%m%d"`
	TELNET_NAME=xx   		 		#用户名
	TELNET_PASSWD=xx				#telnet密码
	ENABLE_PASSWD=xx				#enable密码
	TFTP_IP=x.x.x.x					#TFTP服务器地址
	#function
	function bakshell {
    FILE="$2"_config_"$time"
    touch /bak/$FILE				#需要在循环中提前创建备份文件，否则会上传失败
    chmod 666 /bak/$FILE			#为其他账户增加写的权限
    (
   		sleep 1;					#设置休眠时间，放置因网络延迟导致的备份失败
   		echo $TELNET_NAME
   		sleep 1;
  		echo $TELNET_PASSWD
   		sleep 1;
   		echo "enable";
   		sleep 1;
   		echo $ENABLE_PASSWD
   		sleep 3;
   		echo "copy running-config tftp";
   		sleep 1;
   		echo $TFTP_IP
   		sleep 1;
   		echo $FILE
   		sleep 15;
   		echo "exit";
   	) |telnet $1 |tee -a /var/log/sw/switch_backup_$time.log   
    }

	##zhixing shell
	bakshell 172.16.100.11 L1 		 #备份设备地址及保存文件名称，如有更多设备按格式添加即可
	bakshell 172.16.100.31 L3-1
	bakshell 172.16.100.32 L3-2

###设置计划任务
	crontab -e
	00 23 1 * * sh /bak/swbak.sh	 #每月1号23点自动备份

	
