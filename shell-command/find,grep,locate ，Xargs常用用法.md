# 一、find的常用用法
> find < path > < expression > < cmd >

>`path`： 所要搜索的目录及其所有子目录。默认为当前目录。
`expression`： 所要搜索的文件的特征。
`cmd`： 对搜索结果进行特定的处理。

---------------
**参数**：
> - `-perm` 按照文件权限来查找文件
> - `-mtime -n/+n  `  按照文件的更改时间来查找文件， -n表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前。
> - `-newer file1 ! file2 `查找更改时间比文件file1新但比文件file2旧的文件
> - `-type `查找某一类型的文件，诸如：
> `b` - 块设备文件。
> `d` - 目录。
> `c` - 字符设备文件。
> `p ` - 管道文件。
> `l` - 符号链接文件。
> `f` - 普通文件
> - `-exec`，find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为’command’ `{}`   `\;`  注意`{}`和`\;`之间的空格
> - `-ok`，和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。
example:
```
find . -perm 755 –print 在当前目录下查找文件权限位为755的文件
find / -mtime -5 –print 在系统根目录下查找更改时间在5日以内的文件
find /var/adm -mtime +3 –print 在/var/adm目录下查找更改时间在3日以前的文件
find /etc -type d –print 在/etc目录下查找所有的目录  [type表示查找什么样的文件类型]
find ./ -size 0 -exec rm {} \;   = rm -i `find ./ -size 0`   = find ./ -size 0 | xargs rm -f &  删除文件大小为零的文件 
```
### find 与xargs的结合 常用于find进行查找文件，之后xargs对其进行处理
xargs的使用是由于很多命令不支持|管道来传递参数，才因此使用这个命令
如下方的例子，xargs -i 表示的是将前面find找到的结果一行一行赋值给{}，-t表示的是先打印命令，然后再执行。
xargs的其他参数具体使用方法参见：[xargs的用法](http://blog.csdn.net/zhangfn2011/article/details/6776925)
 ```
[root@master01 test]# touch file{1..3}.log
[root@master01 test]# ls
file1.log  file2.log  file3.log
[root@master01 test]# find . -type f -exec ls {} \;
./file1.log
./file2.log
./file3.log
# 复制并修改符合条件的文件
[root@master01 test]# find . -type f -exec cp {} {}.bak  \;
[root@master01 test]# ls
file1.log  file1.log.bak  file2.log  file2.log.bak  file3.log  file3.log.bak
# rename将符合条件的文件改名
[root@master01 test]# find . -type f -exec rename .log.bak    .txt   {}  \;
[root@master01 test]# ls
file1.log  file1.txt  file2.log  file2.txt  file3.log  file3.txt
------------
# 找到符合条件的文件并复制到远程服务器上
[root@zz-master01 test]# find . -type f -name "*.txt"| xargs -i -t scp  {} zz-master01:/root
scp ./file1.txt zz-master01:/root 
root@zz-master01's password: 
file1.txt                                                                                                                                100%    0     0.0KB/s   00:00    
scp ./file2.txt zz-master01:/root 
root@zz-master01's password: 
file2.txt                                                                                                                                100%    0     0.0KB/s   00:00    
scp ./file3.txt zz-master01:/root 
root@zz-master01's password: 
file3.txt                                                                                                                                100%    0     0.0KB/s   00:00 

```


***值得注意的是 find 在寻找数据的时候相当的耗硬盘，所以没事情不要使用 find 啦！有更棒的指令可以取代呦，那就是下方的`whereis` 与 `locate`***

### 二、locate命令
>`locate`命令其实是“`find -name`”的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库
```
locate /etc/sh
搜索etc目录下所有以sh开头的文件。 【locate后面直接接一个路径，locate /etc/*.log】
$ locate -i ~/m
搜索用户主目录下，所有以m开头的文件，并且忽略大小写。【-i表示忽略大小写】
```
### 三、whereis命令
>`whereis`命令只能用于程序名的搜索，而且只搜索二进制文件（参数 `-b`）、man说明文件（参数`-m`）和源代码文件（参数`-s`）。如果省略参数，则返回所有信息
whereis grep  [如不加任何参数就会搜索所有相关的文件，如grep 的man说明文件，源码包]

example：
```
[root@data ~]# whereis grep 
grep: /bin/grep /usr/share/man/man1p/grep.1p.gz /usr/share/man/man1/grep.1.gz
[root@data ~]# whereis -b grep 
grep: /bin/grep
```
### 四、which命令

>`which`命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

example:
```
[root@data ~]# which  grep    【可以看出grep这个命令是在/bin下】
/bin/grep
```
### 五、grep命令

>grep 命令参数：
> - -i  忽略大小写                                                                                                                                           
> - -A  前面
> - -B  后面
> - -C  上下
> - -l  只显示搜索的文件名
> - -n  给每行数据添加行数
> - -R  递归
> - -w  单词边界
> - -E  正则表达式中不需要扩展
> - -o  输出匹配的那部分，而不是整行
> - -o|wc -l 显示文件中某个单词出现的次数
> - -c 匹配的行数
> - -v 反向
> - -q 不输出但返回值

example:
```
- 在/root/桌面下搜索所有含有root 的文件:
[root@scott 桌面]# grep -l "root" -R  /root/桌面   
/root/桌面/passwd

- 打印出/etc/passwd文件中包含gdm的行,并打出其下面2行:
[root@scott ~]# grep -A2 -n gdm /etc/passwd
26:gdm:x:42:42::/var/lib/gdm:/sbin/nologin
27-ntp:x:38:38::/etc/ntp:/sbin/nologin
28-apache:x:48:48:Apache:/var/www:/sbin/nologin 
------如果是打印出gdm的上面两行用B2，如果是上面两行以及下面两行用C2 --------------
```
- [find详细使用说明链接](http://blog.csdn.net/wzzfeitian/article/details/40985549)
- [find 查找文件条件筛选](http://www.jb51.net/article/99319.htm)
