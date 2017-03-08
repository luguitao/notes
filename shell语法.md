###获取帮助
```shell
<命令名> --help
<命令名> -h
man <命令名>
```
man里面可以看例子

###数组语法
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

###防火墙
####iptables
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


###VIM和shell的切换
在vim中, 按ctrl+z挂起, 再用fg命令回到vim

###截取字符串
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


###awk
```shell
cat file | awk '{print $1}' #每行打印第一个字段
cat file | awk '{print $1,$2}' #每行打印第一个字段和第二个字段
cat file | awk '{print $1"+"$2}' #每行打印第一个字段+第二个字段, 双引号内可以打印任何字符串
cat file | awk -F'[: ]' '{print $2}' #每行以冒号和空格为分隔符, 打印第二列
cat file | awk -F'[: ]+' '{print $2}' #每行以连续的冒号和空格为分隔符, 打印第二列
ifconfig eth0 | grep 't a' | awk -F'[: ]+' '{print $4}' #打印eth0的ip
cat file | awk '{print NF}' #打印每一行的字段数, NF是awk的内置变量
cat file | awk '{print $NF}' #打印每一行的
```
1.默认以空白(一个或多个空格)为分隔符, 或者用-F参数指定分隔符
2.读取一行, 然后执行awk后面的命令
