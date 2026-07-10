
1、虚拟用户rsync的UID为1000的原因

```bash
默认：
	管理员 0
	虚拟用户 1-999
	普通用户 1000+
	
	（1）手动创建可指定uid
		a.虚拟用户可指定1000+
		b.普通用户可指定1-999
	（2）若不指定uid,创建的所有用户默认按顺序排，若当前用户uid 1001已存在，默认创建的虚拟用户uid 1002
```

2、rsync 参数选项

```bash
‐a #归档模式传输, 等于‐tropgDl 
‐v #详细模式输出, 打印速率, 文件数量等 
‐z #传输时进行压缩以提高效率 
‐r #递归传输目录及子目录，即目录下得所有目录都同样传输。 
‐t #保持文件时间信息 
‐o #保持文件属主信息 
‐p #保持文件权限 
‐g #保持文件属组信息 
‐l #保留软连接 
‐P #显示同步的过程及传输时的进度等信息 
‐D #保持设备文件信息 
‐L #保留软连接指向的目标文件 
‐e #使用的信道协议,指定替代rsh的shell程序 
‐‐exclude=PATTERN #指定排除不需要传输的文件模式 
‐‐exclude‐from=file #文件名所在的目录文件 
‐‐bwlimit=100 #限速传输 
‐‐partial #断点续传 
‐‐delete #让目标目录和源目录数据保持一致 
‐‐password‐file=xxx #使用密码文件
```

3、案例

```bash
1、免密交互
	传输过程中需要对方服务器的用户密码：
	方法一：
		（1）将密码写入文件中
			[root@web01 ~]# echo 123456 > /etc/rsync.pass
		（2）设置密码文件权限
			[root@web01 ~]# chmod 600 /etc/rsync.pass
		（3）使用rsync参数指定密码文件的位置
			[root@web01 ~]# rsync -avz one.txt rsync_backup@172.16.1.41::backup --password-file=/etc/rsync.pass
	
	方法二：
		rsync的内置变量:RSYNC_PASSWORD（默认为空）
		在执行rsync推送的过程中，先查找RSYNC_PASSWORD中是否存在密码，若没有则提示输入密码，有则直接使用该变量中的密码
		（1）临时设置环境变量，重启失效
			[root@backup ~]# export RSYNC_PASSWORD=123456
		（2）传输文件
			[root@web01 ~]# rsync -avz two.txt rsync_backup@172.16.1.41::backup

2、无差异同步
	参数：--delete
	（1）若是推送，以推送方文件为准保持一致，多出的自动删除
		10.0.0.41：dir/a.txt
		10.0.0.7: dir/a.txt dir/b.txt
		
		[root@backup ~]# rsync -avz --delete dir/ 10.0.0.7:dir
		root@10.0.0.7 s password: 
		sending incremental file list
		deleting b.txt

	（2）若是拉取，以被拉取方文件为准保持一致，多出的自动删除
		10.0.0.41: dir/a.txt dir/b.txt
		10.0.0.7:dir/a.txt
		
		[root@backup ~]# rsync -avz --delete 10.0.0.7:dir/ dir
		root@10.0.0.7 s password: 
		receiving incremental file list
		deleting b.txt
		./
	
	（3）企业案例：服务器代码中毒是用--delete进行无差异同步

3.传输限速
	参数：--bwlimit=1M
	原因：防止占用带宽影响用户体验(P 显示进度)
	（1）推送文件
		[root@web01 ~]rsync -avzP --bwlimit=1M 1.mp4 rsync_backup@172.16.1.41::backup

```

脚本存放位置
```bash
公司一般会将脚本存放在固定的位置
寻找方法：
	（1）find 查找
		find / -type f -name "*.sh"
	（2）history 查找
		history | grep -C10 sh       #查找关于sh的上下各10行
	（3）定时任务查找
		cat /etc/crontab             #寻找有关sh的路径
```

4、客户端需求

```bash
需求：
1.客户端提前准备存放的备份的目录，目录规则如下: /backup/nfs_172.16.1.31_2018‐09‐02 
2.客户端在本地打包备份(系统配置文件、应用配置等)拷贝至 /backup/nfs_172.16.1.31_2018‐09‐02 3.客户端最后将备份的数据进行推送至备份服务器 
4.客户端每天凌晨1点定时执行该脚本 
5.客户端服务器本地保留最近7天的数据, 避免浪费磁盘空间
```

```bash
1、客户端提前准备存放备份的目录，目录命名规则如下：主机名_ip地址_时间
	示例：/backup/nfs_172.16.1.31_2018-09-01
	（1）通用性指令
		[root@web01 ~]# mkdir -p /backup/`hostname`_`hostname -I|awk '{print $1}'`_`date +%F-%H-%M`
		
		注意：此时时间会变化不固定，可以使用变量将时间固定，永久写到/etc/profile中
		a.变量赋值
			[root@web01 ~]# DATE=`date +%F-%H-%M`
		b.输出变量
			[root@web01 ~]# echo $DATE
			2026-07-10-08-47
	
	（2）变量创建
		[root@web01 ~]# dir=`hostname`_`hostname -I|awk '{print $1}'`_`date +%F-%H-%M`
		[root@web01 ~]# echo $dir
		web01_10.0.0.7_2026-07-10-08-59
		[root@web01 ~]# mkdir -p /backup/$dir
		[root@web01 ~]# ll /backup
		total 0
		drwxr-xr-x 2 root root 6 Jul 10 08:59 web01_10.0.0.7_2026-07-10-08-59

2、客户端在本地打包备份放入创建好的目录中
	（1）创建目录
		Date=`date +%F-%H-%M`
		dir=`hostname`_`hostname -I|awk '{print $1}'`_$Date
		mkdir -p /backup/$dir
	（2）打包存入
		tar zcf /backup/$dir/etc_$Date.tar.gz /etc/hosts /etc/passwd &>/dev/null

3、客户端最后将备份的数据进行推送至备份服务器
	（1）赋值环境变量
		export RSYNC_PASSWORD=123456
	（2）推送文件
		rsync -az /backup/$dir rsync_backup@172.16.1.41::backup

4、客户端服务器本地保留最近7天的数据, 避免浪费磁盘空间
	find /backup/ -mtime +7|xargs rm -rf


```

```bash
脚本执行客户端需求：

[root@web01 ~]# mkdir -p /server/scripts
[root@web01 ~]# cd /server/scripts

[root@web01 ~]# vim test.sh

sh -x test.sh #可查看test.sh的执行流程

test.sh 脚本：


```
