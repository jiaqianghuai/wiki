---
title: "TensorFlow学习"
layout: page
date: 2016-07-25 23:03
---

[TOC]

# TensorFlow学习 #

## 基于Pip安装TensorFlow ##

### 安装Pip ###
	Pip是一个Python的软件包安装与管理工具。

	# Ubuntu/Linux 64-bit
	$ sudo apt-get install python-pip python-dev

	# Mac OS X
	$ sudo easy_install pip

### 安装TensorFlow ###
	# Ubuntu/Linux 64-bit, CPU only, Python 2.7:
	$ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.8.0-cp27-none-linux_x86_64.whl

	# Ubuntu/Linux 64-bit, GPU enabled, Python 2.7. Requires CUDA toolkit 7.5 and CuDNN v4.
	# For other versions, see "Install from sources" below.
	$ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow-0.8.0-cp27-none-linux_x86_64.whl

	# Mac OS X, CPU only:
	$ sudo easy_install --upgrade six
	$ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/mac/tensorflow-0.8.0-py2-none-any.whl

### 测试 ###
	localhost:bin huai$ python
	Python 2.7.10 (default, Oct 23 2015, 19:19:21)
	[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
	Type "help", "copyright", "credits" or "license" for more information.
	>>> import tensorflow
	>>> hello = tensorflow.constant('hello, tensorflow!')
	>>> sess = tensorflow.Session()
	>>> print(sess.run(hello))
	hello, tensorflow!
	>>> a = tensorflow.constant(10)
	>>> b = tensorflow.constant(32)
	>>> print(sess.run(a+b))
	42

### 问题 ###
	Q：-bash: /usr/local/bin/pip: No such file or directory
	A：python -m pip install --upgrade --force-reinstall pip

	Q：module compiled against API version 0xa but this version of numpy is 0x9
	A：升级numpy版本，一种解决方法是通过homebrew安装opencv即可，具体如下：
		brew install python
		brew tap homebrew/science
		brew install opencv
	



