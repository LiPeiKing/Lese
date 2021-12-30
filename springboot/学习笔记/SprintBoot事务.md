# Spring Boot之事务

## 1、事务定义

事务，就是一组操作数据库的动作集合。事务是现代数据库理论中的核心概念之一。如果一组处理步骤或者全部发生或者一步也不执行，我们称该组处理步骤为一个事务。当所有的步骤像一个操作一样被完整地执行，我们称该事务被提交。由于其中的一部分或多步执行失败，导致没有步骤被提交，则事务必须回滚到最初的系统状态。

## 2、@Transactional 事务实现机制

 Spring 为事务管理提供了丰富的功能支持。Spring 事务管理分为编码式和声明式的两种方式。

   （1）编程式事务指的是通过编码方式实现事务；

   （2）声明式事务基于 AOP,将具体业务逻辑与事务处理解耦。

​    声明式事务管理使业务代码逻辑不受污染, 因此在实际使用中声明式事务用的比较多。

   声明式事务有两种方式：

​    （1）一种是在配置文件（xml）中做相关的事务规则声明，

​    （2）另一种是基于@Transactional 注解的方式。

   注释配置是目前流行的使用方式，因此本文将着重介绍基于@Transactional 注解的事务管理。

 在应用系统调用声明了 @Transactional 的目标方法时，Spring Framework 默认使用 AOP 代理，在代码运行时生成一个代理对象，根据 @Transactional 的属性配置信息，这个代理对象决定该声明 @Transactional 的目标方法是否由拦截器 TransactionInterceptor 来使用拦截，在 TransactionInterceptor 拦截时，会在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器 AbstractPlatformTransactionManager 操作数据源 DataSource 提交或回滚事务。

   Spring AOP 代理有 CglibAopProxy 和 JdkDynamicAopProxy 两种，以 CglibAopProxy 为例，对于 CglibAopProxy，需要调用其内部类的 DynamicAdvisedInterceptor 的 intercept 方法。对于 JdkDynamicAopProxy，需要调用其 invoke 方法。


![img](https://img-blog.csdnimg.cn/20191104135149638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbmd5YW5neWU=,size_16,color_FFFFFF,t_70)

**开启事务**

方式1：在 xml 配置中的事务配置信息

```xml
<tx:annotation-driven />
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource" />
</bean>
```

方式2：使用@EnableTransactionManagement 注解也可以启用事务管理功能

## 3、事务传播机制

事务的传播行为是针对嵌套事务而言。

**示例：**

```java
@Transactional(propagation = Propagation.REQUIRED)
```

有六种类型的事务传播：

- REQUIRED （`默认`）

    如果调用者的服务中有一个存在的事务，那么就使用这个存在的，如果调用者中没有事务上下文，则创建一个新的事务。

- SUPPORTS

    如果调用者的服务中有一个存在的事务，那么就使用这个存在的，如果调用者中没有事务上下文，**则不会创建一个新的事务。**

-  NOT_SUPPORTED

    无论调用者是否有事务，都不会创建或参与任何一个事务。

- REQUIRES_NEW

    无论调用者是否有事务，总是会创建自己的一个事务。

- NEVER

    如果调用者有事务，会抛出exception，如果调用者没有事务，也不会创建事务，在没有事务环境下运行。

- MANDATORY

    如果调用者的服务中有一个存在的事务，那么就使用这个存在的，如果调用者中没有事务上下文，会抛出exception。

### 3.1、REQUIRED

Spring默认的事务传播行为就是它。

支持事务。如果业务方法执行时已经在一个事务中，则加入当前事务，否则重新开启一个事务。

#### 3.1.1、抛出RuntimeException异常

​	(1) 内层抛出RuntimeException

​		内外层一起回滚

​	(2) 外层抛出RuntimeException

​	内外层一起回滚

​	(3) 内、外层都抛出RuntimeException

​	内外层一起回滚

#### 3.1.2、抛出Exception异常

- @Transactional

(1) 内层抛出Exception，外层直接抛出

​	外层回滚，内层不回滚

(2) 外层抛出Exception，内层不抛出异常

​	内外层均不回滚

(3) 内层抛出Exception，外层捕获异常后不抛出，外层不抛出异常

​	内外层均不回滚



- @Transactional(rollbackFor = {Exception.class})

(1) 内层抛出Exception，外层直接抛出

​	内外层一起回滚

(2) 外层抛出Exception，内层不抛出异常

​	内外层一起回滚

(3) 内层抛出Exception，外层捕获异常后不抛出，外层不抛出异常

​	内外层均不回滚

**`总结`**

@Transactional(rollbackFor = {Exception.class}当)抛出Exception异常事和@Transactional抛出RuntimeException异常情况相同，异常捕获后不抛出不会回滚。



### 3.2、REQUIRED_NEW

基本同REQUIRED



## 4、事务的隔离机制

1.DEFAULT ,这是spring默认的隔离级别，表示使用数据库默认的事务隔离级别。另外四个与JDBC的隔离级别相对应。

2.READ_UNCOMMITTED 这是事务最低的隔离级别，它充许别外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻读。

3.READ_COMMITTED 保证一个事务修改的数据提交后才能被另外一个事务读取。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻读。

4.REPEATABLE_READ这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻读。

5.SERIALIZABLE 事务被处理为顺序执行。防止脏读，不可重复读，防止幻读。























































