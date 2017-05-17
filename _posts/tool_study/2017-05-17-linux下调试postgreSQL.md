---
layout: post
title: linux下调试postgreSQL
category: 工具学习
keywords: linux,postgreSQL,gdb
---

linux下调试postgreSQL
===
编译前准备工作
---
1. 获取源代码

    [postgreSQL官方网站](https://www.postgresql.org)

2. 创建操作用户组和操作用户

    创建用户组

    `groupadd postgre`

    创建用户同时提供密码和home路径

    `useradd -g postgre -d /home/postgre -m postgre -p 123456`

3. 修改文件用户组和用户主
    由于获取代码时一般使用的不是刚刚创建的用户，为了避免发生各种权限问题，所以这里将文件的属组和属主都改成postgre。

    `chown postgre postgreSQL源代码`

    `chgrp postgre postgreSQL源代码`

    切换成postgre用户

    `su - postgre` 后输入密码即可

4. 解压源代码文件

    `tar -xvf postgreSQL源代码`

5. 为postgreSQL编译做配置使其可调试

    做这件是之前要先了解postgre源码配置都有哪些参数。

    `./configure --help`

    可以发现，有很多参数可以设置，如下：

    * --bindir=DIR            user executables [EPREFIX/bin]
    * --sbindir=DIR           system admin executables [EPREFIX/sbin]
    * --libexecdir=DIR        program executables [EPREFIX/libexec]
    * --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
    * --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
    * --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
    * --libdir=DIR            object code libraries [EPREFIX/lib]
    * --includedir=DIR        C header files [PREFIX/include]
    * --oldincludedir=DIR     C header files for non-gcc [/usr/include]
    * --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
    * --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
    * --infodir=DIR           info documentation [DATAROOTDIR/info]
    * --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
    * --mandir=DIR            man documentation [DATAROOTDIR/man]
    * --docdir=DIR            documentation root [DATAROOTDIR/doc/postgresql]
    * --htmldir=DIR           html documentation [DOCDIR]
    * --dvidir=DIR            dvi documentation [DOCDIR]
    * --pdfdir=DIR            pdf documentation [DOCDIR]
    * --psdir=DIR             ps documentation [DOCDIR]
    
    每个都设置比较麻烦，发现其实总的来说都依赖四个参数。
    * EPREFIX
    * PREFIX
    * DATAROOTDIR
    * DOCDIR

    可以比较取巧的将这四个参数都设置到/home/postgre/xxxx，这样安装文件就都在/home/postgre下了，对系统的影响比较小，而且也保留了原有结构。

    具体的格式如下：

    * PREFIX   --prefix=/home/postgre/pgsql                    
    * EPREFIX  --exec-prefix=/home/postgre/eprefix         
    * DATAROOT --datadir=/home/postgre/datarootdir          
    * DOCDIR   --docdir=/home/postgre/docdir                
    
    另外一些参数主要决定postgreSQL的可调试性，比较自由，调试的话，一般多支持一些特性会比较好。

    因此这些参数中，我选择了一些参数支持。

    * --enable-nls=zh_CN.UTF-8
    * --enable-debug CFLAGS="-O0"                可调试
    * --enable-profiling              性能分析，包括cpu利用率，内存占有等。
    * --enable-dtrace                动态跟踪，实时系统分析寻找性能及其他问题。
    * --enable-cassert              检查assert

    最后，有些参数，主要决定了postgreSQL的一些可定制化的功能，可以选择需要的进行支持。

    * --with-PACKAGE=yes    支持包

    一些不错的参数可以提高postgreSQL的效率。

    * --with-bolcksize=BLOCKSIZE
    * --with-segsize=SEGSIZE
    * --with-wal-blocksize=BLOCKSIZE
    * --with-wal-segsize=SEGSIZE

    由于我是在虚拟机里面安装的，磁盘空间有限，就不调整以上的参数了。

    以下一些参数用来支持各种脚本语言。

    * --with-tcl            tcl支持
    * --with-tclconfig=DIR
    * --with-perl         perl支持
    * --with-python    python支持
    * --with-gssapi    gssapi支持  需要本机有相关内容
    * --with-pam        pam支持
    * --with-bsd-auth     bsd
    * --with-ldap
    * --with-bonjour
    * --with-openssl
    * --with-selinux
    * --with-systemd
    * --with-libedit-preferred
    * --with-uuid=LIB
    * --with-ossp-uuid
    * --with-libxml
    * --with-libxslt                                    
    * --with-system-tzdata=DIR            时区路径

    根据实际的环境，确定自己需要选择哪些参数支持，由于我在虚拟机内安装，受到的局限比较多。所以最后选择的参数如下：

    `./configure --prefix=/home/postgre/pgsql --exec-prefix=/home/postgre/eprefix --datadir=/home/postgre/datarootdir --docdir=/home/postgre/docdir --enable-nls=zh_CN.UTF-8 --enable-debug CFLAGS="-O0" --enable-profiling --enable-cassert --with-python --with-libxml`

编译安装
---
1. 编译postgreSQL
    `make`

2. 安装
    `make install`

初始化库
---
1. 创建数据文件存放文件夹
    `mkdir /home/postgre/data`
    
2. 进入postgreSQL可执行文件路径
    `cd /home/postgre/eprefix/bin`            --请根据实际路径填写
    
3. 初始化数据库
    `./initdb -D /home/postgre/data`
    

启动数据库
---
* 执行可执行程序pg_ctl进行数据库的启动
    `./pg_ctl -D /home/postgre/data -l logfile start`

连接数据库
---
* 执行可执行程序psql连接数据库
    `./psql -h 127.0.0.1 -d postgres -p 5432 -U postgre`

查看后台pid
---
* 查看后台pid，调试时需要指定pid
    `select pg_backend_pid();`

gdb调试
---
* 开始postgreSQL的调试
    `gdb ./postgres xxxx`            --xxxx指端口号
