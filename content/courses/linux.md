---
title: "Linux Command"
layout: page
date: 2016-07-13 23:35
---

[TOC]

# Linux用户管理---创建账户和删除账户 #

## 常用命令 ##

### 创建用户 ###

	adduser：会自动为创建的用户制定目录、系统shell版本，会在创建时输入用户密码。
    useradd：需要使用参数选项指定上述基本设置，如果不使用任何参数，则创建的用户无密码、无主目录、没有制定shell版本.


### 用户删除命令 ###

	userdel

## 使用adduser ##
	如：adduser   apple
	默认情况下：

	adduser在创建用户时会主动调用  /etc/adduser.conf；

	在创建用户主目录时默认在/home下，而且创建为 /home/用户名.   

	如果主目录已经存在，就不再创建，但是此主目录虽然作为新用户的主目录，而且默认登录时会进入这个目录下，但是这个目录并不是属于新用户，当使用userdel删除新用户时，并不会删除这个主目录，因为这个主目录在创建前已经存在且并不属于这个用户.

	为用户指定shell版本为：/bin/bash



##  使用useradd ##

	如 

	sudo  useradd  -d  "/home/tt"   -m   -s "/bin/bash"   tt

	解释：   -d   “/home/tt" ：就是指定/home/tt为主目录.

            -m   就是如果/home/tt不存在就强制创建.

            -s    就是指定shell版本.

	修改tt密码：

	$  sudo passwd tt

	ps：
		* -d：           指定用户的主目录.
		* -m：          如果存在不再创建，但是此目录并不属于新创建用户；如果主目录不存在，则强制创建； -m和-d一块使用.
		* -s：           指定用户登录时的shell版本.
		* -M：           不创建主目录


## 删除用户命令 ##

### 只删除用户 ###
	sudo   userdel   用户名

### 连同用户主目录一块删除 ###
	sudo  userdel   -r   用户名

## 相关文件 ##

    1. /etc/passwd - 使 用 者 帐 号 资 讯，可以查看用户信息
    2. /etc/shadow - 使 用 者 帐 号 资 讯 加 密
    3. /etc/group - 群 组 资 讯
    4. /etc/default/useradd - 定 义 资 讯
    5. /etc/login.defs - 系 统 广 义 设 定
    6. /etc/skel - 内 含 定 义 档 的 目 录


# Linux命令：ln命令 #

## 概述 ##
	ln的功能是为某一个文件在另外一个位置建立一个同步的链接。当我们需要在不同的目录，用到相同的文件时，我们不需要在每一个需要的目录下都放一个必须相同的文件，我们只要在某个固定的目录，放上该文件，然后在其它的目录下用ln命令链接（link）它就可以，不必重复的占用磁盘空间.

## 命令格式 ## 
	ln [参数][源文件或目录][目标文件或目录]

## 命令功能 ##
	Linux文件系统中，有所谓的链接可以将其视为档案的别名，而链接又可分为两种 : 硬链接与软链接，硬链接的意思是一个档案可以有多个名称，而软链接的方式则是产生一个特殊的档案，该档案的内容是指向另一个档案的位置。硬链接是存在同一个文件系统中，而软链接却可以跨越不同的文件系统

### 软链接 ###
	1. 以路径的形式存在.类似于Windows操作系统中的快捷方式.
	2. 可以跨文件系统,硬链接不可以.
	3. 可以对一个不存在的文件名进行链接.
	4. 可以对目录进行链接.

### 硬链接 ###
	1. 以文件副本的形式存在, 但不占用实际空间.
	2. 不允许给目录创建硬链接.
	3. 只有在同一个文件系统中才能创建

## 命令参数 ##

### 必要参数 ###
	* -b 删除，覆盖以前建立的链接
	* -d 允许超级用户制作目录的硬链接
	* -f 强制执行
	* -i 交互模式，文件存在则提示用户是否覆盖
	* -n 把符号链接视为一般目录
	* -s 软链接(符号链接)
	* -v 显示详细的处理过程

### 选择参数 ###
	* -S “-S<字尾备份字符串> ”或 “--suffix=<字尾备份字符串>”
	* -V “-V<备份方式>”或“--version-control=<备份方式>”
	* --help 显示帮助信息
	* --version 显示版本信息

## 使用实例 ##

### 实例1：给文件创建软链接 ###
	命令：ln -s log2013.log link2013
	
	输出：
	[root@localhost test]# ll
	-rw-r--r-- 1 root bin      61 11-13 06:03 log2013.log
	[root@localhost test]# ln -s log2013.log link2013
	[root@localhost test]# ll
	lrwxrwxrwx 1 root root     11 12-07 16:01 link2013 -> log2013.log
	-rw-r--r-- 1 root bin      61 11-13 06:03 log2013.log

	说明：为log2013.log文件创建软链接link2013，如果log2013.log丢失，link2013将失效

### 实例2：给文件创建硬链接 ###
	命令：ln log2013.log ln2013
	
	输出：
	[root@localhost test]# ll
	lrwxrwxrwx 1 root root     11 12-07 16:01 link2013 -> log2013.log
	-rw-r--r-- 1 root bin      61 11-13 06:03 log2013.log
	[root@localhost test]# ln log2013.log ln2013
	[root@localhost test]# ll
	lrwxrwxrwx 1 root root     11 12-07 16:01 link2013 -> log2013.log
	-rw-r--r-- 2 root bin      61 11-13 06:03 ln2013
	-rw-r--r-- 2 root bin      61 11-13 06:03 log2013.log
	
	说明：为log2013.log创建硬链接ln2013，log2013.log与ln2013的各项属性相同

## 说明 ##
	1. 目录只能创建软链接.
	2. 目录创建链接必须用绝对路径，相对路径创建会不成功，会提示：符号连接的层数过多 这样的错误.
	3. 在链接目标目录中修改文件都会在源文件目录中同步变化.

 

