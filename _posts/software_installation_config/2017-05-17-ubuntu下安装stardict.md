---
layout: post
title: Ubuntu 下安装stardict
category: 软件安装配置
keywords: ubuntu,stardict
---

Ubuntu 下安装stardict
===
安装
---
* apt安装
    `sudo apt-get install stardict`
    如果出现依赖问题，可以执行以下语句解决依赖问题。
    `sudo apt-get -f install`
* 其他安装方式请自行百度

本地词典配置
---
1. 下载词典
    `wget http://download.huzheng.org/xxxx/xxxx.xx`
    xxxx和xxxx.xx要替换为自己的词典实际位置。
    可以先去http://download.huzheng.org/检查一下那些是自己需要的词典，当然，也可以去别的地方系在词典。
2. 解压安装
    stardict本地词典比较容易配置，只需要将下载的词典解压到/usr/share/stardict/dic/下即可，可能有些同学的路径需要修改为自己的实际路径。
    这里以.bz2为例：
    `sudo tar -jxvf xxxx.bz2 /usr/share/stardict/dic/`

屏幕取词设置
---
* 默认的屏幕取词是选中即取词，可能会导致一些操作不太方便，可以在首选项里配置。
