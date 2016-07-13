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

    ＊ /etc/passwd - 使 用 者 帐 号 资 讯，可以查看用户信息
    ＊ /etc/shadow - 使 用 者 帐 号 资 讯 加 密
    ＊ /etc/group - 群 组 资 讯
    ＊ /etc/default/useradd - 定 义 资 讯
    ＊ /etc/login.defs - 系 统 广 义 设 定
    ＊ /etc/skel - 内 含 定 义 档 的 目 录

