
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
```