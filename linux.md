###修改启动项
linux系统启动流程  

1. bios找到磁盘上的mbr主引导扇区
2. 进入grub界面选择相应的启动内核
3. 读取kernel内核文件-/boot/vmlinuz-*
4. 读取init的镜像文件-/boot/initrd-*
5. init去读取/etc/inittab
6. 读取启动级别(id:3:initdefault)
7. 读取/etc/rc.d/rc.sysinit, 完成时钟设置, 主机名的设置, 分区表的挂载(/etc/fstab)
8. 读取/etc/rc.d/rc脚本, 通过该脚本吸收3级别, 然后启动/etc/rc.d/rc3.d下所有以s开头的服务, 不启动该目录下以k开头的服务
9. 进入登录界面

用rpm包安装的程序, 都可以用service统一管理, 传入start/restart/stop作为参数  
rpm包会为程序生成一个启动脚本放到/etc/rc.d/init.d中, 供service管理

用chkconfig命令可以修改开机启动  
可以通过-h参数查看帮助  
如chkconfig --level 3 httpd on  
重启后生效  
只有rpm安装的服务脚本, 才能通过chkconfig调整启动级别

不是系统服务，需要命令启动的  
直接编辑 /etc/rc.d/rc.local  
在最下面一行加入启动命令即可。例如：  
```shell
su - svn -c "svnserve -d --listen-port 9999 -r /opt/svndata" //这样开机就可以自动启动svnserver了。  
```  
1、把启动程序的命令添加到/etc/rc.d/rc.local文件中，比如下面的是设置开机启动httpd。  
 代码如下	复制代码  
```shell
#!/bin/sh 
# 
# This script will be executed *after* all the other init scripts. 
# You can put your own initialization stuff in here if you don't 
# want to do the full Sys V style init stuff. 
  
touch /var/lock/subsys/local
/usr/local/apache/bin/apachectl start
```
2、把写好的启动脚本添加到目录/etc/rc.d/init.d/，然后使用命令chkconfig设置开机启动。
例如：我们把httpd的脚本写好后放进/etc/rc.d/init.d/目录，使用
```shell
chkconfig --add httpd 
chkconfig httpd on
```
命令即设置好了开机启动。
