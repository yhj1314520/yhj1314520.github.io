---
title: MySQL八股
tags:
  - 基础知识
  - Database
  - MySQL
categories:
  - 后端
  - Database
  - MySQL
date: 2023-11-15 15:28:30
---
## 基础
### 定义
### 怎么使用
#### 命令


### 问题
#### 数据类型选择
1. 设计金钱的选用数据类型，考虑到精度问题选用Decimal和Numric类型表示
	1. Decimal简介——column_name DECIMAL(P,D)，其中P代表有效数字精度1-65默认值为10，D代表小数点后的位数0-30默认值为0. 【D<=P】
	2. Decimal具有UNSIGNED属性【不接受负值】
	3. Deciaml具有ZEROFILL属性【使用0填充不满足列宽的部分】
	4. 对于每个部分使用**4字节** 存储**9位数的每个倍数** 【待调查】
2. 

## 日志（一致性）
## 索引（持久性）

## 事务（隔离性）

## 锁（原子性）
