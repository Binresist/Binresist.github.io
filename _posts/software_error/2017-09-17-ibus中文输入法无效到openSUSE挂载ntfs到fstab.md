---
layout: post
title: ibus中文输入法无效到openSUSE挂载ntfs到fstab
category: 软件出错解决
keywords: ibus, 输入法, 中文, openSUSE, ntfs, fstab
---

## 事情起因
工作原因，经常需要在windows和linux之间切换，贫穷原因，只有一个笔记本，因此给自己装了openSUSE+Windows 7双系统，先装的Windows 7，然后装的openSUSE，二者共用一块ntfs格式的磁盘作为数据盘，重启之后，Windows 7启动运行正常，openSUSE的中文输入法有问题，总是无法切换。
经过漫长的分析和尝试，排除了输入法设置问题，安装不完整问题，最终在一次偶然尝试ibus-resta先装的Windows 7，然后装的openSUSE，二者共用一块ntfs格式的磁盘作为数据盘，重启之后，Windows 7启动运行正常，openSUSE的中文输入法有问题，总是无法切换。
经过漫长的分析和尝试，排除了输入法设置问题，安装不完整问题，最终在一次偶然尝试`ibus-restart`时，发现使用xxx安装的ibus，自身居然没有权限，后来检查权限才发现，整个磁盘所属为user,属主为users，包括xxx的home文件夹在内！而且使用root用户修改文件夹权限，提示成功，检查时发现权限居然没有发生变化。

## 解决流程
ntfs数据盘挂载，是在openSUSE安装时使用其默认挂载方式挂载的，于是想到检查/etc/fstab，具体设置如下：
```shell
UUID=0200584300583FB9                           /home                ntfs-3g    user,gid=users,fmask=133,dmask=022,locale=zh_CN.UTF-8         0 0
```
果然，该用户属主为user,属组为users。

### 尝试1
将上面的属组属主均修改为root，然后利用root用户权限修改用户文件夹权限，从而使xxx用户的输入法ibus可以正常使用。
```shell
UUID=0200584300583FB9                           /home                ntfs-3g    root,gid=root,fmask=133,dmask=022,locale=zh_CN.UTF-8         0 0
```
修改完毕，重启设置文件夹权限，发现依然是提示成功，而实际权限未修改成功，于是怀疑是fmask和dmask的设置有问题，导致无法正确赋予用户权限，考虑第二次修改。

### 尝试2
将fmask和dmask均修改为000，期望权限设置正常，然而依然失败。
```shell
UUID=0200584300583FB9                           /home                ntfs-3g    root,gid=root,fmask=000,dmask=000,locale=zh_CN.UTF-8         0 0
```
不过两次的尝试，发现挂载时文件属主和属组等的设置的确是能够正常设置的，那么，如果将磁盘挂载时的属主设置为xxx，是不是就能正常工作了？于是继续进行第三次尝试。
注：fmask和dmask配合使用，决定新创建文件和新创建目录的权限，比如fmask=133，决定了默认文件权限是644，即rw--w--w-，具体权限相关请百度

### 尝试3
将属主设置为xxx，属组设置为users，为了防止之前操作的影响，用户是删掉重新创建的。
```shell
userdel -r xxx
useradd xxx -d /home/xxx
```
设置成功后在执行`id xxx`或者在/etc/passwd中，均能看到该用户的uid，将上面的属主和属组进行相应的修改即可。
```shell
binresist@linux-4o87:/home> id binresist 
uid=1000(binresist) gid=100(users) 组=100(users)
```
修改后如下，重启，安装ibus(为减少不必要的麻烦，建议先将原来安装的ibus卸掉)，果然可以正常运行，权限ok。
UUID=0200584300583FB9                           /home                ntfs-3g    uid=1000,gid=100,fmask=133,dmask=022,locale=zh_CN.UTF-8         0 0

## 进一步了解/etc/fstab
修改了半天的/etc/fstab，才发现自己对其了解甚少，于是百度简单了解了下，这里做一个简单的记录。
> /etc/fstab在系统开机时会被读取，然后按照该配置文件中的设置，进行相应的挂载。
> /目录要在靠前的位置挂载，因为其他目录均处于/目录下。
> fstab总计有六列，每列的具体意义如下：
	>> 第一列 磁盘设备的UUID或者Label，可以使用`ls -l /dev/disk/by-uuid/`和`ls -l /dev/disk/by-label/`或`blkid`等命令进行查看。
	>> 第二列 设备挂载点，即挂载磁盘到某个目录下，该目录即为该磁盘的topdir。
	>> 第三列 磁盘文件系统格式，ext2，ext3，ntfs，vfat，btrfs等。
	>> 第四列 文件系统挂载选项，包括以下几点:
		>>> rw/ro 读写方式挂载还是只读方式挂载。
		>>> suid/nosuid 是否允许suid的存在，例如普通用户提升权限。
		>>> dev/nodev 是否解析文件系统上的块特殊设备。
		>>> exec/noexec 限制文件系统内是否能够进行"执行"操作
		>>> auto/noauto 决定在启动时或者mount -a时，此文件系统是否被自动挂载。
		>>> user/nouser 是否允许用户使用mount命令挂载。
		>>> async/sync 是否以同步方式运行，即控制修改是否立刻体现到磁盘上，默认是async，即先放到内存缓冲区内，合适的时机写入。
		>>> nofail设备不存在时不报错。
	>> 第五列 是否被dump备份命令作用，有0(不做dump)，1(每天dump)，2(不定期dump)三种选择。
	>> 第六列 是否检验扇区，开机检测，使用fsck，有0(不做检验)，1(最早检验，一般/会设置)，2(1级别检验后检验)三种选择。
参考自`man fstab`及[linux之fstab文件详解](http://blog.csdn.net/richerg85/article/details/17917129)，以及[Linux命令-自动挂载文件/etc/fstab功能详解转](http://www.cnblogs.com/qiyebao/p/4484047.html)

## 简单总结
不清楚问题发生原因时，日志很重要，man手册很重要，百度也很重要。
