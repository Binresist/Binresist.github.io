---
layout: post
title: vmware下安装greenplum
category: 工具学习
keywords: vmware, greenplum, CentOS
---

## 1 准备虚拟机

安装VMWare Workstation，windows下安装的，比较简单，不再赘述。

## 2 准备CentOS虚拟机环境

greenplum依赖的软件比较新，因此这里选择的是CentOS 7,为了降低一些不必要的依赖缺乏问题，选择的是GUI服务器，安装了基本开发软件及GNOME。

### 2.1 下载CentOS

`wget http://mirrors.163.com/centos/7/isos/x86_64/CentOS-7-x86_64-Everything-1804.iso`

### 2.2 安装CentOS 7

使用VMWare Workstation安装CentOS 7，按照提示操作即可，操作比较简单，注意一些版本的兼容性问题，这里为了方便给一些低版本使用，选择的是6.5版本。

### 2.3 配置虚拟机网络

由于greenplum需要通过网络进行连接，以及方便主机连接到虚拟机内进行调试，这里需要保证虚拟机和宿主机网络互通。
网络配置，VMWare Workstation相对于VirtualBox来说，网络配置要容易一些，为了方便后续主机和虚拟机互联，这里选择的是桥接网络。

* 需要注意的是，桥接时，选择正确的网卡。
* 对于非DHCP的网络，在/etc/sysconfig/network-scripts/ifcfg-xxxx中进行网络配置，需要保证配置的网络IP与主机在同一网段内，网关和子网掩码也要一致。
* 上面说的xxxx为你需要配置的虚拟机网卡名称，进入CentOS内，执行ifconfig中看到的。

### 2.4 准备用户

一般来说，为了系统安全，不建议使用root用户来安装各类软件，这里创建一个用户gp，用来方便操作greenplum。
为了解决环境依赖问题，用户可能需要sudo权限，下面创建了gp用户之后，将其加入sudo列表。

`useradd gp`	--添加用户
`visudo`	--操作方法类似与vim
		--仿照**root    ALL=(ALL)       ALL**这一行，然后添加一行，将root改成自己要添加的用户名即可，这里是gp

## 3 准备greenplum执行程序环境

### 3.1 下载源代码

`git clone https://github.com/greenplum-db/gpdb.git`

### 3.2 编译前环境依赖准备

安装软件遇到最多的大概就是软件依赖问题了，各种软件依赖、版本要求等等问题不胜其扰，这一点greenplum已经做了相应的工作，在源代码内有各种linux系统的脚本，方便用户解决依赖关系。

这里需要使用的是**README.CentOS.bash**，进入代码内该文件所在文件夹后，执行下面的程序。

`./README.CentOS.bash`

上面的脚本基本上可以满足大多数用户的需求，偶尔会有一些问题。

* pip版本较低`sudo pip install --upgrade pip`
* python模块默认版本的问题，会报一些错误，例如`conan 1.4.1 Required pylint < 1.9.0, >=1.8.1`。这样的错误一般安装了对应的模块版本即可。`sudo pip install pylint==1.8.1` `sudo pip install conan==1.4.1`

### 3.3 配置

`./configure --enable-debug --disable-orca --with-perl --with-python --with-libxml --prefix=/home/gp/greenplum/`

为了方便调试，上面加入了enable-debug，外挂优化器orca依赖的东西比较多，暂时没有配置进去。

### 3.4 编译

`make`

有时可能会报错，#include <openssl/hmac.h>处没有找到对应文件，这里说明没有安装openssl或者openssl-devel，利用yun安装后重新编译即可。

### 3.5 安装

`make install`		--这里在`./configure`中配置的路径gp用户具备权限，因此不用在前面加入sudo

## 4 准备MPP环境

为了简单起见，这里使用了greenplum官网对于ubuntu的单个master、两个segment的配置方案。

`cd /home/gp/greenplum`

`. greenplum_path.sh`
`cp docs/cli_help/gpconfigs/gpinitsystem_singlenode .`

`vim hostlist_singlenode`		--在当前路径配置hostlist_singlenode文件，里面写入主机名称，这里为greenplum
`vim gpinitsystem_singlenode`		--配置初始化所需的一些路径等

修改以下内容：

```
--hostlist_singlenode内写入主机名，此处是greenplum
MACHINE_LIST_FILE=./hostlist_singlenode
--两个segment的数据存储路径
declare -a DATA_DIRECTORY=(/home/gp/greenplum/gpdata1 /home/gp/greenplum/gpdata2)
--master主机名
MASTER_HOSTNAME=greenplum
--master数据存储路径
MASTER_DIRECTORY=/home/gp/gpmaster
```

需要保证上面配置的路径真实存在，且具备权限，否则后面的执行会报错。

`gpssh-exkeys -f hostlist_singlenode`			--配置ssh key交换

为了简单起见，初始化之前需要进行的系统配置被略过。

`gpinitsystem -c gpinitsystem_singlenode`		--初始化gp系统

## 创建数据库

`createdb demo`

## 连接数据库

`psql demo`
