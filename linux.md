###修改启动项
linux系统启动流程<br>
1. bios找到磁盘上的mbr主引导扇区
2. 进入grub界面选择相应的启动内核
3. 读取kernel内核文件-/boot/vmlinuz-*
4. 读取init的镜像文件-/boot/initrd-*
5. init去读取/etc/inittab
6. 读取启动级别(id:3:initdefault)
7. 读取/etc/rc.d/rc.sysinit, 完成时钟设置, 主机名的设置, 分区表的挂载(/etc/fstab)
8. 读取/etc/rc.d/rc脚本, 通过该脚本吸收3级别, 然后启动/etc/rc.d/rc3.d下所有以s开头的服务, 不启动该目录下以k开头的服务
9. 进入登录界面
