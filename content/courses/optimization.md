---
title: "最优化算法"
layout: page
date: 2016-07-23 20:35
---

[TOC]

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

# 最优化算法 #
	最优化方法主要运用数学方法研究各种系统的优化途径及方案，为决策者提供科学决策的依据，下面将简单介绍几种优化算法：最速下降法、牛顿法、拟牛顿法、共轭梯度法。
	
## 最速下降法 ##

### 1. 定义 ###
	以负梯度方向作为极小化方法的下降方向，就叫做最速下降法。
	
### 2. 为什么负梯度方向是下降最快的方向？ ###
	(A) 将目标函数$f(x)$在$x_{k}$泰勒级数展开，如下所示：
	$$ f(x) = f(x_{k}) + \nabla^Tf(x_{k})(x - x_{k}) + o(x_{k}) $$
	其中$x_{k}$表示滴k次极值迭代点，$\nabla^Tf(x_{k})$是f(x)在$x_{k}$处的偏导，$o(x_{k})$表示高阶无穷小。
	(B) 将迭代公式$x_{k+1} = x_{k} + \alpha_{k}p_{k}$ (其中p是下降方向，有$||p||=1$，而$\alpha$是迭代步长，可以通过wolfe condition得到) 带入上式，可得
	$$ f(x_{k} + {\alpha}p) = f(x_{k}) + \alpha\nabla^Tf(x_{k})p + o(x_{k}) $$
	其中为了让$f(x_{k} + {\alpha}p) < f(x_{k})$，需要满足$\nabla^Tf(x_{k})p < 0$ (o(x_{k})高阶无穷小，可以忽略)。$\nabla^Tf(x_{k})$和p都是向量，从而满足$\nabla^Tf(x_{k})p$是向量内积，因此满足：$\cos{\theta} = \frac{\nabla^Tf(x_{k})p}{||\nabla^Tf(x_{k})||||p||}$，众所周知，$-1 \leq\ cos{\theta} \leq 1$，因此当$cos{\theta}=-1$时，$\nabla^Tf(x_{k})p$取得最小值。为使等式成立，梯度向量p需要满足$p=-\frac{\nabla^Tf(x_{k})}{||\nabla^Tf(x_{k})||}$，也即负梯度方向，沿着这个方向目标函数值下降最快。
	
### 3. 优缺点 ###

	* 优点：程序简单，计算量小；对初始点没有特别的要求；最速下降法是整体收敛的，且为线性收敛。
	* 缺点：它只在局部范围内具有“最速”属性， 对整体求解过程，它的下降速度是缓慢的。靠近极小值时速度减慢；可能会'之字型'地下降。


### 4. 为什么“慢”的分析？###

	记$x_{k+1} = x_{k} + {\alpha}p$，$\psi(\alpha) = f(x_{k} + {\alpha}p)$，在确定$p=-\frac{\nabla^Tf(x_{k})}{||\nabla^Tf(x_{k})||}$后，需要优化$\alpha$使得$\psi(\alpha)$最小，此时只与参数$\alpha$有关，由精确line search方法，可知最优$\alpha$满足$\psi(\alpha) ＝ 0$，从而有$\nabla^Tf(x_{k+1})p=\nabla^Tf(x_{k+1})\nablaf(x_{k})=0$，因此可以看出前后两次搜索方向是相交的。
	最速下降法采用取目标函数的一阶部分（假设函数是线性），从而搜索线性的，每两次相邻搜索方向间是相交的，因而在一定程度导致收敛速度缓慢。
	
### 5. 参考 ###
	1. http://www.codelast.com/?p=2573	
	2. http://www.codelast.com/?p=8006


 