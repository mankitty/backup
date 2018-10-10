---
title: crond + rsyslog + logrotate 创建日志系统的安全运行环境
date: 2017-05-14 09:55:23
tags:
categories: LOG
---

## crond
## crond简介
crond作为linux系统自带的可按照配置文件定时执行个各种任务的进程，使用性还是相当不错的，特性很稳定
### 启动参数以及说明(OpenWrt)
	crond --help
	Usage: crond -fbS -l N -L LOGFILE -c DIR
		-f      Foreground
        -b      Background (default)
        -S      Log to syslog (default)
        -l N    Set log level. Most verbose:0, default:8
        -L FILE Log to FILE
        -c DIR  Cron dir. Default:/etc/crontabs
### 配置文件格式
	```bash
    * * * * *  执行的任务,具体含义如下
    # Example of job definition:
	# .---------------- minute (0 - 59)
	# |  .------------- hour (0 - 23)
	# |  |  .---------- day of month (1 - 31)
	# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
	# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
	# |  |  |  |  |
	# *  *  *  *  * command to be executed
    ```
### 特殊符号

| 符号        | 含义  |  举例  |
| :---:   | :-----  | :----  |
|*| *号表示任意时间 |比如 “* * * * * 任务” 就代表每分钟执行一次|
|,|	代表不连续的时间|比如 “ 0 1,3 * * * 任务” 就代表每天的1点整，3点整分别都执行一次|
|-|代表连续的时间范围|比如 “0 2 * * 1-3 任务” 就代表每周一到周三的凌晨2点0分执行任务|
|*/n|代表每隔多久执行一次|比如 “*/30 * * * * 任务” 就代表每三十分钟执行一次任务|

### 注意事项
	1. 六个选项都不能为空,必须填写,如果不确定可以使用 * 表示任意时间
	2. crontab定时任务,最小时间是分钟,最大是月,不能指定多少秒或多少年
	3. 定时任务中,最好使用绝对路径,因为进程自身所设置的环境变量可能与你使用的不相同

## rsyslog
### rsyslog简介
rsyslog最主要的作用是用来收集各个模块发送过来的日志信息，并根据规则做相应的记录
### rsyslog的优势
	1. TCP 实现数据的可靠传输
	2. 精细的输出格式控制以及对消息的强大过滤能力
	3. 高精度时间戳,队列操作(内存,磁盘以及混合模式等),支持数据的加密和压缩传输等
### rsyslog配置文件(自己的配置)
	cat /etc/rsyslog.conf
    
	# Provides TCP syslog reception
	#$ModLoad imtcp
	#$InputTCPServerRun 514

	#### GLOBAL DIRECTIVES ####

	# Use default timestamp format
	$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

	# File syncing capability is disabled by default. This feature is usually not required,
	# not useful and an extreme performance hit
	#$ActionFileEnableSync on

	# Include all config files in /etc/rsyslog.d/
	$IncludeConfig /etc/rsyslog.d/*.conf

	# ### begin forwarding rule ###
	# The statement between the begin ... end define a SINGLE forwarding
	# rule. They belong together, do NOT split them. If you create multiple
	# forwarding rules, duplicate the whole block!
	# Remote Logging (we use TCP for reliable delivery)
	#
	# An on-disk queue is created for this action. If the remote host is
	# down, messages are spooled to disk and sent when it is up again.
	#$WorkDirectory /var/lib/rsyslog # where to place spool files
	#$ActionQueueFileName fwdRule1 # unique name prefix for spool files
	#$ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
	#$ActionQueueSaveOnShutdown on # save messages to disk on shutdown
	#$ActionQueueType LinkedList   # run asynchronously
	#$ActionResumeRetryCount -1    # infinite retries if host is down
	# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
	#*.* @@remote-host:514
	# ### end of the forwarding rule ###


	cat /etc/rsyslog.d/default.conf
    
    #### RULES ####

	# Log all kernel messages to the console.
	# Logging much else clutters up the screen.
	#kern.*                                                 /dev/console

	# Log anything (except mail) of level info or higher.
	# Don't log private authentication messages!
	*.info;mail.none;authpriv.none;cron.none                /var/log/messages

	# The authpriv file has restricted access.
	authpriv.*                                              /var/log/secure

	# Log all the mail messages in one place.
	mail.*                                                  -/var/log/maillog

	# Log cron stuff
	cron.*                                                  /var/log/cron

	# Everybody gets emergency messages
	*.emerg                                                 *

	# Save news errors of level crit and higher in a special file.
	uucp,news.crit                                          /var/log/spooler

	# Save boot messages also to boot.log and send to 192.168.1.1:514
	local7.*                   /var/log/boot.log	@192.168.1.1:514

	:msg,contains,"WAN_ONLINE_OFFLINE_RECORD"          /var/log/wan_online_offline_record.log

### 注意事项
	我暂时仅仅研究到这部分的使用,在使用contains关键字时，会使相同的日志分别导向不同的日志文件，可用如下方法解决：
	if $msg contains "[wd_user]" then
    	/var/log/wifidog/wifidog_wd_user_record.log
	else {
        # Log anything (except mail) of level info or higher.
        # # Don't log private authentication messages!
        *.info;mail.none;authpriv.none;cron.none        /var/log/messages
	}
	其他的日后再研究
    
## logrotate
### logrotate简介
    man logrotate
    logrotate  is  designed  to ease administration of systems that generate large numbers of log files.  It allows automatic rotation, compression, removal, and mailing of log files.  Each log file may be handled daily, weekly, monthly,  or  when it grows too large.
### 转储日志的脚本
	cat /etc/cron.minutes/logrotate		--crond执行的定时任务，每分钟检查一次

    #!/bin/sh

	/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
	EXITVALUE=$?
	if [ $EXITVALUE != 0 ]; then
	    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
	fi
	exit 0
### logrotate配置文件详细
	cat /etc/logrotate.conf
    # see "man logrotate" for details
    # rotate log files weekly
    weekly

    # keep 4 weeks worth of backlogs
    rotate 4

    # file max size is 16M
    maxsize 16M

	# file min size is 16M
    # minsize 16M

    # create new (empty) log files after rotating old ones
    create

    # use date as a suffix of the rotated file
    #dateext

	# Do not archive  old versions of log files with date extension (this overrides the dateext option).
    nodateext
    # uncomment this if you want your log files compressed
    #compress

	# Rotate the log file even if it is empty, overriding the notifempty option (ifempty is the default).
    notifempty

	# Do not mail old log files to any address.
    nomail

    # RPM packages drop log rotation information into this directory
    include /etc/logrotate.d

### 注意事项
	在/etc/logrotate.conf文件中,全部的定义都可以认为是全局的,若在局部再次定义相同的表项，则会覆盖全局表项
    暂时我用到的就这么多，事后有空再补充