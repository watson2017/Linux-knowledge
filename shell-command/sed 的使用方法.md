# sed 用法

>sed [参数] 'command' 输入文本  
```
常用选项：
-n : 使用安静模式。在一般 sed 的用法中，所有来自 STDIN的资料一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(者动作)才会被列出来
-e ∶直接在指令列模式上进行 sed 的动作编辑；
-f ∶直接将 sed 的动作写在一个档案内， -f filename 则可以执行 filename 内的sed 动作；
-r ∶sed 的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
   =注意== -i ∶直接修改读取的档案内容，而不是由萤幕输出。       

常用命令：
a   ∶新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
c   ∶取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d   ∶删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
i   ∶插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p   ∶列印，亦即将某个选择的资料印出。通常 p 会与参数 sed -n 一起运作～
s   ∶取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！
```

###  例子：
```
 sed '1a drink tea\nor coffee' ab   #第一行后增加多行，使用换行符\n
 [root@localhost ruby] # sed '1c Hi' ab                #第一行代替为Hi     
```

-----------------------------------
```
查看本机开放的所有端口：
[root@scott 桌面]# netstat -tulpan
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      1704/rpcbind        
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1991/sshd           
tcp        0      0 127.0.0.1:631               0.0.0.0:*                   LISTEN      1803/cupsd          
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      2077/master         
tcp        0      0 127.0.0.1:6010              0.0.0.0:*                   LISTEN      2383/sshd     
[root@scott 桌面]# netstat -tulpan | awk '{print $4}' | grep [0-9] | awk -F ':' '{print $NF}' | sort -nu   【grep [0-9]表示搜索包含数字的行】

```
###  删除包含某些字符的行
```
[ansible@test56-pgpool2-node1 ~]$ cat file 
aaa
bbb
ccc
ddd
[ansible@test56-pgpool2-node1 ~]$ sed -i '/aaa/d' file
[ansible@test56-pgpool2-node1 ~]$ cat file 
bbb
ccc
ddd
```
###  在每行的末尾添加指定的字符
```
[ansible@test56-pgpool2-node1 ~]$ cat file 
bbb
ccc
ddd
[ansible@test56-pgpool2-node1 ~]$ sed -i 's/$/;/g' file
[ansible@test56-pgpool2-node1 ~]$ cat file 
bbb;
ccc;
ddd;
```
###  每行行首添加字符
```
sed -i  's/^/xxxxx/g'  file
```

###  删除第ｎ行
   
    sed ‘Nd’ filename
