---
layout: post
title: ubuntu管理PPA (Personal Package Archive-个人软件包存档）
category: 软件安装配置
keywords: ubuntu,PPA
---


# ubuntu管理PPA (Personal Package Archive-个人软件包存档）

## 选择PPA源
[PPA源for ubuntu][1]
可以在这里选择自己感兴趣的源。

## 添加源
sudo add-apt-repository ppa:winski/chromium
将上面的winski/chromium替换为你要添加的源。

## 更新
sudo apt-get update

## 清除不需要的/无效的源
sudo add-apt-repository --remove ppa:winski/chromium
和添加源一样，替换掉winski/chromium

[1]:https://launchpad.net/ubuntu/+ppas

