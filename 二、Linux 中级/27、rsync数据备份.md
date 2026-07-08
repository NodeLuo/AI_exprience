
1、rsync传输模式（只要文件有r权限就可以）

```bash
1、本地模式
	Local:  rsync [OPTION...] SRC... [DEST]
	本地     命令    选项参数    源文件 目标位置
	
	示例：
		----------------------------------------------------
		#将a.txt拷贝到dir下
			[root@backup ~]# rsync -avz a.txt dir
			sending incremental file list
			a.txt
		
		#再次将a.txt拷贝到dir下，此时默认为增量拷贝
			此时进行的判断：
				a.文件没变 -> 跳过
				b.文件变了 -> 重新传整个文件
		----------------------------------------------------
		#拷贝dir1目录下的文件到dir2下
			[root@backup ~]# ll dir1
			total 0
			-rw-r--r-- 1 root root 0 Jul  8 09:25 a.txt
			-rw-r--r-- 1 root root 0 Jul  8 09:25 b.txt
		#不包含目录
			[root@backup ~]# rsync -avz dir1/ dir2  （类似cp dir1/* dir2）
			sending incremental file list
			./
			a.txt
			b.txt
		#包含目录
			[root@backup ~]# rsync -avz dir1 dir2
			sending incremental file list
			dir1/
			dir1/a.txt
			dir1/b.txt
		----------------------------------------------------
		#删除拷贝不同步内容
			[root@backup ~]# ll dir1
			total 0
			-rw-r--r-- 1 root root 0 Jul  8 09:25 a.txt
			[root@backup ~]# ll dir2
			total 0
			-rw-r--r-- 1 root root 0 Jul  8 09:25 a.txt
			-rw-r--r-- 1 root root 0 Jul  8 09:25 b.txt
			[root@backup ~]# rsync -avz --delete dir1/ dir2
			sending incremental file list
			deleting b.txt
			./
		----------------------------------------------------

2、远程模式
    Access via remote shell:
         Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
         拉取   命令    参数选项     用户  主机  源文件  目标位置
         #HOST:主机IP地址 域名 主机名
         Push: rsync [OPTION...] SRC... [USER@]HOST:DEST
         推送   命令    参数选项     源文件  用户  主机  目标位置
    
    远端路径（默认家目录）：
	    - root@192.168.1.100: → 远端路径 `/root`
		- bob@192.168.1.100: → 远端路径 `/home/bob`
		- 192.168.1.100 -> 默认和当前用户一致，本地当前是bob，那么从远端bob拉取/推送
			- web01 -> 可以在/etc/hosts中配置简短名称，提升效率
    
    示例：
		----------------------------------------------------
		#10.0.0.41 拉取 10.0.0.7 中的 /root/bob.txt 到本地当前目录
			[root@backup ~]# rsync -avz root@10.0.0.7:/root/bob.txt ./
			#这里使用172.16.1.7也行，只要局域网可以连通就行
			Are you sure you want to continue connecting (yes/no)? yes
			root@10.0.0.7 s password: 
			receiving incremental file list
			bob.txt                                                            
		----------------------------------------------------
		#将10.0.0.41本地当前目录的 bob1.txt 推送到 10.0.0.7 的 /root/下
			[root@backup ~]# rsync -avz ./bob1.txt root@10.0.0.7:/root/
			root@10.0.0.7 s password: 
			sending incremental file list
			bob1.txt
		----------------------------------------------------

3、守护进程模式（企业常用）
	a.安装服务
		yum -y install rsync
	b.配置服务
		配置文件：/etc/rsyncd.conf
		
		‐‐‐ 配置详解 
		[root@backup ~]# vim /etc/rsyncd.conf 
		uid = rsync                     # 运行进程的用户 
		gid = rsync                     # 运行进程的用户组 
		port = 873                      # 监听端口 
		fake super = yes                # 无需让rsync以root身份运行，允许接收文件的完整属性 
		use chroot = no                 # 禁锢推送的数据至某个目录, 不允许跳出该目录 
		max connections = 200           # 最大连接数 
		timeout = 600                   # 超时时间 
		ignore errors                   # 忽略错误信息 
		read only = false               # 对备份数据可读写 
		list = false                    # 不允许查看模块信息 
		auth users = rsync_backup       # 定义虚拟用户，作为连接认证用户 
		secrets file = /etc/rsync.passwd # 定义rsync服务用户连接认证密码文件路径 
		
		[backup]                # 定义模块信息 
		comment = commit        # 模块注释信息 
		path = /backup          # 定义接收备份数据目录
		
		#此时/backup的权限是rwxr-xr-x root root ,要修改成rwxr-xr-x rsync rsync
		
	c.根据配置文件创建必要数据
		（1）创建rsync虚拟用户
			[root@backup ~]# useradd -M -s /sbin/nologin rsync
		（2）创建密码文件
			[root@backup ~]# cat /etc/rsync.passwd
			rsync_backup:123456
		（3）修改密码权限为600
			[root@backup ~]# chmod 600 /etc/rsync.passwd
			[root@backup ~]# ll /etc/rsync.passwd
			-rw------- 1 root root 20 Jul  8 21:58 /etc/rsync.passwd
		（4）创建backup目录
			[root@backup ~]# mkdir /backup
			[root@backup ~]# ll -d /backup
			drwxr-xr-x 2 root root 6 Jul  8 22:01 /backup

	d.启动服务
	
	d.测试检查





查看三种模式：
	[root@backup ~]# man rsync
	
	SYNOPSIS
       Local:  rsync [OPTION...] SRC... [DEST]

       Access via remote shell:
         Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
         Push: rsync [OPTION...] SRC... [USER@]HOST:DEST

       Access via rsync daemon:
         Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
               rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
         Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
               rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

       Usages  with  just  one SRC arg and no DEST arg will list the source files instead of
       copying.

```