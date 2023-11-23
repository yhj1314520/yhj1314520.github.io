---
title: Spring数据源配置
tags:
  - Spring
  - CodingExp
categories:
  - 后端
  - CodingLanguage
  - Java
  - Spring
date: 2023-11-16 11:36:19
---
## 数据源
保证应用程序与目标数据之间交互规范和协议的数据库或文件系统----对数据库交互操作的抽象
特性是封装了建立连接，向外暴露获取连接的接口

### 不提供连接池
Spring中提供的数据源，例如DriverManagerDataSource
**对每个连接请求建立新的连接，程序使用完毕后销毁**，交互频繁时明显存在大量时空开销
建议仅在测试时使用
### 提供连接池
创建和管理一组连接对象的技术，以Apaceh Jakarta Commons DBCP and C3P0为例
#### Apache连接池技术


## 数据源配置
无论是自动配置还是手动配置，均需要在pom.xml添加相关依赖
eg:
```xml
<dependencies>
	<!--添加MySQL依赖-->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
	</dependency>
	<!--添加JDBC依赖-->
	<dependency>
		<groupId>org,springframework.boot</groupId>
		<artifactId>spring-boot-starter-jdbc</artifactId>
	</dependency>
</dependencies>
```
### SpringBoot自动配置
#### 原理
默认选用HikariCP
~~如不可用，如果Tomcat池化DataSource可用选用它~~
~~如果以上都不可用， Commons DBCP2可用选用它？~~
#### 示例
在`application.propertites`文件中添加以下配置
```java
//
spring.datasource.url=jdbc:mysql://localhost/test
//
spring.datasource.username=...
//
spring.datasource.paddword=...
//数据库驱动程序类名，该驱动被废弃，选用com.mysql.cj.jdbc.Drvier，该驱动自动加载
//所以无需再propertis指定驱动
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
//配置数据源类型，默认情况下使用HikariCP，其他则需要在pom.xml文件中添加相应的依赖项
spring.datasource.type=....
//配置连接池大小
spring.datasource.hikari.maximum-pool-size=
//

//
```
或者`application.yml`文件中添加如下配置
```yml
spring:
	datasource:
		url:
		driverClassName:
		username:
		password
```
### 手动配置
#### 原理
在IoC容器初始化时实例化该数据源

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
### 多数据源配置
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

### 切换默认数据源
#### 保留数据源依赖
  在引入spring-boot-starter-jdbc依赖时，包含了Tomcat-JDBC依赖，因此切换时，需要排除该依赖，再填上需要的数据源依赖
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
	<exclusions>
		<!--排除Tomcat-JDBC依赖-->
		<exclusion>
			<groupId>org.apache.tomcat</groupId>
			<artifactId>tomcat-jdbc</artifactId>
		</exclusion>
	<exclusions>
</dependency>
<!--添加依赖-->
<dependency>
	<groupId> com.zaxxer</groupId>
	<artifactId> HikariCP</artifactId>
</dependency>
```
#### spring.datasource.type
  在核心配置(application.properties)指定数据源类型
```
spring.datasource.type = com.zaxxer.hikari.HikariDataSource
# spring.datasource.type = org.apache.tomcat.jdbc.pool.DataSource
# spring.datasource.type = org.apache.commons.dbcp.BasicDataSource
# spring.datasource.type = org.apache.commons.dbcp2.BasicDataSource
```

## 问题


## 参考文献
[1]https://cloud.tecent.com/developer/article/2258704
[2]https://juejin.cn/post/6844903654223265800#heading-2
