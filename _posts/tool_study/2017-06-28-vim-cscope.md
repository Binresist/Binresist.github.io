---
layout: post
title: vim插件cscope使用方法
category: 工具学习
keywords: vim,插件,cscope
---

## 1 安装cscope
安装方式比较多样，可以在各自linux软件管理工具中安装，也可以去官网下载安装。

## 2 插件安装
这里选择的是Vundle来管理vim插件，所以只需要在.vimrc中添加`Plugin 'brookhong/cscope.vim'`，然后执行`:PluginInstall`就搞定了。

## 3 生成标记库
cscope在执行过程中实际上是通过扫描标记库来首先找到标记以及标记的位置，从而实现跳转的，因此首先需要为个人的代码项目生成一个完整的标记库。

```c
binresist@binresist:~/work_space/KStoreCode/Kstore> cscope --help
Usage: cscope [-bcCdehklLqRTuUvV] [-f file] [-F file] [-i file] [-I dir] [-s dir]
              [-p number] [-P path] [-[0-8] pattern] [source files]

			  -b            Build the cross-reference only.
			  -C            Ignore letter case when searching.
			  -c            Use only ASCII characters in the cross-ref file (don't compress).
			  -d            Do not update the cross-reference.
			  -e            Suppress the <Ctrl>-e command prompt between files.
			  -F symfile    Read symbol reference lines from symfile.
			  -f reffile    Use reffile as cross-ref file name instead of cscope.out.
			  -h            This help screen.
			  -I incdir     Look in incdir for any #include files.
			  -i namefile   Browse through files listed in namefile, instead of cscope.files
			  -k            Kernel Mode - don't use /usr/include for #include files.
			  -L            Do a single search with line-oriented output.
			  -l            Line-oriented interface.
			  -num pattern  Go to input field num (counting from 0) and find pattern.
			  -P path       Prepend path to relative file names in pre-built cross-ref file.
			  -p n          Display the last n file path components.
			  -q            Build an inverted index for quick symbol searching.
			  -R            Recurse directories for files.
			  -s dir        Look in dir for additional source  files.
			  -T            Use only the first eight characters to match against C symbols.
			  -U            Check file time stamps.
			  -u            Unconditionally build the cross-reference file.
			  -v            Be more verbose in line mode.
			  -V            Print the version number.

			  Please see the manpage for more information.
```
参数比较多，但一般来说`cscope -Rbq`就够用了，R表示递归，默认cscope会进入搜索页面，b可以不进入这个页面，q会生成索引，加快查找速度。

## 4 在vim中添加标记库
上面执行过后会生成三个文件，cscope.out就是生成的数据库，cscope.in.out和cscope.po.out是q控制对应的索引。使用vim打开项目中某个文件，然后`cscope add cscope.out`就能够识别该数据库，方便后续使用。

## 5 vim中使用cscope查找
使用`cscope find s xxx`来进行cscope的使用即可。s是参数，其他参数及意义如下：
```shell
s: 查找C语言符号，即查找函数名、宏、枚举值等出现的地方
g: 查找函数、宏、枚举等定义的位置，类似ctags所提供的功能
d: 查找本函数调用的函数
c: 查找调用本函数的函数
t: 查找指定的字符串
e: 查找egrep模式，相当于egrep功能，但查找速度快多了
f: 查找并打开文件，类似vim的find功能
i: 查找包含本文件的文件
```
