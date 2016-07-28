---
title: "TensorFlow学习"
layout: page
date: 2016-07-25 23:03
---

[TOC]

# TensorFlow学习 #

## 1. 基于Pip安装TensorFlow ##

### 1.1 安装Pip ###
	Pip是一个Python的软件包安装与管理工具。

	# Ubuntu/Linux 64-bit
	$ sudo apt-get install python-pip python-dev

	# Mac OS X
	$ sudo easy_install pip

### 1.2 安装TensorFlow ###
	# Ubuntu/Linux 64-bit, CPU only, Python 2.7:
	$ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.8.0-cp27-none-linux_x86_64.whl

	# Ubuntu/Linux 64-bit, GPU enabled, Python 2.7. Requires CUDA toolkit 7.5 and CuDNN v4.
	# For other versions, see "Install from sources" below.
	$ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow-0.8.0-cp27-none-linux_x86_64.whl

	# Mac OS X, CPU only:
	$ sudo easy_install --upgrade six
	$ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/mac/tensorflow-0.8.0-py2-none-any.whl

### 1.3 测试 ###
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

### 1.4 问题 ###
	Q：-bash: /usr/local/bin/pip: No such file or directory
	A：python -m pip install --upgrade --force-reinstall pip

	Q：module compiled against API version 0xa but this version of numpy is 0x9
	A：升级numpy版本，一种解决方法是通过homebrew安装opencv即可，具体如下：
		brew install python
		brew tap homebrew/science
		brew install opencv

## 2. 基本使用 ##
	使用TensorFlow，需要明白几点：
		1. 使用图（graph）来表示计算任务.
		2. 在会话（Session）的上下文中执行图.
		3. 使用tensor表示数据.
		4. 通过变量（Variable）维护状态.
		5. 使用feed和fetch可以为任意的操作(arbitrary operation) 赋值或者从其中获取数据.


### 2.1 综述 ###
	TensorFlow是一个编程系统, 使用图来表示计算任务. 图中的节点被称之为op(operation 的缩写). 一个op获得0个或多个 Tensor, 执行计算, 产生0个或多个Tensor. 每个Tensor是一个类型化的多维数组. 例如, 你可以将一小组图像集表示为一个四维浮点数数组, 这四个维度分别是 [batch, height, width, channels].

### 2.2 计算图 ###
	TensorFlow程序通常被组织成一个构建阶段和一个执行阶段. 在构建阶段, op的执行步骤被描述成一个图. 在执行阶段, 使用会话执行执行图中的op. 例如, 通常在构建阶段创建一个图来表示和训练神经网络, 然后在执行阶段反复执行图中的训练op.

### 2.3 构建图 ###
	构建图的第一步，是创建source op, 其不需要任何输入，它的输出被传递给其它op做运算. 事例如下：

		>>> import tensorflow as tf
		# 创建一个常量 op, 产生一个 1x2 矩阵. 这个 op 被作为一个节点, 加到默认图中.
		# 构造器的返回值代表该常量op的返回值.
		>>> mx1 = tf.constant([[5.,5.]])
		# 创建另外一个常量op, 产生一个2x1矩阵.
		>>> mx2 = tf.constant([[2.],[2.]])
		# 创建一个矩阵乘法matmul op, 把‘mx1’和‘mx2’作为输入，返回值‘product’代表矩阵乘法的结果.
		>>> product = tf.matmul(mx1, mx2)

### 2.4 在一个会话中启动图 ###
	构造完后, 才能启动图. 启动图的第一步是创建一个Session对象, 如果无任何创建参数, 会话构造器将启动默认图。

		＃ 启动默认图
		>>> sess = tf.Session()
		# 调用sess的'run()'方法来执行矩阵乘法op, 传入 'product' 作为该方法的参数. 上面提到, 'product'代表了矩阵乘法op的输出, 传入它是向方法表明, 我们希望取回矩阵乘法o的输出. 整个执行过程是自动化的, 会话负责传递 op 所需的全部输入. op通常是并发执行的.
		# 函数调用'run(product)'触发了图中三个op(两个常量 op 和一个矩阵乘法op)的执行.
		# 返回值'result'是一个numpy `ndarray`对象.
		>>> result = sess.run(product)
		>>> print result
		[[ 20.]]
		#   任务完成, 关闭会话.
		>>> sess.close()

	除了如上显示调用close外, 也可以使用‘with’代码块来自动完成关闭动作.

		>>> with tf.Session() as sess:
					result = sess.run(product)
					print result

### 2.5 交互式使用 ###
	为了便于使用诸如IPython之类的Python交互环境, 可以使用 InteractiveSession代替Session类, 使用 Tensor.eval()和Operation.run()方法代替 Session.run(). 这样可以避免使用一个变量来持有会话.

		# 进入一个交互式TensorFlow会话
		>>> sess = tf.InteractiveSession()
		>>> x = tf.Variable([1.0, 2.0])
		>>> a = tf.constant([3.0, 3.0])
	  # 使用初始化器initializer op的run()方法初始化 'x'  
		>>> x.initializer.run()
		# 增加一个减法sub op, 从 'x' 减去 'a'. 运行减法op, 输出结果
		>>> sub = tf.sub(x, a)
		>>> print sub.eval()
		[-2. -1.]

### 2.6 Tensor ###
	TensorFlow程序使用tensor数据结构来代表所有的数据, 计算图中, 操作间传递的数据都是tensor. 你可以把TensorFlow tensor看作是一个n维的数组或列表. 一个tensor包含一个静态类型rank, 和一个shape.

### 2.7 Fetch ###
	为了取回操作的输出内容, 可以在使用Session对象的run()调用 执行图时, 传入一些tensor, 这些tensor会帮助你取回结果. 在之前的例子里, 我们只取回了单个节点state, 但是你也可以取回多个tensor:

		input1 = tf.constant(3.0)
		input2 = tf.constant(2.0)
		input3 = tf.constant(5.0)
		intermed = tf.add(input2, input3)
		mul = tf.mul(input1, intermed)

		with tf.Session() as sess:
				result = sess.run([mul, intermed])
				print result

				# 输出:
				# [array([ 21.], dtype=float32), array([ 7.], dtype=float32)]

### 2.8 Feed
	feed使用一个tensor值临时替换一个操作的输出结果. 你可以提供feed数据作为run() 调用的参数. feed 只在调用它的方法内有效, 方法结束, feed 就会消失. 最常见的用例是将某些特殊的操作指定为 "feed" 操作, 标记的方法是使用 tf.placeholder() 为这些操作创建占位符.

		input1 = tf.placeholder(tf.float32)
		input2 = tf.placeholder(tf.float32)
		output = tf.mul(input1, input2)

		with tf.Session() as sess:
		  print sess.run([output], feed_dict={input1:[7.], input2:[2.]})

		# 输出:
		# [array([ 14.], dtype=float32)]
