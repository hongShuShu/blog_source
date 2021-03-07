---
title: 常用shell命令——grep、find、xargs、sed、awk
---
原文写作时间: 20180719
grep、find、xargs、sed、awk命令的基础用法。

### grep
命令格式：grep -options pattern file
作用：配合正则使用，根据文件内容过滤或搜索特定字符。

主要参数
-c:只输出匹配行的计数
-i:不区分大小写
-h:查询多文件时不显示文件名
-l:查询多文件时只输出包含匹配字符的文件名
-n:显示匹配行及行号
-s:不显示不存在或无匹配文本的错误信息
-v:显示不包含匹配文本的所有行
–color=auto:可以将找到的关键词部分加上颜色显示
-r:递归找子目录
-d:忽略子目录
\< 和 > 标记单词的开始与结尾
^:表示匹配以表达式开头
$:表示匹配以表达式结尾

例子：
```
grep 'start' 1.9_time_take.sh
作用：显示此文件中包含start的行

grep '[a-z]\{5\}' 1.9_time_take.sh
作用：显示此文件中包含每个字符串有5个联系小写字符的行

grep '^end' 1.9_time_take.sh
作用：匹配本文件以end开头的行

grep -c '#*' 1.8_password.sh 1.9_time_take.sh
作用：统计当前目录下这两个文件中包含#的行数
```
如果输出很多，可以通过管道符将内容转到‘less’上阅读。

```
grep g test.txt | less
```

### find
根据文件名过滤
命令格式：
find pathname -options [-print -exec -ok …]
命令->查找路径-参数->匹配表达式->要查询的文件.

主要参数
-exec: 将匹配到的文件执行后面的shell命令
命令选项
-name: 按照文件名查找文件
-perm: 按照文件权限来查找文件
-prune: 不在当前指定的目录中查找
-type: 查找某一类型的文件
```
b - 块设备文件
d - 目录
c - 字符设备文件
p - 管道文件
l - 符号链接文件
f - 普通文件
```
-size n: 查找文件长度为n的文件
-depth: 先查找当前目录中的文件，再在子目录中查询

例子：
<font color=red size=3>{} \; 是-exec的固定写法；和-ok等价，ok会向用户确认操作</font>

```
find . -type f -exec ls -l {} \;
作用：找到当前目录下的普通文件，查看文件属性

find . -type d | sort
作用：查找当前目录中所有子目录并排序

find . -name "*.sh" -print
作用：找到当前目录下后缀是sh的文件并输出
```

### xargs
上面说到了 -exec,是执行后面的shell，但是他有一个缺点就是会等待前面的命令执行完成之后再执行。
假如要查的文件很大，就会把缓冲区撑爆。那有人就说了，能不能找到了一部分文件就去执行后续操作，别等全部查询完再执行不就好了吗？是的，xargs就是解决这个问题的，前面边查询，后面边执行。

### sed
作用：一行一行读取待处理文件，把待处理文件的内容和处理结果一起输出到标注输出。

常用命令：
/pattern/p 打印匹配pattern的行
/pattern/d 删除匹配pattern的行
/pattern/s/pattern1/pattern2/ 查找符合pattern的行，将第一个匹配pattern1的字符替换成pattern2
/pattern/s/pattern1/pattern2/g 查找符合pattern的行，将该行所有匹配pattern1的字符替换成pattern2

-n: 静默输出，’/pattern/p’默认会把匹配行重复一遍和源文件一起输出，加上此参数会只输出匹配项
-f: 从文件中读取脚本
-i: 处理后的内容会修改源文件！！！
-r: 使用扩展正则表达式

a,append 追加
i,insert 插入
d,delete 删除
s,substitution 替换

例子:
```
sed '/taken/p' 1.9_time_take.sh
作用：查找当前文件的包含taken的行

sed -e '/qwe/d' test.txt
作用：删除文件中包含qwe的行，但不改变源文件，结果在终端界面显示

sed '2,3d' test.txt
作用：删除文件第2行到第3行，但不改变源文件
```
```
sed -i '' -e '1i \
Hello World.' test.txt
```
作用：在第一行插入Hello Word
插入语句在测试的时候一直显示失败，经过搜索，发现是我环境的原因。
这条命令在Mac OS X下运行会失败，除非你已经使用GNU的sed替换了系统的内置sed，FreeBSD下也一样，经过反复测试，得出BSD版本的sed需要使用上面的办法。
即在1i后加入空格在加入一个反斜杠 \ ，回车后 另起一行，在下一行输入要插入的内容，以单引号结束，后面跟文件名,可以正常运行。
这样的命令在GNU版本下的sed同样可以正常运行。
原文链接在这里:[点我查看](https://my.oschina.net/jsk/blog/166974)

### awk
基本形式：awk option ‘script’ file1 file2 。。。
命令格式：/pattern/{actions}
condition{actions}
作用：列处理工具，用法和sed类似，pattern是正则表达式，actions是操作。
例子：
```
awk '{print $2;}' test.txt
作用：打印文件第二列数据

ps aux > file
awk '{print $1 "\t" $2;}' file
作用：查询当前进程及进程id，并格式化输出

awk '/^ *$/ {x=x+1;} END {print x;}' test.txt
作用：统计文件的空行数，END  是文件查询完毕之后执行。

awk 'BEGIN {FS=":"} {print $2}' test.txt
作用：以冒号分割行，并打印第2列。BEGIN是先做什么事，在此例子中是先把FS命令变成冒号。
```

补充：awk的内建变量
```
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
FILENAME           awk浏览的文件名
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行 -F选项
NF                 浏览记录的域的个数
NR                 已读的记录数
OFS                输出域分隔符
ORS                输出记录分隔符
```



