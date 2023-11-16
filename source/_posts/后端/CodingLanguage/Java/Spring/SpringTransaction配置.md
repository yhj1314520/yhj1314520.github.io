---
title: SpringTransaction配置
date: 2023-11-15 16:33:32
tags:
  - Spring
  - Coding
---
## Spring 声明式事务
### 基于XML配置

### 基于@Transaction注解
#### 代码
```xml
<!-- 配置事务管理器-->
<bean id = "transactionManger"
	class = "org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<!--注入数据库连接池-->
	<property name = "dataSource", ref="dataSource"/>
</bean>
<!-- 使用基于注解方式配置事务-->
<tx:annotation-driven transaction-manager="transactionManger"/>
```
接下来只需要在Service类或方法上面添加@Transactional注解即可

#### 问题
1. 默认配置下Spring只会回滚**运行时、未检查异常(继承RuntimeException)** 或者Error。但是我们可以通过**rollbackFor** 来指定要处理的异常【待添加相关代码示例】
2. `@Transactional` 注解只能应用到**public** 修饰的方法上，在其他(protected...)等方法使用，不会报错，但是*该方法不会展示已配置的事务设置*？【待理解最后一句】
3. `@Transaction`注解虽然可以被应用到接口定义或接口方法、类定义和类的public方法，但是建议在**具体的类或方法** 使用`@Transactional`注解，而不是在类实现的**接口** 上----
4. 如果`@Transactional`注解被写在Service类上面，表示该类中所有方法均被事务管理，但是有些查询方法不需要事务管理
```Java
@Transactional(propagation= Propagation.NOT_SUPPORTED, readOnly=true)
//NOT_SUPPORTED表示以非事务方式运行，如果当前存在事务则暂停当前事务
//readOnly表示是否为只读事务，设置为true可以忽略无需事务的方法
```
5. 如果需要手动回滚事务，两种方法可选
	- service层处理事务，service中的方法中不做异常捕获，或在catch语句中增加`throw new RuntimeException()` 语句，以便AOP捕获异常再回滚，在service上层(eg:view层Controller)捕获该异常处理
	- 在service层方法的catch语句中增加：`TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()` 语句，手动回滚，无需上层处理

## Spring编程式事务
	使用编码的方使用spring中提供的事务相关类来控制事务
### PlatformTransactionManager控制事务

### TransactionTemplate控制事务