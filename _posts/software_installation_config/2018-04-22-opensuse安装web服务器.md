---
layout: post
title: openSUSE安装web服务器
category: 软件安装配置
keywords: openSUSE, apache2, php7, mysql
---

## 1 安装php7

`sudo yast2`							--进去之后选择php7需要的相关组件，包括php7-devel等

## 2 安装配置apache2

`sudo yast2`							--安装apache2及相关组件，同时记得安装php7模块的支持,apache2-mod_php7

`systemctl enable apache2.service`		--允许开机启动

`systemctl start apache2.service`		--启动apache2服务器

`vim /etc/apache2/httpd.conf`			--检查配置文件

* 检查配置文件，看DirectoryIndex对应值中是否有index.php
* 检查/etc/apache2/php7.conf是否存在，在/etc/apache2/httpd.conf中是否被引入
	+ 内容一般如下

	```
	 	<IfModule mod_php7.c>
       <FilesMatch "\.ph(p[345]?|tml)$">
           SetHandler application/x-httpd-php
       </FilesMatch>
       <FilesMatch "\.php[345]?s$">
           SetHandler application/x-httpd-php-source
       </FilesMatch>
        DirectoryIndex index.php4
        DirectoryIndex index.php5
        DirectoryIndex index.php
		</IfModule>

	```

* 如果没有引入，也没有上述文件，可以自己创建类似文件引入
	httpd.conf.local

	```
		LoadModule proxy_module /usr/lib64/apache2/mod_proxy.so
		LoadModule proxy_fcgi_module /usr/lib64/apache2/mod_proxy_fcgi.so
		LoadModule php7_module /usr/lib64/apache2/mod_php7.so
		AddType application/x-httpd-php .php
	```

	`IncludeOptional /etc/apache2/httpd.conf.local`

**注意，一定要安装php7支持，并且必须在配置中加载。**

至此，php7和apache2已经安装成功，可以通过在/srv/www/htdocs/下创建index.php来测试是否安装配置成功。

## 3 安装mysql

`sudo yast2`					--进入选择mysql及相关库安装，包括mysql-devel等

`systemctl start mysql.service`

`mysql -u root -p`

`select password('xxxxxx');`

设置root用户密码，需要密文。

创建新远程用户

执行mysql_secure_installation来保证安全访问。

**注意，要创建普通用户，避免远程用户访问。**

至此，mysql安装成功，可以通过mysql -u root -p来检查mysql是否正常提供服务。

## 4 安装wordpress

去[wordpress官方网站](https://cn.wordpress.org/)下载并解压，然后按照说明去做。

**注意，wordpress的上传、主题升级等会需要一个ftp用户，且需要该用户具有wordpress相应目录权限。**
**粗暴的解决方案是将对应的文件夹owner修改为ftp用户**
