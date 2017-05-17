---
layout: post
title: dpkg 处理软件包 xxx (--configure)时出错 解决办法
category: 软件出错解决办法
keywords: dpkg,configure
---

# dpkg: 处理软件包 xxx (--configure)时出错 解决办法

## 第一步：备份
`$ sudo mv /var/lib/dpkg/info /var/lib/dpkg/info.bk`

## 第二步：新建
`$ sudo mkdir /var/lib/dpkg/info`

## 第三步：更新
`$ sudo apt-get update`
`$ sudo apt-get -f install`

## 第四步：替换
`$ sudo mv /var/lib/dpkg/info/* /var/lib/dpkg/info.bk`
//把更新的文件替换到备份文件夹

## 第五步：删除
`$ sudo rm -rf /var/lib/dpkg/info`
//把自己新建的info文件夹删掉

## 第六步：还原
`$ sudo mv /var/lib/dpkg/info.bk /var/lib/dpkg/info`
//把备份的info.bk还原
