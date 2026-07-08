
1、rsync传输模式

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

2、远程模式
    Access via remote shell:
         Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
         拉取   命令    参数选项     用户  主机  源文件  目标位置
         HOST:主机IP地址 域名 主机名
         Push: rsync [OPTION...] SRC... [USER@]HOST:DEST





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