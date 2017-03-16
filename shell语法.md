### 获取帮助
```shell
<命令名> --help
<命令名> -h
man <命令名>
```
man里面可以看例子

### 数组语法
1.数组定义
typeset和declare完全一样
```shell
typeset -a abc
```
2.数组赋值
```shell
abc[0]=1
abc[1]=5
abc[2]=8
```
3.获取数组的每一个元素, @和*一样
```shell
echo ${abc[@]}
echo ${abc[*]}
```
4.获取数组的每一个下标, 下表可以不连续
```shell
echo ${!abc[@]}
echo ${!abc[*]}
```
5.获取下标的长度
```shell
echo ${#abc[@]}
echo ${#abc[*]}
```

### 防火墙
#### iptables
存储防火墙策略
```shell
service iptables save
```
存储在/etc/sysconfig/iptables中
####selinux
1.查看
```shell
sestatus
```
2.关闭
```shell
vi /etc/selinux/config
SELINUX=disabled
```
3.开启
```shell
vi /etc/selinux/config
SELINUX=enforce
```
4.修改需要重启


### VIM和shell的切换
在vim中, 按ctrl+z挂起, 再用fg命令回到vim

### 截取字符串
```shell
file=/dir1/dir2/dir3/my.file.txt

${file#*/} #删掉第一个"/"及其左边的字符串, 输出为dir1/dir2/dir3/my.file.txt
${file##*/} #删掉最后一个"/"及其左边的字符串, 输出为my.file.txt
${file#*.} #删掉第一个"."及其左边的字符串, 输出为file.txt
${file##*.} #删掉最后一个"."及其左边的字符串, 输出为txt

${file%%/*} #删掉第一个"/"及其右边的字符串, 输出为空值
${file%/*} #删掉最后一个"/"及其右边的字符串, 输出为/dir1/dir2/dir3
${file%%.*} #删掉第一个"."及其右边的字符串, 输出为/dir1/dir2/dir3/my.file
${file%.*} #删掉最后一个"."及其右边的字符串, 输出为/dir1/dir2/dir3/my
```


### awk
```shell
cat file | awk '{print $1}' #每行打印第一个字段
cat file | awk '{print $1,$2}' #每行打印第一个字段和第二个字段
cat file | awk '{print $1"+"$2}' #每行打印第一个字段+第二个字段, 双引号内可以打印任何字符串
cat file | awk -F'[: ]' '{print $2}' #每行以冒号和空格为分隔符, 打印第二列
cat file | awk -F'[: ]+' '{print $2}' #每行以连续的冒号和空格为分隔符, 打印第二列
ifconfig eth0 | grep 't a' | awk -F'[: ]+' '{print $4}' #打印eth0的ip

cat file | awk '{print NF}' #打印每一行的字段数, NF是awk的内置变量
cat file | awk '{print $NF}' #打印每一行的最后字段
cat file | awk '{print $(NF-1)}' #打印每一行的倒数第二个字段, 不加括号会先拿到最后字段在每个值上减1

awk '{print NR}' file file1 #打印每行行号, 第二个文件行号累加
awk '{print NR}' file file1 #打印每行行号, 第二个文件行号重置
awk 'NR==FNR{print $0}' file file1 #打印前一个文件的所有字段
awk 'NR!=FNR{print $0}' file file1 #打印后一个文件的所有字段
cat file | awk 'NR>1&&NR<4{print $0}' #打印第2行和第3行
cat file | awk '{print NR}' | tail -1 #打印文件行数
 
cat file | awk 'BEGIN{print "================title============"}{print $0}END{print "=============end=========="}' #可以在原有文件加头加尾

cat file | awk '{print $0, $3+$4+$5,($3+$4+$5)/3,int(($3+$4+$5)/3)}' #多加一些运算结果列, 除法支持小数, 加上int可取整
cat file | awk '{a=$3+$4+$5;print $0,a}' #可以在同一个大括号内使用变量
cat file | awk '{a+=$1}END{print a}' #可以得到第一个字段的和

cat file | awk '{if($3>80)print $0}' #第三个字段大于80时输出整行
cat file | awk '$3>80{print $0}' #与上一条语句等价

ls -l | awk '/^-/{if($5>1024)print $0}' #花括号前为正则表达式,匹配以-开头的行
```

1. 默认以空白(一个或多个空格)为分隔符, 或者用-F参数指定分隔符
2. 读取一行, 然后执行awk后面的命令
3. NF, 字段数, number of fields
4. NR, 行号, number of record
5. FNR, 相对行号, 单文件awk中效果和NR一样, 多文件时不同, 见用例
6. 语句结构: awk '[设定1]{动作2}[设定2]{动作2}...', 设定包括NR,BEGIN,END,$3>80,正则表达式等等
7. BEGIN和END分别对应读取所有行之前和所有行之后要进行的动作, BEGIN最常用的地方是变量赋值, END最常用的地方是变量输出
8. awk是弱类型的语句,变量赋值前为空,但做数学运算时默认为0
9. 条件也可以写在大括号里面
