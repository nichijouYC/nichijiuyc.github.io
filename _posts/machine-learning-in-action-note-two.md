---
layout: post
title: "机器学习实战笔记(二)"
date: 2015-09-02 14:29:22 +0800
comments: true
categories: 
---
## 分类算法——K近邻算法(K-Nearest Nerghboor)

### 简单介绍

- 将未知样本与已知样本的特征属性作对比，计算与已有样本之间的距离
- 取与未知样本最小距离的k个已知样本，统计样本中出现次数最多的分类作为未知样本的分类
<!-- more -->

### 优缺点

- 优点
    - 精度高，对异常值不敏感
- 缺点
    - 复杂度高，效率低
    - 每次需计算未知样本点与每个已知样本点的距离，时间久
    - 每次分类时需要将所有已知点的特征属性存储下来，空间大

### 实现步骤

1. 计算未知点与已知点的距离(欧几里德距离)
2. 按距离递增排序
3. 选取与未知点距离最小的k个已知点
4. 确定k个点的所在类别
5. 将k个点类别最多的类别作为未知点的类别

### 实现工具包

Python机器学习工具包`sklearn`中的`KNeighborsClassifier`类
{% codeblock lang:python %}
#-*- coding:utf8 -*-
from sklearn.neighbors import KNeighborsClassifier
#创建分类器，设置k值为3，取最近3个邻居
neigh = KNeighborsClassifier(n_neighbors = 3)
X = [[1.0, 1.1], [1.0, 1.0], [0, 0], [0, 0.1]]
Y = ['A', 'A', 'B', 'B']
neigh.fit(X,Y)
print neigh.predict([[1,0.5]])
print neigh.predict_proba([[1,0.5]])
{% endcodeblock %}
