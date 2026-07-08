
1、rsync传输模式

```bash
1、本地模式
	Local:  rsync [OPTION...] SRC... [DEST]
	本地     命令    选项参数    源文件 目标位置
	
	示例：
		#将a.txt拷贝到dir下
		[root@backup ~]# rsync -avz a.txt dir
		sending incremental file list
		a.txt
		
		sent 123 bytes  received 35 bytes  316.00 bytes/sec
		total size is 35  speedup is 0.22
		
		#再次将a.txt拷贝到dir下，此时默认为增量拷贝
			此时进行的判断：
				a.文件没变 -> 跳过
				b.文件变了 -> 重新传整个文件



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