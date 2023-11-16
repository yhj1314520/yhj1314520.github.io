---
title: Spring八股
tags:
  - Java
  - Spring
  - 基础知识
categories:
  - 后端
  - 编程语言
  - Java
  - SpringBoot
date: 2023-11-14 22:35:04
---
## Spring

### 是什么
### 怎么用
### 怎么实现
### 问题
#### 事务
#### 异常处理


### Spring 注解
#### 注解定义
#### 注解实现
##### 注解原理
##### 注解处理

#### 注解使用(哪些注解)


## Bean
### 是什么
### 怎么用
### 怎么实现

### 问题
#### 单例Bean线程安全
	条件：判断Spring 单例Bean是否线程安全，需要根据bean对象中是否有可变的状态（变量）。
	为啥：多个线程对单例bean的状态存在进行修改（体现在单例成员的属性修改）
	结论：单例singleton的bean没有需要注入的状态（无可修改变量），例如Service类或Dao类。如果有bean中有可修改的变量，则需要改变作用域从singleton到prototype或者自定义并发处理逻辑
	【待处理https://cloud.tecent.com/developer/article/1743283】
#### 作用域
#### 生命周期
#### 缓存依赖





## Spring AOP
### AOP定义（逻辑

### AOP实现
### AOP使用

## Spring IoC
### IoC定义
### IoC实现
### IoC使用
#### IoC执行流程



## SpringBoot
### 自动装配
#### 启动流程

#### 条件注解
#### 配置属性绑定

#### 事件监听机制


### 使用
#### 数据源配置

