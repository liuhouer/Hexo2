

title: "Spring配置数据源的几种形式"
date: 2016-09-20   23:46:49
tags: [java]
categories: java
---

# Spring配置数据源的几种形式

> 由于我的网站之前用的c3p0数据连接池配置，总是引发一些莫名其妙的错误，几次内存泄漏都和这个有关系，google之发现好多人都发现了这些bug.
> 于是了解了一下常见的数据源的配置，并改成了dbcp的配置方案。

Spring中提供了4种不同形式的数据源配置方式：

 - 1、Spring自带的数据源(DriverMangerDataSource);

 - 2、DBCP数据源;

 - 3、C3P0数据源;

 - 4、JNDI数据源。

以上数据源配置需要用的Jar包在http://www.java2s.com/Code/Jar/c/Catalogc.htm中都可以下载到

下面详细介绍这四种数据源配置方式：

## 1. DriverMangerDataSource

> 使用org.springframework.jdbc.datasource.DriverManagerDataSource
> 说明：DriverManagerDataSource建立连接是只要有连接就新建一个connection,根本没有连接池的作用。

XML代码

``` vbscript-html
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource"> 
          <property name="driverClassName"><value>${jdbc.driverClassName}</value></property> 
          <property name="url"><value>${jdbc.url}</value></property> 
          <property name="username"><value>${jdbc.username}</value></property> 
          <property name="password"><value>${jdbc.password}</value></property> 

</bean> 
```

## 2.DBCP数据源

> 使用org.apache.commons.dbcp.BasicDataSource
> 说明:这是一种推荐说明的数据源配置方式，它真正使用了连接池技术
> 
> DBCP的配置依赖于2个jar包commons-dbcp.jar，commons-pool.jar。

XML代码：

``` vbscript-html
<!-- 使用org.apache.commons.dbcp.BasicDataSource   
         说明:这是一种推荐说明的数据源配置方式，它真正使用了连接池技术 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
destroy-method="close">
<property name="driverClassName" value="${jdbc.driver}" />
<property name="url" value="${jdbc.url}" />
<property name="username" value="${jdbc.username}" />
<property name="password" value="${jdbc.password}" />
<!--maxActive: 最大连接数量 -->
<property name="maxActive" value="150" />
<!--minIdle: 最小空闲连接 -->
<property name="minIdle" value="5" />
<!--maxIdle: 最大空闲连接 -->
<property name="maxIdle" value="20" />
<!--initialSize: 初始化连接 -->
<property name="initialSize" value="30" />
<!-- 连接被泄露时是否打印 -->
<property name="logAbandoned" value="true" />
<!--removeAbandoned: 是否自动回收超时连接 -->
<property name="removeAbandoned" value="true" />
<!--removeAbandonedTimeout: 超时时间(以秒数为单位) -->
<property name="removeAbandonedTimeout" value="10" />
<!--maxWait: 超时等待时间以毫秒为单位 1000等于60秒 -->
<property name="maxWait" value="1000" />
<!-- 在空闲连接回收器线程运行期间休眠的时间值,以毫秒为单位. -->
<property name="timeBetweenEvictionRunsMillis" value="10000" />
<!-- 在每次空闲连接回收器线程(如果有)运行时检查的连接数量 -->
<property name="numTestsPerEvictionRun" value="10" />
<!-- 1000 * 60 * 30 连接在池中保持空闲而不被空闲连接回收器线程 -->
<property name="minEvictableIdleTimeMillis" value="10000" />
<property name="validationQuery" value="SELECT NOW() FROM DUAL" />
</bean>
```


上面代码的解释：

> BasicDataSource提供了close()方法关闭数据源，所以必须设定destroy-method=”close”属性，
> 以便Spring容器关闭时，数据源能够正常关闭。除以上必须的数据源属性外，还有一些常用的属性：
> 
> defaultAutoCommit：设置从数据源中返回的连接是否采用自动提交机制，默认值为 true；
> defaultReadOnly：设置数据源是否仅能执行只读操作， 默认值为 false；
> maxActive：最大连接数据库连接数，设置为0时，表示没有限制； maxIdle：最大等待连接中的数量，设置为0时，表示没有限制；
> maxWait：最大等待秒数，单位为毫秒， 超过时间会报出错误信息；
> validationQuery：用于验证连接是否成功的查询SQL语句，SQL语句必须至少要返回一行数据，
> 如你可以简单地设置为：“select count(*) from user”； removeAbandoned：是否自我中断，默认是
> false ；
> removeAbandonedTimeout：几秒后数据连接会自动断开，在removeAbandoned为true，提供该值；
> logAbandoned：是否记录中断事件， 默认为 false；


<!--more-->

## 3.C3P0数据源

> C3P0是一个开放源代码的JDBC数据源实现项目，C3P0依赖于jar包c3p0.jar。

XML代码：

``` vbscript-html
<!-- 配置数据源 c3p0 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
destroy-method="close">
<property name="driverClass" value="${jdbc.driver}" />
<property name="jdbcUrl" value="${jdbc.url}" />
<property name="user" value="${jdbc.username}" />
<property name="password" value="${jdbc.password}" />

<!-- 请求超时时间 -->
<!--当连接池用完时客户端调用getConnection()后等待获取新连接的时间，超时后将抛出
SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
<property name="checkoutTimeout" value="5000" />
<!-- 每60秒检查所有连接池中的空闲连接。默认值: 0，不检查 -->
<property name="idleConnectionTestPeriod" value="60" />
<!-- 连接数据库连接池最大空闲时间 -->
<property name="maxIdleTime" value="30" />
<!-- 连接池初始化连接数 -->
<property name="initialPoolSize" value="5" />
<property name="minPoolSize" value="5" />
<property name="maxPoolSize" value="20" />
<!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。默认值: 3 -->
<property name="acquireIncrement" value="5" />
<!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
<property name="acquireRetryAttempts" value="10" />
<!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效
保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试
获取连接失败后该数据源将申明已断开并永久关闭。Default: false-->
<property name="breakAfterAcquireFailure" value="true" />
<!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的
时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable
等方法来提升连接测试的性能。Default: false -->
<property name="testConnectionOnCheckout" value="false" />
</bean>
```


## 4.JNDI数据源

> 使用org.springframework.jndi.JndiObjectFactoryBean
> 说明:JndiObjectFactoryBean 能够通过JNDI获取DataSource
> 如果应用配置在高性能的应用服务器（如WebLogic或Websphere,tomcat等）上，我们可能更希望使用应用服务器本身提供的数据源。应用服务器的数据源
> 使用JNDI开放调用者使用，Spring为此专门提供引用JNDI资源的JndiObjectFactoryBean类。

xml 代码：

``` nix
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">

<property name="jndiName" value="java:comp/env/jdbc/orclight"/>        

</bean>

<beans xmlns=http://www.springframework.org/schema/beans

xmlns:xsi=http://www.w3.org/2001/XMLSchema-instance

xmlns:jee=http://www.springframework.org/schema/jee

xsi:schemaLocation="http://www.springframework.org/schema/beans

http://www.springframework.org/schema/beans/spring-beans-2.0.xsd

http://www.springframework.org/schema/jee

http://www.springframework.org/schema/jee/spring-jee-2.0.xsd">

<jee:jndi-lookup id="dataSource" jndi-name=" java:comp/env/jdbc/orclight"/>

</beans\>
```
