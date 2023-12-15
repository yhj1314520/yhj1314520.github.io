---
title: Java特性八股
tags:
  - Java
  - 基础知识
  - 语言特性
categories:
  - 后端
  - CodingLanguage
  - Java
  - Language
date: 2023-11-15 15:06:07
---
## ReadMe
## 设计模式
### 六大原则
- 开闭原则——对扩展开发，对修改关闭
- 里氏代换原则——基类可以出现的地方，子类一定可以出现，抽象化的具体实现
- 依赖倒转原则——针对接口编程，依赖于抽象而不依赖于具体
- 接口隔离原则——使用多个隔离的接口，比使用耽搁接口要好，降低类之间的耦合度
- 迪米特法则（最少知道原则）——一个实体应当尽量少地与其他实体之间发生相互作用
- 合成复用原则——尽量使用合成/聚合的方式，而不是使用继承
### 代理模式
#### 基础
**定义**：使用代理对象代替真实对象的访问。
**特点**：扩展目标对象的功能
**分类**：
- 静态代理：对目标对象的每个方法的增强（添加功能）都是手动完成（写在代码里的）；一旦新增方法需要对目标对象和代理对象进行修改
- 动态代理：不需要针对每个目标类单独创建一个代理类
	- JDK代理：
	- CGLIB代理：
	- 区别：JDK只能代理**实现了接口的类或者直接代理接口**，CGLIB可代理**未实现任何接口的类**。获取代理类的方式不同，**JDK是通过proxy类**，**CGLIB则是直接需要代理的目标类**。【Spring5.x使用CGLIB，是因为jdk动态代理只能基于接口，代理生成的对象只能赋值给**接口变量**，而Cglib则是通过子类实现的，代理对象既可以赋值给**实现类**也可以赋值给**接口**。且Cglib速度比jdk动态代理更快，性能更好】
#### JDK代理
##### 逻辑
##### Code
#### CGLIB代理
##### 逻辑
##### Code


### 单例模式
#### 基础
**定义**：一个类**仅**有一个实例
**特点**：1、构造器私有 2、持有自己类型的属性 3、对台提供获取实例的静态方法
**分类**：
- 懒汉模式，延迟初始化，调用时才实例对象，线程不安全
- 饿汉模式，加载类时进行实例化，线程安全但易产生垃圾（不需要实例时）
- 双检索模式，双重校验锁，【两次判断null，第一次避免不要的实例，类Synchronized加锁，第二次时为了进行线程同步，避免多线程问题，对创建的实例进行volatile修饰避免重排序在多线程下发生危险】
```java

```
### 工厂模式
### 装饰器模式
不改变原有对象的情况下扩展其功能，通过**组合**替代**继承**来扩展原始功能。但是逻辑上采用继承相同的抽象类或实现相同的接口。
eg:【BufferedInputStream】增强【FileInputStream】的功能
```java
BufferedInputStream bis = new BufferedInpuStream(new FileInputStream(filename));

ZipInputStream zis = new ZipInputStream(bis);
```
### 适配器模式
适配器模式中被适配的对象或者类称为**适配者**，作用于适配者的对象或者类称为**适配器**【**类适配器，对象适配器**】类适配器使用继承实现，对象适配器使用组合实现。
eg:通过适配器，将字节流对象适配成一个字符流对象，这样可以直接通过字节流对象读写字符数据。【InputStreamReader 使用StreamDecoder实现**字节流到字符流的转换**&& OutputStreamWriter 使用 StreamEncoder 实现**字符流到字节流的转换**】
```java
// InputStreamReader 是适配器，FileInputStream是被适配的类

InputStreamReader isr = new InputStreamReader(new FileInputStream(fileName), "UFT-8");

//BufferedReader 增强InputStreamReader 的功能（装饰器模式）

BufferedReader bufferedReader = new BufferedReader(isr);
```
## 代理

## 杂七杂八
### 自动拆箱装箱

基本数据类型和包装类的互换。其中装箱是调用包装类的valueOf()方法，拆箱则是调用包装类的xxxValue()
编译器有时会自动拆箱，比如三元式[解析空指针异常](https://mp.weixin.qq.com/s/ExIaOUpJM4U3iEb_f7Bvpw)
### 泛型
### lamda表达式
