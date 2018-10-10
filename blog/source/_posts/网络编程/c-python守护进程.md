---
title: c-python守护进程
date: 2018-09-17 10:30:25
tags:
categories: 网络编程
---
## python 守护进程
```
    def try_daemon(self,stdin='/dev/null', stdout='/dev/null', stderr='/dev/null'):
        # 重定向标准文件描述符（默认情况下定向到/dev/null）
        try:
            pid = os.fork()
            # 父进程(会话组头领进程)退出
            if pid > 0:
                sys.exit(0)  # 父进程退出

            # chdir确认进程不保持任何目录于使用状态,也可以改变到对于守护程序运行重要的文件所在目录
            os.chdir("/")
            # 调用umask(0)以便拥有对于写的任何东西的完全控制
            os.umask(0)
            # setsid调用成功后，进程成为新的会话组长和新的进程组长，并与原来的登录会话和进程组脱离
            os.setsid()

            pid = os.fork()
            if pid > 0:
                sys.exit(0)  # 第二个父进程退出

            # 进程已经是守护进程了，重定向标准文件描述符
            for f in sys.stdout, sys.stderr:
                f.flush()
                si = open(stdin, 'w+')
                so = open(stdout, 'w+')
                se = open(stderr, 'w+', 0)
                os.dup2(si.fileno(), sys.stdin.fileno())
                os.dup2(so.fileno(), sys.stdout.fileno())
                os.dup2(se.fileno(), sys.stderr.fileno())

        except OSError, e:
            LOG.error("fork failed: (%d) %s" % (e.errno, e.strerror))
            sys.exit(1)
```
## C 守护进程
```
void try_deamon(void)
{
	int i;
	pid_t pid;
	
	pid = fork();
	
	if (pid < 0)
	{
		image_msg_log(LOG_ERR,"first fork failed");
		exit(1);
	}
	else if (pid > 0)
	{
		exit(0);
	}
	
	if (FAILURE == setsid())
	{
		image_msg_log(LOG_ERR,"set ssid failed");
		exit(1);
	}

	pid = fork();
	
	if (pid < 0)
	{
		image_msg_log(LOG_ERR,"second fork failed");
		exit(1);
	}
	else if (pid > 0)
	{
		exit(0);
	}
	
	for(i = 0;i < NOFILE;i++)
		close(i); 

	if (FAILURE == chdir("/"))
	{
		image_msg_log(LOG_ERR,"change work dir failed");
		exit(1);
	}
	umask(022);

	return;
}
```