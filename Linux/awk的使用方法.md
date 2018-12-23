# awk 用法：
>常用格式：  awk    [options  参数]    'commands'        testfiles

options选项：
```
① -F    定义字段分隔符，默认分隔符为连续空格或制表符。用$1,$2,$3等顺序表示files中每行以间隔符号分隔的各列不同域
② -v    定义变量并赋值，也可以借用其他方式从shell变量中引入变量

FS 目前的分隔符，默认是空格键
NF 每一行（$0）拥有字段的总数
NR 目前awk所处理的是“第几行”数据 
$NF 每一行的最后一列
例子：
  awk -F":/" '{print $1,$7}'   testfile     以冒号加斜杠 “：/”整体作为分隔符
  awk -F"[:/]" '{print $1,$7}' testfile     以冒号或斜杠 / 作为分隔符，打印第1第7个字段
  awk -v num=$num -v num1=$num1 'NR==num,NR==num+num1{print}' a 
```

###  例子1:
列出与密码文件的前五行，并展示出行数、每行有多少列（以0开始计数）
```
[root@scott 桌面]# cat /etc/passwd  | head -n 5 | awk -F ':' '{print $1 "\t lines:" NR "\t" "column:" NF}'
root	     lines:1	column:7
bin	         lines:2	column:7
daemon	     lines:3	column:7
adm	         lines:4	column:7
lp	         lines:5	column:7
```
--------------------------------------------------
###  找出uid 小于10（第三列） 并且列出账号和uid 
```
[root@scott 桌面]# cat passwd  | awk -F ':'  '$3 <= 10 {print $1 "\t" $3 }' 
root	0
bin	1
daemon	2
adm	3
lp	4
sync	5
shutdown	6
halt	7
mail	8
uucp	10
```
###  统计文件行数
```
[root@scott ~]# awk 'END{print NR}' file 
40
```


###  通过ipconfig显示出IP ：
 ```
ifconfig |awk 'NR==2{split($2,a,":");print a[2]}'
```
###  通过ip a 显示出IP：
```
ip a | awk 'NR==8 {print $2}' | awk -F '[/]' '{print $1}' 
````
###  在passwd文件最前和最后一行添加** ：
```
 awk ' BEGIN {print "********"} {print  $0} END {print "********"}' passwd
```


###  统计tcp各状态的连接数
```
[root@zabbix-test121 wz]#  netstat -ant|grep [0-9] |awk ' {S[$NF]++}END{for(k in S) print S[k],k}'
18 TIME_WAIT
4 ESTABLISHED
11 LISTEN
```
### 参考文章：
- http://www.cnblogs.com/dong008259/archive/2011/12/06/2277287.html
- http://zhidao.baidu.com/link?url=7rxAB6X24BmYnNcWe3de_LR7IkAOC5DPaOZrGY7FlXHzgt49i68bepiraohjz4mNSMIz2eDfaDO6ujZgyzYLKeNLmxEpUZFbBOsezFv8OrW
- http://www.linuxidc.com/Linux/2013-11/92787.htm
