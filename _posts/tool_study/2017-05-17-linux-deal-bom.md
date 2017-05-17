---
layout: post
title: linux文件BOM去除方法
category: 工具学习
keywords: linux,BOM,shell
---


## 查找包含BOM开头的文件
`grep -r -I -l $'^\xEF\xBB\xBF' ./`

## 去掉所有包含BOM开头的文件
`find . -type f -exec sed -i 's/\xEF\xBB\xBF//' {} \;`
