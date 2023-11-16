---
title: SpringTransaction配置
tags:
  - Spring
  - CodingExp
categories:
  - 后端
  - CodingLanguage
  - Java
  - Spring
date: 2023-11-15 16:33:32
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
6. @Transactional注解可以在类上也可以在方法上。大多数情况下，方法上的事务，会首先执行？什么样的方法事务注解会优先于类级别呢？
7. Proppagation.REQUIERES_NEW的问题——Spring中在默认代理模式下，目标方法由外部调用时，才能被Spring的事务拦截器拦截，同一类中的方法调用不会被拦截。[https://blog.csdn.net/qq_34845394/article/details/102921479]
## Spring编程式事务

使用编码的方使用spring中提供的事务相关类来控制事务
### PlatformTransactionManager控制事务
**步骤1**——定义事务管理器PlatformTransactionManager
	用于开启事务、提交事务，回滚事务等操作，在spring中使用PlatformTransactionManager接口表示
```Java
public interface PlatformTransactionManager{
//获取一个事务（开启事务）
TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

//提交事务
void commit(TransactionStatus status) throws TransactionException;

//回滚事务
void rollback(TransactionStatus status) throws TransactionException;

}
```
**接口实现类** 有许多，以应对不同的环境：
- `JpaTransactionManager`:使用Jpa来操作db
- `DataSourceTransactionManager`:采用指定数据源的方式，使用JdbcTemplate||MyBatis||ibatis操作db
- `HibernateTransactionManager`:使用hibernate操作db
- `JtaTransactionManager`:通常是分布式事务下，使用Java中的jta操作db

**步骤2——** 定义事务属性TransactionDefinition
	定义事务属性，例如事务隔离级别（spring5种）、事务传播方式、事务超时时间、是否只读事务等。
**步骤3——** 开启事务，使用getTransaction方法
	该方法会返回一个`TransactionStatus`表示事务状态的一个对象，通过该对象种的一些方法可以用来控制事务的一些状态（例如提交还是回滚）
```Java
class TransactionStatus
``` 
**保存连接** 执行完getTransaction方法后，Spring会从datasource中开启连接。将datasource和其对应的connection 存放在名resource的threadlocal中，以便后期获取 
	

**步骤4——** 执行业务操作
	在jdbc中，和事务管理使用的是同一个datasource,所以，jdbcTemplate会从threadlocal中找有无关联的连接，有则使用没有则重新创建。通过相同connection来使得jdbcTemplate参与到spring中事务
**步骤5——** 提交or回滚

#### 案例代码
from[https://blog.csdn.net/qq_32062699/article/details/109195228]
```Java
@Test
	public void test1() throws Exception{
		//定义一个数据源
		DataSource dataSource = new DtaSource();
		dataSource.set......;//(url,DriverClassName)
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		//定义事务管理其，指定一个数据
		PlatfromTransactionManager platformTransactionManager = new DataSourceTransactionManager(dataSource);
		//定义事务属性
		TransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
		//开启事务
		TransactionStatus transactionStatus = platformTransactionManager.getTransaction(transactionDefinition);
		//执行业务操作
		try{
			....//执行sql语句
			platformTransactionManager.commit(transactionStatus);
		}catch(Exception e){
			platformTransactionManager.rollback(transactionStatus);
		}
	
	}
```
### TransactionTemplate控制事务
 对platformTransactionManager的提交和回滚事务代码进行优化,同样需要定义事务管理器，定义事务属性。
 但是在开启事务时，选用TransactionTemplate
 提交时，两种方法
 - `executeWithoutResult`，无返回值场景
```Java
transactionTemplate.executeWithoutResult(new Consumer<TransactionStatus>(){
	@Override
	public void accept(TransactionStatus transactionStatus){
		....//执行业务操作
	}	
});
```
 -  `execute` ，返回值场景，这里以Integer为例
```Java
Integer result = transactionTemplate.execute(new TransactionCallback<Integer>(){
	@Nullable
	@Override
	public Integer doInTransaction(TransactionStatus status){
		return jdbcTemplate......;
	}
});
```


 回滚时，两种方法
 - 在提交方法内部执行transactionStatus.setRollbackOnly(), 将事务状态标注为回滚状态，spring会自动让事务回滚
 - 提交方法内部抛出任意异常即可
#### 案例代码
from[https://blog.csdn.net/qq_32062699/article/details/109195228]

```Java
@Test2
	public void test2() throws Exception{
		DataSource dataSource ....
		dataSource.set.....
		PlatformTransactionManager ....
		DefaultTransaction ....
		//创建TransactionTemplate对象
		TransactionTemplate transactionTemplate  = new TransactionTemplate(platformTransactionManager,transactionDefinition);
		//提交事务
		transactionTemplate.executeWithoutResult(new Consumer<TransactionStatus>(){
			@Override
			public void accept(TransactionStatus transactionStatus){
				....//执行业务操作
			}	
		});
	}
```