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
