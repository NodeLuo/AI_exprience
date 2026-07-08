
1、rsync传输模式

```bash
1、本地模式
	Local:  rsync [OPTION...] SRC... [DEST]
	本地     命令    选项参数    源文件 目标位置


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