---
layout: post
title: Linux指令拾遗01——find
---

因为日常接触linux比较多，为了提升文件浏览效率，有必要学习一下find指令的使用

## 语法

find [路径] [匹配条件] [动作]

**参数说明 :**

**路径** 是要查找的目录路径，可以是一个目录或文件名，也可以是多个路径，多个路径之间用空格分隔，如果未指定路径，则默认为当前目录。

expression 是可选参数，用于指定查找的条件，可以是文件名、文件类型、文件大小等等。

**匹配条件** 中可使用的选项有二三十个之多，以下列出最常用的部份：

`-name` pattern：按文件名查找，支持使用通配符 * 和 ?。
`-type` type：按文件类型查找，可以是 f（普通文件）、d（目录）、l（符号链接）等。
`-size` [+-]size[cwbkMG]：按文件大小查找，支持使用 + 或 - 表示大于或小于指定大小，单位可以是 c（字节）、w（字数）、b（块数）、k（KB）、M（MB）或 G（GB）。
`-mtime` days：按修改时间查找，支持使用 + 或 - 表示在指定天数前或后，days 是一个整数表示天数。
`-user` username：按文件所有者查找。
`-group` groupname：按文件所属组查找。

**动作** 是可选的，用于对匹配到的文件执行操作，比如删除、复制等。

find 命令中用于时间的参数如下：

`-amin` n：查找在 n 分钟内被访问过的文件。
`-atime` n：查找在 n*24 小时内被访问过的文件。
`-cmin` n：查找在 n 分钟内状态发生变化的文件（例如权限）。
`-ctime` n：查找在 n*24 小时内状态发生变化的文件（例如权限）。
`-mmin` n：查找在 n 分钟内被修改过的文件。
`-mtime` n：查找在 n*24 小时内被修改过的文件。
在这些参数中，n 可以是一个正数、负数或零。正数表示在指定的时间内修改或访问过的文件，负数表示在指定的时间之前修改或访问过的文件，零表示在当前时间点上修改或访问过的文件。

正数应该表示时间之前，负数表示时间之内。

例如：-mtime 0 表示查找今天修改过的文件，-mtime -7 表示查找一周以前修改过的文件。

关于时间 n 参数的说明：

+n：查找比 n 天前更早的文件或目录。

-n：查找在 n 天内更改过属性的文件或目录。

n：查找在 n 天前（指定那一天）更改过属性的文件或目录。

## 示例

查找当前目录下名为 file.txt 的文件：
`find . -name file.txt`

将当前目录及其子目录下所有文件后缀为 .c 的文件列出来:
`find . -name "*.c"`

将当前目录及其子目录中的所有文件列出：
`find . -type f`

查找 /home 目录下大于 1MB 的文件：
`find /home -size +1M`

查找 /var/log 目录下在 7 天前修改过的文件：
`find /var/log -mtime +7`

查找过去 7 天内被访问的文件:
`find /path/to/search -atime -7`

在当前目录下查找最近 20 天内状态发生改变的文件和目录:
`find . -ctime  20`

将当前目录及其子目录下所有 20 天前及更早更新过的文件列出:
`find . -ctime  +20`

查找 /var/log 目录中更改时间在 7 日以前的普通文件，并在删除之前询问它们：
`find /var/log -type f -mtime +7 -ok rm {} \;`

查找当前目录中文件属主具有读、写权限，并且文件所属组的用户和其他用户具有读权限的文件：
`find . -type f -perm 644 -exec ls -l {} \;`

查找系统中所有文件长度为 0 的普通文件，并列出它们的完整路径：
`find / -type f -size 0 -exec ls -l {} \;`

找并执行操作（例如删除）：
`find /path/to/search -name "pattern" -exec rm {} \;`
这个例子中，-exec 选项允许你执行一个命令，{} 将会被匹配到的文件名替代，\; 表示命令结束。

## Reference

https://www.runoob.com/linux/linux-comm-find.html
