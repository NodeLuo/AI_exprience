
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
1、免密交互
	传输过程中需要对方服务器的用户密码：
		（1）将密码写入文件中
			[root@backup ~]# echo 123456 > /etc/rsync.pass
		（2）设置密码文件权限
			[root@backup ~]# chmod 600 /etc/rsync.pass
		（3）使用rsync参数指定密码文件的位置
			[root@backup ~]# rsync one.txt rsync_backup@172.16.1.41::backup --password-file=/etc/rsync.pass
```