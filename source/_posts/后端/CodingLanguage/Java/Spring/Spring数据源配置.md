---
title: Spring数据源配置
date: 2023-11-16 11:36:19
tags:
  - Spring
  - CodingExp
---
## 数据源
保证应用程序与目标数据之间交互规范和协议的数据库或文件系统----对数据库交互操作的抽象
特性是封装了建立连接，向外暴露获取连接的接口

### 不提供连接池
Spring中提供的数据源，例如DriverManagerDataSource
**对每个连接请求建立新的连接，程序使用完毕后销毁**，交互频繁时明显存在大量时空开销
建议仅在测试时使用
### 提供连接池
创建和管理一组连接对象的技术，以Apaceh Jakarta Commons DBCP adn C3P0为例
#### Apache连接池技术


## 数据源配置
无论是自动配置还是手动配置，均需要在pom.xml添加相关依赖
eg:

### SpringBoot自动配置
#### 原理

#### 示例
在`application.propertites`或`application.yml`文件中添加以下配置
```java
//
spring.datasource.url=jdbc:mysql://localhost/test
//
spring.datasource.username=...
//
spring.datasource.paddword=...
//数据库驱动程序类名
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
//配置数据源类型，默认情况下使用HikariCP，其他则需要在pom.xml文件中添加相应的依赖项
spring.datasource.type=....
//配置连接池大小
spring.datasource.hikari.maximum-pool-size=
//

//
```

### 手动配置
#### 原理

#### 示例
```java
@Configuration
public calss DataSourceConfig{
	@Bean
	public DataSource dataSource(){
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
		dataSource.setDriverClassName("com.mysql.jdbc.Driver");
		dataSource.setUrl...
		return dataSource;
	}
}
```
通过Configuration注解将该类标记为配置类，通过Bean注解将dataSource标记为Bean
## 多数据源配置
如果需要同时连接多个数据库，在SpringBoot中使用多数据源
```java
@Configuration
public class DataSource Config{
	@Bean(name="primaryDataSource")
	@Primary
	@ConfigurationProperties(prefix="spring.datasource.primary")
	public DataSource primaryDataSource(){
		return DataSourceBuilder.create().build();
	}
	@Bean(name="secondaryDataSource")
	@ConfigurationProperties(prefix="spring.datasource.secondary")
	public DataSource secondaryDataSource(){
		return DataSourceBuilder.create().build();
	}
}
```
通过使用@ConfigurationProperties注解来指定配置文件中的前缀，确保SpringBoot自动将属性绑定到DataSource对象上。
通过@Primary注解确保没有指定具体的数据源名称时，自动选择primary

## 参考文献
[1]https://cloud.tecent.com/developer/article/2258704
[2]https://juejin.cn/post/6844903654223265800#heading-2
