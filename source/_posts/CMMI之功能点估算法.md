---
title: CMMI之功能点估算法
tags: [CMMI]
grammar_cjkRuby: true
categories: [项目管理]
date: 2017-05-19
---


## FP功能点估算分类
FP功能点估算法将功能点分为以下5类：
- ILF：Internal Logical File内部逻辑文件
- EIF: External Interface File外部接口文件
- EI: External Input外部输入
- EO: External Output外部输出
- EQ: External Inquiry外部查询

ILF和EIF属于数据类型的功能点，EI、EO、EQ属于人机交互类型的功能点。

以外贸订单系统项目为例：

![enter description here][1]

录入订单、修改订单、删除订单是EI；
查询订单是EO
统计订单是EQ
汇率查询转换系统为EIF
订单和客户资料是ILF

## 个人理解
### EI、EO、EQ
这三类，是针对功能点的分类：

- EI：所有会导致内部数据变化的功能点均属于此类，包括增、删、改
- EO：所有将内部数据导出从而使数据脱离系统的功能点均属于此类，包括数据导出、生成外部文件等
- EQ：所有查询类的功能点，即展示数据，但是数据并不脱离系统的功能点

### EIF、ILF
这二类，其实是针对实体的分类：

- EIF：可以理解为外部协作实体，如外部系统、独立文件、外部数据等
- ILF：可以理解为系统内部的实体，如角色、文件、数据等

### 判断模型

![enter description here][2]


备注：这里所说的数据是广义的，包括结构化数据、非结构化数据、文件等


 ## 参考资料：
  http://blog.sina.com.cn/s/blog_6d723ede01015xe0.html


  [1]: ./images/1495176047706.jpg " "
  [2]: ./images/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6%20%281%29.png " "