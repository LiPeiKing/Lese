# Spring Data Jpa使用手册

[TOC]

# 基础知识

## Spring Data JPA 初识

JPA 是 JDK 5.0 新增的协议，通过相关持久层注解（@Entity 里面的各种注解）来描述对象和关系型数据里面的表映射关系，并将 Java 项目运行期的实体对象，通过一种Session持久化到数据库中。

JPA 的宗旨是为 POJO 提供持久化标准规范，可以集成在 Spring 的全家桶使用，也可以直接写独立 application 使用，任何用到 DB 操作的场景，都可以使用，极大地方便开发和测试，所以 JPA 的理念已经深入人心了。Spring Data JPA、Hibernate 3.2+、TopLink 10.1.3 以及 OpenJPA、QueryDSL 都是实现 JPA 协议的框架，他们之间的关系结构如下图所示：

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20201231164749.jpg)

Spring Data 项目是从 2010 年开发发展起来的，Spring Data 利用一个大家熟悉的、一致的、基于“注解”的数据访问编程模型，做一些公共操作的封装，它可以轻松地让开发者使用数据库访问技术，包括关系数据库、非关系数据库（NoSQL）。同时又有不同的数据框架的实现，保留了每个底层数据存储结构的特殊特性。

Spring Data Common 是 Spring Data 所有模块的公共部分，该项目提供了基于 Spring 的共享基础设施，它提供了基于 repository 接口以 DB 操作的一些封装，以及一个坚持在 Java 实体类上标注元数据的模型。

Spring Data 不仅对传统的数据库访问技术如 JDBC、Hibernate、JDO、TopLick、JPA、MyBatis 做了很好的支持和扩展、抽象、提供方便的操作方法，还对 MongoDB、KeyValue、Redis、LDAP、Cassandra 等非关系数据的 NoSQL 做了不同的实现版本，方便我们开发者触类旁通。

下图为目前 Spring Data 的框架分类结构图，里面都有哪些模块可以一目了然，也可以知道哪些是我们需要关心的项目。

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20201231164802.jpg)

## Repository接口

Repository 是 Spring Data Common 里面的顶级父类接口，里面什么方法都没有，但是如果任何接口继承它，就能得到一个 Repository，还可以实现 JPA 的一些默认实现方法。Spring 利用 Repository 作为 DAO 操作的 Type，以及利用 Java 动态代理机制就可以实现很多功能。

```java
package org.springframework.data.repository;
import org.springframework.stereotype.Indexed;
@Indexed
public interface Repository<T, ID> {
}
```

Spring 在做动态代理的时候，只要是它的子类或者实现类，再利用 T 类以及 T 类的 主键 ID 类型作为泛型的类型参数，就可以来标记出来、并捕获到要使用的实体类型，就能帮助使用者进行数据库操作。

Repository 分为以下 4 个大类：

- `ReactiveCrudRepository` ：响应式编程，主要支持当前 NoSQL 方面的操作，因为这方面大部分操作都是分布式的，所以由此我们可以看出 Spring Data 想统一数据操作的“野心”，即想提供关于所有 Data 方面的操作。目前 Reactive 主要有 Cassandra、MongoDB、Redis 的实现。
- `RxJava2CrudRepository` ：为了支持 RxJava 2 做的标准响应式编程的接口。
- `CoroutineCrudRepository` ：为了支持 Kotlin 语法而实现的。
- `CrudRepository` ：JPA 相关的操作接口，也是我们主要用到的接口。

更详细一点，我们需要掌握和使用到的**7 大 Repository 接口**如下所示：

- `Repository(org.springframework.data.repository)`，没有暴露任何方法；
- `CrudRepository(org.springframework.data.repository)`，简单的 Curd 方法；
- `PagingAndSortingRepository(org.springframework.data.repository)`，带分页和排序的方法；
- `QueryByExampleExecutor(org.springframework.data.repository.query)`，简单 Example 查询；
- `JpaRepository(org.springframework.data.jpa.repository)`，JPA 的扩展方法；
- `JpaSpecificationExecutor(org.springframework.data.jpa.repository)`，JpaSpecification 扩展查询；
- `QueryDslPredicateExecutor(org.springframework.data.querydsl)`，QueryDsl 的封装。

**两大 Repository 实现类：**

- `SimpleJpaRepository(org.springframework.data.jpa.repository.support)`，JPA 所有接口的默认实现类；
- `QueryDslJpaRepository(org.springframework.data.jpa.repository.support)`，QueryDsl 的实现类。

## Defining Query Methods

Spring Data JPA 的最大特色是利用**方法名定义查询方法**（Defining Query Methods）来做 CRUD 操作。

DQM 语法共有 2 种，具体如下：

- 一种是直接通过方法名就可以实现；
- 另一种是 @Query 手动在方法上定义。

### 定义查询方法的配置和使用方法

若想要实现 CRUD 的操作，常规做法是写一大堆 SQL 语句。但在 JPA 里面，只需要继承 Spring Data Common 里面的任意 Repository 接口或者子接口，然后直接通过方法名就可以实现：

```java
interface UserRepository extends CrudRepository<User, Long> {
    User findByEmailAddress(String emailAddress);
}
```

### 方法的查询策略设置

目前在实际生产中还没有遇到要修改默认策略的情况，但我们必须要知道有这样的配置方法，做到心中有数，这样我们才能知道为什么方法名可以，@Query 也可以。通过 `@EnableJpaRepositories` 注解来配置方法的查询策略，详细配置方法如下：

```java
@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)
```

其中，QueryLookupStrategy.Key 的值共 3 个，具体如下：

- **Create**：直接根据方法名进行创建，规则是根据方法名称的构造进行尝试，一般的方法是从方法名中删除给定的一组已知前缀，并解析该方法的其余部分。如果方法名不符合规则，启动的时候会报异常，这种情况可以理解为，即使配置了 @Query 也是没有用的。
- **USE_DECLARED_QUERY**：声明方式创建，启动的时候会尝试找到一个声明的查询，如果没有找到将抛出一个异常，可以理解为必须配置 @Query。
- **CREATE_IF_NOT_FOUND**：这个是默认的，除非有特殊需求，可以理解为这是以上 2 种方式的兼容版。先用声明方式（@Query）进行查找，如果没有找到与方法相匹配的查询，那用 Create 的方法名创建规则创建一个查询；这两者都不满足的情况下，启动就会报错。

以 Spring Boot 项目为例，更改其配置方法如下：

```java
@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)
public class Example1Application {
    public static void main(String[] args) {
        SpringApplication.run(Example1Application.class, args);
    }
}
```

### Defining Query Method（DQM）语法

该语法是：带查询功能的方法名由查询策略（关键字）+ 查询字段 + 一些限制性条件组成，具有语义清晰、功能完整的特性，我们实际工作中 80% 的 API 查询都可以简单实现。

下面表格是一个我们在上面 DQM 方法语法里常用的关键字列表：

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20201231164824.jpg)

综上，总结 3 点经验：

- 方法名的表达式通常是实体属性连接运算符的组合，如 And、or、Between、LessThan、GreaterThan、Like 等属性连接运算表达式，不同的数据库（NoSQL、MySQL）可能产生的效果不一样，如果遇到问题，我们可以打开 SQL 日志观察。
- IgnoreCase 可以针对单个属性（如 `findByLastnameIgnoreCase(…)`），也可以针对查询条件里面所有的实体属性忽略大小写（所有属性必须在 String 情况下，如 `findByLastnameAndFirstnameAllIgnoreCase(…)`）。
- OrderBy 可以在某些属性的排序上提供方向（Asc 或 Desc），称为静态排序，也可以通过一个方便的参数 Sort 实现指定字段的动态排序的查询方法（如 `repository.findAll(Sort.by(Sort.Direction.ASC, "myField"))`）。

我们看到上面的表格虽然大多是 find 开头的方法，除此之外，JPA 还支持read、get、query、stream、count、exists、delete、remove等前缀，如字面意思一样。实例代码如下：

```java
interface UserRepository extends CrudRepository<User, Long> {
    long countByLastname(String lastname);//查询总数
    long deleteByLastname(String lastname);//根据一个字段进行删除操作，并返回删除行数
    List<User> removeByLastname(String lastname);//根据Lastname删除一堆User,并返回删除的User
}
```

有的时候随着版本的更新，也会有更多的语法支持，或者不同的版本语法可能也不一样，我们通过源码来看一下上面说的几种语法。感兴趣的同学可以到类 `org.springframework.data.repository.query.parser.PartTree` 查看相关源码的逻辑和处理方法，关键源码如下：

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20201231164840.png)

### Sort 排序和 Pageable 分页

Spring Data JPA 为了方便我们排序和分页，支持了两个特殊类型的参数：Sort 和 Pageable。

Pageable 是一个接口，里面有常见的分页方法排序、当前页、下一行、当前指针、一共多少页、页码、pageSize 等。

在查询方法中如何使用 Pageable 和 Sort 呢？下面代码定义了根据 Lastname 查询 User 的分页和排序的实例，此段代码是在 UserRepository 接口里面定义的方法：

```java
Page<User> findByLastname(String lastname, Pageable pageable);//根据分页参数查询User，返回一个带分页结果的Page对象（方法一）

Slice<User> findByLastname(String lastname, Pageable pageable);//我们根据分页参数返回一个Slice的user结果（方法二）

List<User> findByLastname(String lastname, Sort sort);//根据排序结果返回一个List（方法三）

List<User> findByLastname(String lastname, Pageable pageable);//根据分页参数返回一个List对象（方法四）
```

我们可以通过 PageRequest 里面提供的几个 of 静态方法（多态），分别构建页码、页面大小、排序等。

```java
//查询user里面的lastname=jk的第一页，每页大小是20条；并会返回一共有多少页的信息

Page<User> users = userRepository.findByLastname("jk",PageRequest.of(1, 20));

//查询user里面的lastname=jk的第一页的20条数据，不知道一共多少条

Slice<User> users = userRepository.findByLastname("jk",PageRequest.of(1, 20));

//查询出来所有的user里面的lastname=jk的User数据，并按照name正序返回List

List<User> users = userRepository.findByLastname("jk",new Sort(Sort.Direction.ASC, "name"))

//按照createdAt倒序，查询前一百条User数据

List<User> users = userRepository.findByLastname("jk",PageRequest.of(0, 100, Sort.Direction.DESC, "createdAt"));
```

### 限制查询结果 First 和 Top

有的时候我们想直接查询前几条数据，也不需要动态排序，那么就可以简单地在方法名字中使用 First 和 Top 关键字，来限制返回条数。

我们来看看 userRepository 里面可以定义的一些限制返回结果的使用。在查询方法上加限制查询结果的关键字 First 和 Top。

```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

List<User> findDistinctUserTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

其中：

- 查询方法在使用 First 或 Top 时，数值可以追加到 First 或 Top 后面，指定返回最大结果的大小；
- 如果数字被省略，则假设结果大小为 1；
- 限制表达式也支持 Distinct 关键字；
- 支持将结果包装到 Optional 中（下一课时详解）。
- 如果将 Pageable 作为参数，以 Top 和 First 后面的数字为准，即分页将在限制结果中应用。

### [@NonNull](https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNull.html)、[@NonNullApi](https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html)、[@Nullable](https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/Nullable.html)

从 Spring Data 2.0 开始，JPA 新增了[@NonNull](https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNull.html) [@NonNullApi](https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html) [@Nullable](https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/Nullable.html)，是对 null 的参数和返回结果做的支持。

- **@NonNullApi**：在包级别用于声明参数，以及返回值的默认行为是不接受或产生空值的。
- **@NonNull**：用于不能为空的参数或返回值（在 @NonNullApi 适用的参数和返回值上不需要）。
- **@Nullable**：用于可以为空的参数或返回值。

## Repository 中的方法返回值

Repository 的返回类型包括：Optional、Iterable、List、Page、Long、Boolean、Entity 对象等，而实际上支持的返回类型还要多一些。

由于 Repository 里面支持 Iterable，所以其实 java 标准的 List、Set 都可以作为返回结果，并且也会支持其子类，Spring Data 里面定义了一个特殊的子类 Steamable，Streamable 可以替代 Iterable 或任何集合类型。它还提供了方便的方法来访问 Stream，可以直接在元素上进行 ….filter(…) 和 ….map(…) 操作，并将 Streamable 连接到其他元素。

```java
User user = userRepository.save(User.builder().name("jackxx").email("123456@126.com").sex("man").address("shanghai").build());
Assert.assertNotNull(user);
Streamable<User> userStreamable = userRepository.findAll(PageRequest.of(0,10)).and(User.builder().name("jack222").build());
userStreamable.forEach(System.out::println);
```

### 返回结果类型 List/Stream/Page/Slice

```java
public interface UserRepository extends JpaRepository<User,Long> {

   //自定义一个查询方法，返回Stream对象，并且有分页属性
    @Query("select u from User u")
    Stream<User> findAllByCustomQueryAndStream(Pageable pageable);

    //测试Slice的返回结果
    @Query("select u from User u")
    Slice<User> findAllByCustomQueryAndSlice(Pageable pageable);
}
```

### 异步返回结果Feature/CompletableFuture

我们可以使用 Spring 的异步方法执行Repository查询，这意味着方法将在调用时立即返回，并且实际的查询执行将发生在已提交给 Spring TaskExecutor 的任务中，比较适合定时任务的实际场景。异步使用起来比较简单，直接加@Async 注解即可，如下所示：

```java
@Async
Future<User> findByFirstname(String firstname); (1)

@Async
CompletableFuture<User> findOneByFirstname(String firstname); (2)

@Async
ListenableFuture<User> findOneByLastname(String lastname);(3)
```

关于实际使用需要注意以下三点内容：

- 在实际工作中，直接在 Repository 这一层使用异步方法的场景不多，一般都是把异步注解放在 Service 的方法上面，这样的话，可以有一些额外逻辑，如发短信、发邮件、发消息等配合使用；
- 使用异步的时候一定要配置线程池，这点切记，否则“死”得会很难看；
- 万一失败我们会怎么处理？关于事务是怎么处理的呢？这种需要重点考虑的。

下表列出了 Spring Data JPA Query Method 机制支持的方法的返回值类型：

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20201231164853.jpg)

## DTO映射 Projections

Spring JPA 对 Projections 扩展的支持，我个人觉得这是个非常好的东西，从字面意思上理解就是映射，指的是和 DB 的查询结果的字段映射关系。一般情况下，返回的字段和 DB 的查询结果的字段是一一对应的；但有的时候，需要返回一些指定的字段，或者返回一些复合型的字段，而不需要全部返回。

原来我们的做法是自己写各种 entity 到 view 的各种 convert 的转化逻辑，而 Spring Data 正是考虑到了这一点，允许对专用返回类型进行建模，有选择地返回同一个实体的不同视图对象。

下面还以我们的 User 查询对象为例，看看怎么自定义返回 DTO：

```java
@Entity
public class User {
   @Id
   @GeneratedValue(strategy= GenerationType.AUTO)
   private Long id;
   private String name;
   private String email;
   private String sex;
   private String address;
}
```

看上面的原始 User 实体代码，如果我们只想返回 User 对象里面的 name 和 email，应该怎么做？下面我们介绍三种方法。

### 第一种方法：新建一张表的不同 Entity

首先，我们新增一个Entity类：通过 @Table 指向同一张表，这张表和 User 实例里面的表一样都是 user。

```java
@Entity
@Table(name = "user")
public class UserOnlyNameEmailEntity {
   @Id
   @GeneratedValue(strategy= GenerationType.AUTO)
   private Long id;
   private String name;
   private String email;
}
```

然后，新增一个 UserOnlyNameEmailEntityRepository，做单独的查询：

```java
import org.springframework.data.jpa.repository.JpaRepository;
public interface UserOnlyNameEmailEntityRepository extends JpaRepository<UserOnlyNameEmailEntity,Long> {
}
```

这种方式的好处是简单、方便，很容易可以想到；缺点就是通过两个实体都可以进行 update 操作，如果同一个项目里面这种实体比较多，到时候就容易不知道是谁更新的，从而导致出 bug 不好查询，实体职责划分不明确。我们来看第二种返回 DTO 的做法。

### 第二种方法：直接定义一个 UserOnlyNameEmailDto

首先，我们新建一个 DTO 类来返回我们想要的字段，它是 UserOnlyNameEmailDto，用来接收 name、email 两个字段的值，具体如下：

```java
public class UserOnlyNameEmailDto {
    private String name;
    private String email;
}
```

其次，在 UserRepository 里面做如下用法：

```java
public interface UserRepository extends JpaRepository<User,Long> {
    //测试只返回name和email的DTO
    UserOnlyNameEmailDto findByEmail(String email);
}
```

所以这种方式的优点就是返回的结果不需要实体对象，对 DB 不能进行除了查询之外的任何操作；缺点就是有 set 方法还可以改变里面的值，构造方法不能更改，必须全参数，这样如果是不熟悉 JPA 的新人操作的时候很容易引发 Bug。

### 第三种方法：返回结果是一个 POJO 的接口

我们再来学习一种返回不同字段的方式，这种方式与上面两种的区别是只需要定义接口，它的好处是只读，不需要添加构造方法，我们使用起来非常灵活，一般很难产生 Bug，那么它怎么实现呢？

首先，定义一个 UserOnlyName 的接口：

```java
package com.example.jpa.example1;
public interface UserOnlyName {
    String getName();
    String getEmail();
}
```

其次，我们的 UserRepository 写法如下：

```java
package com.example.jpa.example1;
import org.springframework.data.jpa.repository.JpaRepository;
public interface UserRepository extends JpaRepository<User,Long> {
    /**
     * 接口的方式返回DTO
     * @param address
     * @return
     */
    UserOnlyName findByAddress(String address);
}
```

这个时候会发现我们的 userOnlyName 接口成了一个代理对象，里面通过 Map 的格式包含了我们的要返回字段的值（如：name、email），我们用的时候直接调用接口里面的方法即可，如 userOnlyName.getName() 即可；这种方式的优点是接口为只读，并且语义更清晰，所以这种是我比较推荐的做法。

## @Query的使用

### @Query 的基本用法

```java
public interface UserRepository extends JpaRepository<User, Long>{
	
    /**
     * JPL写法
     */
    @Query("select u from User u where u.emailAddress = ?1")
    User findByEmailAddress1(String emailAddress);

    /**
     * Naive写法
     */
    @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
    User findByEmailAddress2(String emailAddress);

    /**
     * Native排序写法
     */
    @Query(value = "select * from user_info where first_name=?1 order by ?2",nativeQuery = true)
    List<UserInfoEntity> findByFirstName(String firstName,String sort);

}
```

### @Query 的排序

@Query中在用JPQL的时候，想要实现排序，方法上直接用 PageRequest 或者 Sort 参数都可以做到。

在排序实例中，实际使用的属性需要与实体模型里面的字段相匹配，这意味着它们需要解析为查询中使用的属性或别名。我们看一下例子，这是一个`state_field_path_expression JPQL`的定义，并且 Sort 的对象支持一些特定的函数。

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("select u from User u where u.lastname like ?1%")
    List<User> findByAndSort(String lastname, Sort sort);

    @Query("select u.id, LENGTH(u.firstname) as fn_len from User u where u.lastname like ?1%")
    List<Object[]> findByAsArrayAndSort(String lastname, Sort sort);
}
```

### @Query 的分页

@Query 的分页分为两种情况，分别为 JQPl 的排序和 nativeQuery 的排序。看下面的案例。

直接用 Page 对象接受接口，参数直接用 Pageable 的实现类即可。

```java
public interface UserRepository extends JpaRepository<User, Long> {
    @Query(value = "select u from User u where u.lastname = ?1")
    Page<User> findByLastname(String lastname, Pageable pageable);
}
//调用者的写法
repository.findByFirstName("jackzhang",new PageRequest(1,10));
```

@Query 对原生 SQL 的分页支持，并不是特别友好，因为这种写法比较“骇客”，可能随着版本的不同会有所变化。我们以 MySQL 为例。

```java
public interface UserRepository extends JpaRepository<UserInfoEntity, Integer>, JpaSpecificationExecutor<UserInfoEntity> {
    @Query(value = "select * from user_info where first_name=?1 /* #pageable# */",
           countQuery = "select count(*) from user_info where first_name=?1",
           nativeQuery = true)
    Page<UserInfoEntity> findByFirstName(String firstName, Pageable pageable);
}
//调用者的写法
return userRepository.findByFirstName("jackzhang",new PageRequest(1,10, Sort.Direction.DESC,"last_name"));
//打印出来的sql
select  *   from  user_info  where  first_name=? /* #pageable# */  order by  last_name desc limit ?, ?
```

**这里需要注意：这个注释 /\* #pageable# \*/ 必须有。**

另外，随着版本的变化，这个方法有可能会进行优化。此外还有一种实现方法，就是自己写两个查询方法，自己手动分页。

### @Param 用法

@Param 注解指定方法参数的具体名称，通过绑定的参数名字指定查询条件，这样不需要关心参数的顺序。**我比较推荐这种做法，因为它比较利于代码重构**。如果不用 @Param 也是可以的，参数是有序的，这使得查询方法对参数位置的重构容易出错。我们看个案例。

根据 firstname 和 lastname 参数查询 user 对象。

```java
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
    User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                   @Param("firstname") String firstname);
}
```

### @Query整合DTO映射 Projections

首先，新增一个 UserSimpleDto 接口来得到我们想要的 name、email、idCard 信息。

```java
package com.example.jpa.example1;
public interface UserSimpleDto {
    String getName();
    String getEmail();
    String getIdCard();
}
```

其次，在 UserDtoRepository 里面新增一个方法，返回结果是 UserSimpleDto 接口。

```java
public interface UserDtoRepository extends JpaRepository<User, Long> {
    //利用接口DTO获得返回结果，需要注意的是每个字段需要as和接口里面的get方法名字保持一样
    @Query("select CONCAT(u.name,'JK123') as name,UPPER(u.email) as email ,e.idCard as idCard from User u,UserExtend e where u.id= e.userId and u.id=:id")
    UserSimpleDto findByUserSimpleDtoId(@Param("id") Long id);
}
```

比起 DTO 我们不需要 new 了，并且接口只能读，那么我们返回的结果 DTO 的职责就更单一了，只用来查询。

**接口的方式是我比较推荐的做法，因为它是只读的，对构造方法没有要求，返回的实际是 HashMap。**

### @Query 动态查询解决方法

我们看一个例子，来了解一下如何实现 @Query 的动态参数查询。

首先，新增一个 UserOnlyName 接口，只查询 User 里面的 name 和 email 字段。

```java
package com.example.jpa.example1;
//获得返回结果
public interface UserOnlyName {
    String getName();
    String getEmail();
}
```

其次，在我们的 UserDtoRepository 里面新增两个方法：一个是利用 JPQL 实现动态查询，一个是利用原始 SQL 实现动态查询。

```java
package com.example.jpa.example1;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;
public interface UserDtoRepository extends JpaRepository<User, Long> {
    /**
    * 利用JQPl动态查询用户信息
    * @param name
    * @param email
    * @return UserSimpleDto接口
    */
    @Query("select u.name as name,u.email as email from User u where (:name is null or u.name =:name) and (:email is null or u.email =:email)")
    UserOnlyName findByUser(@Param("name") String name,@Param("email") String email);
    /**
    * 利用原始sql动态查询用户信息
    * @param user
    * @return
    */
    @Query(value = "select u.name as name,u.email as email from user u where (:#{#user.name} is null or u.name =:#{#user.name}) and (:#{#user.email} is null or u.email =:#{#user.email})",nativeQuery = true)
    UserOnlyName findByUser(@Param("user") User user);
}
```

> 我们知道定义方法名可以获得想要的结果，@Query 注解亦可以获得想要的结果，nativeQuery 也可以获得想要的结果，那么我们该如何做选择呢？**下面我从个人经验中总结了一些观点。**
>
> 1. 能用方法名表示的，尽量用方法名表示，因为这样语义清晰、简单快速，基本上只要编译通过，一定不会有问题；
> 2. 能用 @Query 里面的 JPQL 表示的，就用 JPQL，这样与 SQL 无关，万一哪天换数据库了，基本上代码不用改变；
> 3. 最后实在没有办法了，可以选择 nativeQuery 写原始 SQL，特别是一开始从 MyBatis 转过来的同学，选择写 SQL 会更容易一些。

## JPA的注解

JPA 协议里面关于实体有如下定义，（这里推荐一个查看 JPA 协议的官方地址：https://download.oracle.com/otn-pub/jcp/persistence-2_2-mrel-spec/JavaPersistence.pdf）：

- 实体是直接进行数据库持久化操作的领域对象（即一个简单的 POJO，可以按照业务领域划分），必须通过 @Entity 注解进行标示。
- 实体必须有一个 public 或者 protected 的无参数构造方法。
- Entity 里面的注解生效只有两种方式：将注解写在字段上或者将注解写在方法上（JPA 里面称 Property）。

需要注意的是，在同一个 Entity 里面只能有一种方式生效，也就是说，注解要么全部写在 field 上面，要么就全部写在 Property 上面。

JPA 里面支持的注解大概有一百多个，这里只提及一些最常见的，包括 @Entity、@Table、@Access、@Id、@GeneratedValue、@Enumerated、@Basic、@Column、@Transient、@Lob、@Temporal 等。

### @Entity

用于定义对象将会成为被 JPA 管理的实体，必填，将字段映射到指定的数据库表中，使用起来很简单，直接用在实体类上面即可，通过源码表达的语法如下：

```java
@Target(TYPE) //表示此注解只能用在class上面
public @interface Entity {
    //可选，默认是实体类的名字，整个应用里面全局唯一。
    String name() default "";
}
```

### @Table

用于指定数据库的表名，表示此实体对应的数据库里面的表名，非必填，默认表名和 entity 名字一样。

```java
@Target(TYPE) //一样只能用在类上面
public @interface Table {
    //表的名字，可选。如果不填写，系统认为好实体的名字一样为表名。
    String name() default "";
    //此表所在schema，可选
    String schema() default "";
    //唯一性约束，在创建表的时候有用，表创建之后后面就不需要了。
    UniqueConstraint[] uniqueConstraints() default { };
    //索引，在创建表的时候使用，表创建之后后面就不需要了。
    Index[] indexes() default {};
}
```

### @Access

用于指定 entity 里面的注解是写在字段上面，还是 get/set 方法上面生效，非必填。在默认不填写的情况下，当实体里面的第一个注解出现在字段上或者 get/set 方法上面，就以第一次出现的方式为准。

### @Id

定义属性为数据库的主键，一个实体里面必须有一个主键，但不一定是这个注解，可以和 @GeneratedValue 配合使用或成对出现。

### @GeneratedValue

主键生成策略。

### @Enumerated

这个注解很好用，因为它对 enum 提供了下标和 name 两种方式，用法直接映射在 enum 枚举类型的字段上。

### @Basic

表示属性是到数据库表的字段的映射。如果实体的字段上没有任何注解，默认即为 @Basic。也就是说默认所有的字段肯定是和数据库进行映射的，并且默认为 Eager 类型（EAGER（默认）：立即加载；LAZY：延迟加载。（LAZY主要应用在大字段上面））。

### @Transient

表示该属性并非一个到数据库表的字段的映射，表示非持久化属性。JPA 映射数据库的时候忽略它，与 @Basic 有相反的作用。也就是每个字段上面 @Transient 和 @Basic 必须二选一，而什么都不指定的话，默认是 @Basic。

### @Column

定义该属性对应数据库中的列名。

### @Temporal

用来设置 Date 类型的属性映射到对应精度的字段，存在以下三种情况：

```java
@Temporal(TemporalType.DATE) //映射为日期date （只有日期）
@Temporal(TemporalType.TIME) //映射为日期time （只有时间）
@Temporal(TemporalType.TIMESTAMP) //映射为日期date time （日期+时间）
```

### IDEA生成实体

打开 Persistence 视图，点击 Generate Persistence Mapping>，接着点击选中数据源，然后，选择表和字段，并点击 OK。

这样就可以生成我们想要的实体了。如果是新库、新表，我们也可以先定义好实体，通过实体配置JPA的 `spring.jpa.generate-ddl=true`，反向直接生成 DDL 操作数据库生成表结构。

### 联合主键

在实际的工作中，我们会经常遇到联合主键的情况。可以通过 javax.persistence.EmbeddedId 和 javax.persistence.IdClass 两个注解实现联合主键的效果。

```java
public class UserInfoID implements Serializable {
   private String name,telephone;
}

@Entity
@IdClass(UserInfoID.class)
public class UserInfo {
   private Integer ages;
   @Id
   private String name;
   @Id
   private String telephone;
}

import org.springframework.data.jpa.repository.JpaRepository;
public interface UserInfoRepository extends JpaRepository<UserInfo,UserInfoID> {
}
```

@Embeddable 与 @EmbeddedId 注解同样可以做到联合主键的效果。

```java
@Embeddable
public class UserInfoID implements Serializable {
   private String name,telephone;
}

@Entity
public class UserInfo {
   private Integer ages;
   @EmbeddedId
   private UserInfoID userInfoID;
   @Column(unique = true)
   private String uniqueNumber;
}
```

@IdClass 和 @EmbeddedId 的区别是什么？有以下两个方面：

1. 如上面测试用例，在使用的时候，Embedded 用的是对象，而 IdClass 用的是具体的某一个字段；
2. 二者的JPQL 也会不一样：

① 用 @IdClass JPQL 的写法：`SELECT u.name FROM UserInfo u`

② 用 @EmbeddedId 的 JPQL 的写法：`select u.userInfoId.name FROM UserInfo u`

联合主键还有需要注意的就是，它与唯一性索引约束的区别是写法不同，如上面所讲，唯一性索引的写法如下：

```java
@Column(unique = true)
private String uniqueNumber;
```

## 实体之间的继承关系

> 会使得表与实体之间的关系变得复杂不直观，不建议使用。

在 Java 面向对象的语言环境中，@Entity 之间的关系多种多样，而根据 JPA 的规范，我们大致可以将其分为以下几种：

1. 纯粹的继承，和表没关系，对象之间的字段共享。利用注解 @MappedSuperclass，协议规定父类不能是 @Entity。
2. 单表多态问题，同一张 Table，表示了不同的对象，通过一个字段来进行区分。利用`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`注解完成，只有父类有 @Table。
3. 多表多态，每一个子类一张表，父类的表拥有所有公用字段。通过`@Inheritance(strategy = InheritanceType.JOINED)`注解完成，父类和子类都是表，有公用的字段在父表里面。
4. Object 的继承，数据库里面每一张表是分开的，相互独立不受影响。通过`@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`注解完成，父类（可以是一张表，也可以不是）和子类都是表，相互之间没有关系。

### @Inheritance(strategy = InheritanceType.SINGLE_TABLE)

父类实体对象与各个子实体对象共用一张表，通过一个字段的不同值代表不同的对象，我们看一个例子。

### @Inheritance(strategy = InheritanceType.JOINED)

在这种映射策略里面，继承结构中的每一个实体（entity）类都会映射到数据库里一个单独的表中。也就是说，每个实体（entity）都会被映射到数据库中，一个实体（entity）类对应数据库中的一个表。

其中根实体（root entity）对应的表中定义了主键（primary key），所有的子类对应的数据库表都要共同使用 Book 里面的 @ID 这个主键。

### @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)

我们在使用 @MappedSuperClass 主键的时候，如果不指定 @Inhertance，默认就是此种TABLE_PER_CLASS模式。当然了，我们也显示指定，要求继承基类的都是一张表，而父类不是表，是 java 对象的抽象类。

# 高级用法与实战

## JpaSpecificationExecutor

### QueryByExampleExecutor用法

QueryByExampleExecutor（QBE）是一种用户友好的查询技术，具有简单的接口，它允许动态查询创建，并且不需要编写包含字段名称的查询。

下面是一个 UML 图，你可以看到 QueryByExampleExecutor 是 JpaRepository 的父接口，也就是 JpaRespository 里面继承了 QueryByExampleExecutor 的所有方法。

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20201231164947.jpg)

QBE 的基本语法可以分为下述几种。

```java
public interface QueryByExampleExecutor<T> { 
    //根据“实体”查询条件，查找一个对象
    <S extends T> S findOne(Example<S> example);
    //根据“实体”查询条件，查找一批对象
    <S extends T> Iterable<S> findAll(Example<S> example); 
    //根据“实体”查询条件，查找一批对象，可以指定排序参数
    <S extends T> Iterable<S> findAll(Example<S> example, Sort sort);
    //根据“实体”查询条件，查找一批对象，可以指定排序和分页参数 
    <S extends T> Page<S> findAll(Example<S> example, Pageable pageable);
    //根据“实体”查询条件，查找返回符合条件的对象个数
    <S extends T> long count(Example<S> example); 
    //根据“实体”查询条件，判断是否有符合条件的对象
    <S extends T> boolean exists(Example<S> example); 
}
```

这里写一个测试用例，来熟悉一下 QBE 的语法，看一下完整的测试用例的写法：

```java
@DataJpaTest
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class UserAddressRepositoryTest {

    @Autowired
    private UserAddressRepository userAddressRepository;
    @Test

    @Rollback(false)
    public void testQBEFromUserAddress() throws JsonProcessingException {
        User request = User.builder()
            .name("jack").age(20).email("12345")
            .build();
        UserAddress address = UserAddress.builder().address("shang").user(request).build();
        ObjectMapper objectMapper = new ObjectMapper();
        //    System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(address)); //可以打印出来看看参数是什么
        //创建匹配器，即如何使用查询条件
        ExampleMatcher exampleMatcher = ExampleMatcher.matching()
            .withMatcher("user.email", ExampleMatcher.GenericPropertyMatchers.startsWith())
            .withMatcher("address", ExampleMatcher.GenericPropertyMatchers.startsWith());
        Page<UserAddress> u = userAddressRepository.findAll(Example.of(address,exampleMatcher), PageRequest.of(0,2));
System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(u));
    }
}
```

### JpaSpecificationExecutor 使用案例

我们假设一个后台管理页面根据 name 模糊查询、sex 精准查询、age 范围查询、时间区间查询、address 的 in 查询这样一个场景，来查询 user 信息，我们看看这个例子应该怎么写。

```java
@DataJpaTest
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class UserJpeTest {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private UserAddressRepository userAddressRepository;

    private Date now = new Date();

    @Test
    public void testSPE() {
        //模拟请求参数
        User userQuery = User.builder()
            .name("jack")
            .email("123456@126.com")
            .sex(SexEnum.BOY)
            .age(20)
            .addresses(Lists.newArrayList(UserAddress.builder().address("shanghai").build()))
            .build();

        //假设的时间范围参数
        Instant beginCreateDate = Instant.now().plus(-2, ChronoUnit.HOURS);
        Instant endCreateDate = Instant.now().plus(1, ChronoUnit.HOURS);

        //利用Specification进行查询
        Page<User> users = userRepository.findAll(new Specification<User>() {
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                List<Predicate> ps = new ArrayList<Predicate>();
                if (StringUtils.isNotBlank(userQuery.getName())) {
                    //我们模仿一下like查询，根据name模糊查询
                    ps.add(cb.like(root.get("name"),"%" +userQuery.getName()+"%"));
                }

                if (userQuery.getSex()!=null){
                    //equal查询条件，这里需要注意，直接传递的是枚举
                    ps.add(cb.equal(root.get("sex"),userQuery.getSex()));
                }

                if (userQuery.getAge()!=null){
                    //greaterThan大于等于查询条件
                    ps.add(cb.greaterThan(root.get("age"),userQuery.getAge()));

                }

                if (beginCreateDate!=null&&endCreateDate!=null){
                    //根据时间区间去查询创建
                    ps.add(cb.between(root.get("createDate"),beginCreateDate,endCreateDate));

                }

                if (!ObjectUtils.isEmpty(userQuery.getAddresses())) {
                    //联表查询，利用root的join方法，根据关联关系表里面的字段进行查询。
                    ps.add(cb.in(root.join("addresses").get("address")).value(userQuery.getAddresses().stream().map(a->a.getAddress()).collect(Collectors.toList())));

                }
                return query.where(ps.toArray(new Predicate[ps.size()])).getRestriction();
            }
        }, PageRequest.of(0, 2));
        System.out.println(users);
    }

}
```

我们看一下生成的HQL：

```sql
select user0_.id as id1_1_, user0_.age as age2_1_, user0_.create_date as create_d3_1_, user0_.email as email4_1_, user0_.name as name5_1_, user0_.sex as sex6_1_, user0_.update_date as update_d7_1_ from user user0_ inner join user_address addresses1_ on user0_.id=addresses1_.user_id where (user0_.name like ?) and user0_.sex=? and user0_.age>20 and (user0_.create_date between ? and ?) and (addresses1_.address in (?)) limit ?
```

### JpaSpecificationExecutor 实战应用场景

其实JpaSpecificationExecutor 的目的不是让我们做日常的业务查询，而是给我们提供了一种自定义 Query for rest 的架构思路，如果做日常的增删改查，肯定不如我们前面介绍的 Defining Query Methods 和 @Query 方便。

### JpaSpecificationExecutor 解决了哪些问题

1. 我们通过 QueryByExampleExecutor 的使用方法和原理分析，不难发现，JpaSpecificationExecutor 的查询条件 Specification 十分灵活，可以帮我们解决动态查询条件的问题，正如 QueryByExampleExecutor 的用法一样；
2. 它提供的 Criteria API 的使用封装，可以用于动态生成 Query 来满足我们业务中的各种复杂场景；
3. 既然QueryByExampleExecutor 能利用 Specification 封装成框架，我们是不是也可以利用 JpaSpecificationExecutor 封装成框架呢？这样就学会了举一反三。

## JPA 的审计功能

Auditing 是帮我们做审计用的，当我们操作一条记录的时候，需要知道这是谁创建的、什么时间创建的、最后修改人是谁、最后修改时间是什么时候，甚至需要修改记录……这些都是 Spring Data JPA 里面的 Auditing 支持的，它为我们提供了四个注解来完成上面说的一系列事情，如下：

- @CreatedBy 是哪个用户创建的。
- @CreatedDate 创建的时间。
- @LastModifiedBy 最后修改实体的用户。
- @LastModifiedDate 最后一次修改的时间。

利用上面的四个注解实现方法，一共有三种方式实现 Auditing，我们分别看看。

### 第一种方式：直接在实例里面添加上述四个注解

**第一步：在 @Entity：User 里面添加四个注解，并且新增 @EntityListeners(AuditingEntityListener.class) 注解。**

添加完之后，User 的实体代码如下：

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class User implements Serializable {

   @Id
   @GeneratedValue(strategy= GenerationType.AUTO)
   private Long id;

   @CreatedBy
   private Integer createUserId;

   @CreatedDate
   private Date createTime;

   @LastModifiedBy
   private Integer lastModifiedUserId;

   @LastModifiedDate
   private Date lastModifiedTime;
}
```

**第二步：实现 AuditorAware 接口，告诉 JPA 当前的用户是谁。**

我们需要实现 AuditorAware 接口，以及 getCurrentAuditor 方法，并返回一个 Integer 的 user ID。

```java
public class MyAuditorAware implements AuditorAware<Integer> {
    //需要实现AuditorAware接口，返回当前的用户ID
    @Override
    public Optional<Integer> getCurrentAuditor() {
        ServletRequestAttributes servletRequestAttributes =
            (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        Integer userId = (Integer) servletRequestAttributes.getRequest().getSession().getAttribute("userId");
        return Optional.ofNullable(userId);
    }
}
```

这里关键的一步，是实现 AuditorAware 接口的方法，如下所示：

```java
public interface AuditorAware<T> {
    T getCurrentAuditor();
}
```

需要注意的是：这里获得用户 ID 的方法不止这一种，实际工作中，我们可能将当前的 user 信息放在 Session 中，可能把当前信息放在 Redis 中，也可能放在 Spring 的 security 里面管理。

**第三步：通过 @EnableJpaAuditing 注解开启 JPA 的 Auditing 功能。**

第三步是最重要的一步，如果想使上面的配置生效，我们需要开启 JPA 的 Auditing 功能（默认没开启）。这里需要用到的注解是 `@EnableJpaAuditing`。

```java
@Inherited
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(JpaAuditingRegistrar.class)
public @interface EnableJpaAuditing {
    //auditor用户的获取方法，默认是找AuditorAware的实现类；
    String auditorAwareRef() default "";
    //是否在创建修改的时候设置时间，默认是true
    boolean setDates() default true;
    //在创建的时候是否同时作为修改，默认是true
    boolean modifyOnCreate() default true;
    //时间的生成方法，默认是取当前时间(为什么提供这个功能呢？因为测试的时候有可能希望时间保持不变，它提供了一种自定义的方法)；
    String dateTimeProviderRef() default "";
}
@Configuration
@EnableJpaAuditing
public class JpaConfiguration {
   @Bean
   @ConditionalOnMissingBean(name = "myAuditorAware")
   MyAuditorAware myAuditorAware() {
      return new MyAuditorAware();
   }
}
```

### 第二种方式：实体里面实现Auditable 接口

我们改一下上面的 User 实体对象，如下：

```java
@Entity
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@ToString(exclude = "addresses")
@EntityListeners(AuditingEntityListener.class)
public class User implements Auditable<Integer,Long, Instant> {
    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private Long id;
    private String name;
    private String email;
    @Enumerated(EnumType.STRING)
    private SexEnum sex;
    private Integer age;
    @OneToMany(mappedBy = "user")
    @JsonIgnore
    private List<UserAddress> addresses;
    private Boolean deleted;
    private Integer createUserId;
    private Instant createTime;
    private Integer lastModifiedUserId;
    private Instant lastModifiedTime;
    @Override
    public Optional<Integer> getCreatedBy() {
        return Optional.ofNullable(this.createUserId);
    }
    @Override
    public void setCreatedBy(Integer createdBy) {
        this.createUserId = createdBy;
    }
    @Override
    public Optional<Instant> getCreatedDate() {
        return Optional.ofNullable(this.createTime);
    }
    @Override
    public void setCreatedDate(Instant creationDate) {
        this.createTime = creationDate;
    }
    @Override
    public Optional<Integer> getLastModifiedBy() {
        return Optional.ofNullable(this.lastModifiedUserId);
    }
    @Override
    public void setLastModifiedBy(Integer lastModifiedBy) {
        this.lastModifiedUserId = lastModifiedBy;
    }
    @Override
    public void setLastModifiedDate(Instant lastModifiedDate) {
        this.lastModifiedTime = lastModifiedDate;
    }
    @Override
    public Optional<Instant> getLastModifiedDate() {
        return Optional.ofNullable(this.lastModifiedTime);
    }
    @Override
    public boolean isNew() {
        return id==null;
    }
}
```

与第一种方式的差异是，这里我们要去掉上面说的四个注解，并且要实现接口 Auditable 的方法，代码会变得很冗余和啰唆，从代码的复杂程度来看，这种方式我不推荐使用。

### 第三种方式：利用 @MappedSuperclass 注解

**第一步：创建一个 BaseEntity，里面放一些实体的公共字段和注解。**

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseEntity {

   @CreatedBy
   private Integer createUserId;

   @CreatedDate
   private Instant createTime;

   @LastModifiedBy
   private Integer lastModifiedUserId;

   @LastModifiedDate
   private Instant lastModifiedTime;
}
```

> **注意：** BaseEntity里面需要用上面提到的四个注解，并且加上@EntityListeners(AuditingEntityListener.class)，这样所有的子类就不需要加了。

**第二步：实体直接继承 BaseEntity 即可。**

我们修改一下上面的 User 实例继承 BaseEntity，代码如下：

```java
@Entity
public class User extends BaseEntity {

    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private Long id;
    
    private Boolean deleted;
}
```

这样的话，User 实体就不需要关心太多，我们只关注自己需要的逻辑即可。**这种方式，是我最推荐的，也是实际工作中使用最多的一种方式**。它的好处显而易见就是公用性强，代码简单，需要关心的少。

### JPA 的审计功能解决了哪些问题？

1. 可以很容易地让我们写自己的 BaseEntity，把一些公共的字段放在里面，不需要我们关心太多和业务无关的字段，更容易让我们公司的表更加统一和规范。

   实际工作中，BaseEntity 可能还更复杂一点，比如说把 ID 和 @Version 加进去，会变成如下形式：

   ```java
   @Data
   @MappedSuperclass
   @EntityListeners(AuditingEntityListener.class)
   public class BaseEntity {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       @CreatedBy
       private Integer createUserId;
       @CreatedDate
       private Instant createTime;
       @LastModifiedBy
       private Integer lastModifiedUserId;
       @LastModifiedDate
       private Instant lastModifiedTime;
       @Version
       private Integer version;
   }
   ```

2. Auditing 在实战应用场景中，比较适合做后台管理项目，对应纯粹的 RestAPI 项目，提供给用户直接查询的 API 的话，可以考虑一个特殊的 UserID。

### Auditing 的实现原理

JPA的审计功能利用了 Java Persistence API 里面的**@PrePersist**、**@PreUpdate** 回调函数，在更新和创建之前通过`AuditingHandler` 添加了用户信息和时间信息。

## @Entity 的回调方法

JPA 协议里面规定，可以通过一些注解，为其监听回调事件、指定回调方法。下面我整理了一个回调事件注解表，分别列举了 @PrePersist、@PostPersist、@PreRemove、@PostRemove、@PreUpdate、@PostUpdate、@PostLoad注解及其概念。

### 回调事件注解表

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20201231165047.jpg)

### 语法注意事项

关于上表所述的几个方法有一些需要注意的地方，如下：

1. 回调函数都是和 EntityManager.flush 或 EntityManager.commit 在同一个线程里面执行的，只不过调用方法有先后之分，都是同步调用，所以当任何一个回调方法里面发生异常，**都会触发事务进行回滚，而不会触发事务提交**。
2. Callbacks 注解可以放在实体里面，可以放在 super-class 里面，也可以定义在 entity 的 listener 里面，但需要注意的是：放在实体（或者 super-class）里面的方法，签名格式为“void ()”，即没有参数，方法里面操作的是 this 对象自己；放在实体的 EntityListener 里面的方法签名格式为“void (Object)”，也就是方法可以有参数，参数是代表用来接收回调方法的实体。
3. 使上述注解生效的回调方法可以是 public、private、protected、friendly 类型的，但是不能是 static 和 final 类型的方法。

JPA 里面规定的回调方法还有一些，但不常用，就不过多介绍了。接下来，我们看一下回调注解在实体里面是如何使用的。

### JPA Callbacks 的使用方法

#### 第一种用法：在实体和 super-class 中使用

**第一步：修改 BaseEntity，在里面新增回调函数和注解，代码如下:**

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseEntity {

   @Id
   @GeneratedValue(strategy= GenerationType.AUTO)
   private Long id;

// @CreatedBy 这个可能会被 AuditingEntityListener覆盖，为了方便测试，我们先注释掉
   private Integer createUserId;

   @CreatedDate
   private Instant createTime;

   @LastModifiedBy
   private Integer lastModifiedUserId;

   @LastModifiedDate
   private Instant lastModifiedTime;

//  @Version 由于本身有乐观锁机制，这个我们测试的时候先注释掉，改用手动设置的值；
   private Integer version;

   @PreUpdate
   public void preUpdate() {
      System.out.println("preUpdate::"+this.toString());
      this.setCreateUserId(200);
   }

   @PostUpdate
   public void postUpdate() {
      System.out.println("postUpdate::"+this.toString());
   }

   @PreRemove
   public void preRemove() {
      System.out.println("preRemove::"+this.toString());
   }

   @PostRemove
   public void postRemove() {
      System.out.println("postRemove::"+this.toString());
   }

   @PostLoad
   public void postLoad() {
      System.out.println("postLoad::"+this.toString());
   }
}
```

上述代码中，我在类里面使用了@PreUpdate、@PostUpdate、@PreRemove、@PostRemove、@PostLoad 几个注解，并在相应的回调方法里面加了相应的日志。

**第二步：修改一下 User 类，也新增两个回调函数，并且和 BaseEntity 做法一样，代码如下：**

```java
@Entity
public class User extends BaseEntity {// implements Auditable<Integer,Long, Instant> {
    private String name;
    private String email;
    @Enumerated(EnumType.STRING)
    private SexEnum sex;
    private Integer age;
    @OneToMany(mappedBy = "user")
    @JsonIgnore
    private List<UserAddress> addresses;
    private Boolean deleted;
    @PrePersist
    private void prePersist() {
        System.out.println("prePersist::"+this.toString());
        this.setVersion(1);
    }
    @PostPersist
    public void postPersist() {
        System.out.println("postPersist::"+this.toString());
    }
}
```

我在其中使用了 @PrePersist、@PostPersist 回调事件。

然后通过JPA调用进行测试，可以发现响应的回调函数被触发了。

#### 第二种用法：自定义 EntityListener

**第一步：自定义一个 EntityLoggingListener 用来记录操作日志，通过 listener 的方式配置回调函数注解，代码如下:**

```java
@Log4j2
public class EntityLoggingListener {

    @PrePersist
    private void prePersist(BaseEntity entity) {
    //entity.setVersion(1); 如果注释了，测试用例这个地方的验证也需要去掉
        log.info("prePersist::{}",entity.toString());
    }

    @PostPersist
    public void postPersist(Object entity) {
        log.info("postPersist::{}",entity.toString());
    }

    @PreUpdate
    public void preUpdate(BaseEntity entity) {
    //entity.setCreateUserId(200); 如果注释了，测试用例这个地方的验证也需要去掉
        log.info("preUpdate::{}",entity.toString());
    }

    @PostUpdate
    public void postUpdate(Object entity) {
        log.info("postUpdate::{}",entity.toString());
    }

    @PreRemove
    public void preRemove(Object entity) {
        log.info("preRemove::{}",entity.toString());
    }

    @PostRemove
    public void postRemove(Object entity) {
        log.info("postRemove::{}",entity.toString());
    }

    @PostLoad
    public void postLoad(Object entity) {
    //查询方法里面可以对一些敏感信息做一些日志
        if (User.class.isInstance(entity)) {
            log.info("postLoad::{}",entity.toString());
        }
    }
}
```

在这一步骤中需要注意的是：

1. 我们上面注释的代码，也可以改变 entity 里面的值，但是在这个 Listener 的里面我们不做修改，所以把 setVersion 和 setCreateUserId 注释掉了，要注意测试用例里面这两处也需要修改。
2. 如果在 @PostLoad 里面记录日志，不一定每个实体、每次查询都需要记录日志，只需要对一些敏感的实体或者字段做日志记录即可。
3. 回调函数时我们可以加上参数，这个参数可以是父类 Object，可以是 BaseEntity，也可以是具体的某一个实体；我推荐用 BaseEntity，因为这样的方法是类型安全的，它可以约定一些框架逻辑，比如 getCreateUserId、getLastModifiedUserId 等。

**第二步：还是一样的道理，写一个测试用例跑一下**。

### JPA Callbacks 的最佳实践

**我以个人经验总结了几个最佳实践。**

1. 回调函数里面应尽量避免直接操作业务代码，最好用一些具有框架性的公用代码，如上一课时我们讲的 Auditing，以及本课时前面提到的实体操作日志等；

2. 注意回调函数方法要在同一个事务中进行，异常要可预期，非可预期的异常要进行捕获，以免出现意想不到的线上 Bug；

3. 回调函数方法是同步的，如果一些计算量大的和一些耗时的操作，可以通过发消息等机制异步处理，以免阻塞主流程，影响接口的性能;

4. 在回调函数里面，尽量不要直接在操作 EntityManager 后再做 session 的整个生命周期的其他持久化操作，以免破坏事务的处理流程；也不要进行其他额外的关联关系更新动作，业务性的代码一定要放在 service 层面，否则太过复杂，时间长了代码很难维护；

5. 回调函数里面比较适合用一些计算型的transient方法，如下面这个操作：

   ```java
   public class UserListener {
       @PrePersist
       public void prePersist(User user) {
           //通过一些逻辑计算年龄；
           user.calculationAge();
       }
   }
   ```

6. JPA 官方比较建议放一些默认值，但是我不是特别赞同，因为觉得那样不够直观，我们直接用字段初始化就可以了，没必要在回调函数里面放置默认值。

## 乐观锁和重试机制

乐观锁和重试其实不是Spring Data Jpa的功能，这里结合Spring Data Jpa说明使用方法。

Spring 全家桶里面提供了@Retryable 的注解，会帮我们进行重试。下面看一个 @Retryable 的例子。

第一步：利用 gradle 引入 spring-retry 的依赖 jar，如下所示：

```groovy
implementation 'org.springframework.retry:spring-retry'
```

第二步：在 UserInfoserviceImpl 的方法中添加 @Retryable 注解，就可以实现重试的机制了

第三步：新增一个RetryConfiguration并添加@EnableRetry 注解，是为了开启重试机制，使 @Retryable 生效。

```java
@EnableRetry
@Configuration
public class RetryConfiguration {
}
```

通过案例你会发现 Retry 的逻辑其实很简单，只需要利用 @Retryable 注解即可。

下面对常用的 @Retryable 注解中的参数做一下说明：

- maxAttempts：最大重试次数，默认为 3，如果要设置的重试次数为 3，可以不写；
- value：抛出指定异常才会重试；
- include：和 value 一样，默认为空，当 exclude 也为空时，默认异常；
- exclude：指定不处理的异常；
- backoff：重试等待策略，默认使用 @Backoff的 value，默认为 1s。

其中：

- value=delay：隔多少毫秒后重试，默认为 1000L，单位是毫秒；
- multiplier（指定延迟倍数）默认为 0，表示固定暂停 1 秒后进行重试，如果把 multiplier 设置为 1.5，则第一次重试为 2 秒，第二次为 3 秒，第三次为 4.5 秒。

下面是一个关于 @Retryable 扩展的使用例子，具体看一下代码：

```java
@Service
public interface MyService {
    @Retryable( value = SQLException.class, maxAttempts = 2, backoff = @Backoff(delay = 100))
    void retryServiceWithCustomization(String sql) throws SQLException;
}
```

可以看到，这里明确指定 SQLException.class 异常的时候需要重试两次，每次中间间隔 100 毫秒。

```java
@Service 
public interface MyService { 
    @Retryable( value = SQLException.class, maxAttemptsExpression = "${retry.maxAttempts}",
               backoff = @Backoff(delayExpression = "${retry.maxDelay}")) 
    void retryServiceWithExternalizedConfiguration(String sql) throws SQLException; 
}
```

此外，你也可以利用 SpEL 表达式读取配置文件里面的值。

关于 Retryable 的语法就介绍到这里，常用的基本就这些，如果你遇到更复杂的场景，可以到 GitHub 中看一下官方的 Retryable 文档：https://github.com/spring-projects/spring-retry。下面再给你分享一个我在使用乐观锁+重试机制中的最佳实践。

### 乐观锁+重试机制的最佳实践

我比较建议你使用如下配置：

```java
@Retryable(value = ObjectOptimisticLockingFailureException.class,backoff = @Backoff(multiplier = 1.5,random = true))
```

这里明确指定 `ObjectOptimisticLockingFailureException.class` 等乐观锁异常要进行重试，如果引起其他异常的话，重试会失败，没有意义；而 backoff 采用随机 +1.5 倍的系数，这样基本很少会出现连续 3 次乐观锁异常的情况，并且也很难发生重试风暴而引起系统重试崩溃的问题。

到这里讲的一直都是乐观锁相关内容，那么 JPA 也支持悲观锁吗？

### 悲观锁

Java Persistence API 2.0 协议里面有一个 LockModeType 枚举值，里面包含了所有它支持的乐观锁和悲观锁的值，我们看一下。

```java
public enum LockModeType {
    //等同于OPTIMISTIC，默认，用来兼容2.0之前的协议
    READ,
    //等同于OPTIMISTIC_FORCE_INCREMENT，用来兼容2.0之前的协议
    WRITE,
    //乐观锁，默认，2.0协议新增
    OPTIMISTIC,
    //乐观写锁，强制version加1，2.0协议新增
    OPTIMISTIC_FORCE_INCREMENT,
    //悲观读锁 2.0协议新增
    PESSIMISTIC_READ,
    //悲观写锁，version不变，2.0协议新增
    PESSIMISTIC_WRITE,
    //悲观写锁，version会新增，2.0协议新增
    PESSIMISTIC_FORCE_INCREMENT,
    //2.0协议新增无锁状态
    NONE
}
```

悲观锁在 Spring Data JPA 里面是如何支持的呢？很简单，只需要在自己的 Repository 里面覆盖父类的 Repository 方法，然后添加 @Lock 注解并指定 LockModeType 即可，请看如下代码：

```java
public interface UserInfoRepository extends JpaRepository<UserInfo, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    Optional<UserInfo> findById(Long userId);
}
```

**在生产环境中要慎用悲观锁，因为它是阻塞的，一旦发生服务异常，可能会造成死锁的现象。**

## JPA 对 Web MVC的支持

我们使用 Spring Data JPA 的时候，一般都会用到 Spring MVC，Spring Data 对 Spring MVC 做了很好的支持，体现在以下几个方面：

- 支持在 Controller 层直接返回实体，而不使用其显式的调用方法；
- 对 MVC 层支持标准的分页和排序功能；
- 扩展的插件支持 Querydsl，可以实现一些通用的查询逻辑。

正常情况下，我们开启 Spring Data 对 Spring Web MVC 支持的时候需要在 @Configuration 的配置文件里面添加 @EnableSpringDataWebSupport 这一注解，如下面这种形式：

```java
@Configuration
@EnableWebMvc
//开启支持Spring Data Web的支持
@EnableSpringDataWebSupport
public class WebConfiguration { }
```

由于我们用了 Spring Boot，其有自动加载机制，会自动加载 SpringDataWebAutoConfiguration 类，发生如下变化：

```java
@EnableSpringDataWebSupport
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ PageableHandlerMethodArgumentResolver.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(PageableHandlerMethodArgumentResolver.class)
@EnableConfigurationProperties(SpringDataWebProperties.class)
@AutoConfigureAfter(RepositoryRestMvcAutoConfiguration.class)
public class SpringDataWebAutoConfiguration {}
```

从类上面可以看出来，@EnableSpringDataWebSupport 会自动开启，所以当我们用 Spring Boot + JPA + MVC 的时候，什么都不需要做，因为 Spring Boot 利用 Spring Data 对 Spring MVC 做了很多 Web 开发的天然支持。支持的组件有 DomainConverter、Page、Sort、Databinding、Dynamic Param 等。

那么我们先来看一下它对 DomainClassConverter 组件的支持。

### DomainClassConverter 组件

这个组件的主要作用是帮我们把 Path 中 ID 的变量，或 Request 参数中的变量 ID 的参数值，直接转化成实体对象注册到 Controller 方法的参数里面。怎么理解呢？我们看个例子，就很好懂了。

首先，写一个 MVC 的 Controller，分别从 Path 和 Param 变量里面，根据 ID 转化成实体，代码如下：

```java
@RestController
public class UserInfoController {
    /**
    * 从path变量里面获得参数ID的值，然后直接转化成UserInfo实体
    * @param userInfo
    * @return
    */
    @GetMapping("/user/{id}")
    public UserInfo getUserInfoFromPath(@PathVariable("id") UserInfo userInfo) {
        return userInfo;
    }
    /**
    * 将request的param中的ID变量值，转化成UserInfo实体
    * @param userInfo
    * @return
    */
    @GetMapping("/user")
    public UserInfo getUserInfoFromRequestParam(@RequestParam("id") UserInfo userInfo) {
        return userInfo;
    }
}
```

Controller 里面的 getUserInfoFromRequestParam 方法会自动根据 ID 查询实体对象 UserInfo，然后注入方法的参数里面。

### Page 和 Sort 的参数支持

这是一个通过分页和排序参数查询 UserInfo 的实例。

首先，我们新建一个 UserInfoController，里面添加如下两个方法，分别测试分页和排序。

```java
@GetMapping("/users")
public Page<UserInfo> queryByPage(Pageable pageable, UserInfo userInfo) {
    return userInfoRepository.findAll(Example.of(userInfo),pageable);
}
@GetMapping("/users/sort")
public HttpEntity<List<UserInfo>> queryBySort(Sort sort) {
    return new HttpEntity<>(userInfoRepository.findAll(sort));
}
```

其中，queryByPage 方法中，两个参数可以分别接收分页参数和查询条件，我们请求一下，看看效果：
复制代码

```bash
GET http://127.0.0.1:8089/users?size=2&page=0&ages=10&sort=id,desc
```

参数里面可以支持分页大小为 2、页码 0、排序（按照 ID 倒序）、参数 ages=10 的所有结果，Pageable 既支持分页参数，也支持排序参数。也可以单独调用 Sort 参数。

### Web Databinding Support

Spring Data JPA 里面，可以通过 @ProjectedPayload 和 @JsonPath 对接口进行注解支持，不过要注意这与前面所讲的 Jackson 注解的区别在于，此时我们讲的是接口。

这里我依然结合一个实例来对这个接口进行讲解，请看下面的步骤。

第一步：如果要支持 Projection，必须要在 gradle 里面引入 jsonpath 依赖才可以：

```groovy
implementation 'com.jayway.jsonpath:json-path'
```

第二步：新建一个 UserInfoInterface 接口类，用来接收接口传递的 json 对象。

```java
package com.example.jpa.example1;
import org.springframework.data.web.JsonPath;
import org.springframework.data.web.ProjectedPayload;
@ProjectedPayload
public interface UserInfoInterface {
    @JsonPath("$.ages") // 第一级参数/JSON里面找ages字段
    // @JsonPath("$..ages") $..代表任意层级找ages字段
    Integer getAges();
    @JsonPath("$.telephone") //第一级找参数/JSON里面的telephone字段
    // @JsonPath({ "$.telephone", "$.user.telephone" }) //第一级或者user下面的telephone都可以
    String getTelephone();
}
```

第三步：在 Controller 里面新建一个 post 方法，通过接口获得 RequestBody 参数对象里面的值。

```java
@PostMapping("/users/projected")
public UserInfoInterface saveUserInfo(@RequestBody UserInfoInterface userInfoInterface) {
    return userInfoInterface;
}
```

第四步：我们发送一个 get 请求，代码如下：

```json
POST /users HTTP/1.1
{"ages":10,"telephone":"123456789"}
```

此时可以正常得到如下结果：

```json
{
    "ages": 10,
    "telephone": "123456789"
}
```

这个响应结果说明了接口可以正常映射。

### QueryDSL Web Support

实际工作中，经常有人会用 Querydsl 做一些复杂查询，方便生成 Rest 的 API 接口，那么这种方法有什么好处，又会暴露什么缺点呢？我们先看一个实例。

这是一个通过 QueryDSL 作为请求参数的使用案例，通过它你就可以体验一下 QueryDSL 的用法和使用场景，我们一步一步来看一下。

第一步：需要 grandle 引入 querydsl 的依赖。

```groovy
implementation 'com.querydsl:querydsl-apt'
implementation 'com.querydsl:querydsl-jpa'
annotationProcessor("com.querydsl:querydsl-apt:4.3.1:jpa",
        "org.hibernate.javax.persistence:hibernate-jpa-2.1-api:1.0.2.Final",
        "javax.annotation:javax.annotation-api:1.3.2",
        "org.projectlombok:lombok")
annotationProcessor("org.springframework.boot:spring-boot-starter-data-jpa")
annotationProcessor 'org.projectlombok:lombok'
```

第二步：UserInfoRepository 继承 QuerydslPredicateExecutor 接口，就可以实现 QueryDSL 的查询方法了，代码如下：

```java
public interface UserInfoRepository extends JpaRepository<UserInfo, Long>, QuerydslPredicateExecutor<UserInfo> {}
```

第三步：Controller 里面直接利用 @QuerydslPredicate 注解接收 Predicate predicate 参数。

```java
@GetMapping(value = "user/dsl")
Page<UserInfo> queryByDsl(@QuerydslPredicate(root = UserInfo.class) com.querydsl.core.types.Predicate predicate, Pageable pageable) {
    //这里面我用的userInfoRepository里面的QuerydslPredicateExecutor里面的方法
    return userInfoRepository.findAll(predicate, pageable);
}
```

第四步：直接请求我们的 user / dsl 即可，这里利用 queryDsl 的语法 ，使 &ages=10 作为我们的请求参数。

```json
GET http://127.0.0.1:8089/user/dsl?size=2&page=0&ages=10&sort=id%2Cdesc&ages=10
Content-Type: application/json
{
  "content": [
    {
      "id": 2,
      "version": 0,
      "ages": 10,
      "telephone": "123456789"
    },
    {
      "id": 1,
      "version": 0,
      "ages": 10,
      "telephone": "123456789"
    }
  ],
  "pageable": {
    "sort": {
      "sorted": true,
      "unsorted": false,
      "empty": false
    },
    "offset": 0,
    "pageNumber": 0,
    "pageSize": 2,
    "unpaged": false,
    "paged": true
  },
  "totalPages": 1,
  "totalElements": 2,
  "last": true,
  "size": 2,
  "number": 0,
  "sort": {
    "sorted": true,
    "unsorted": false,
    "empty": false
  },
  "numberOfElements": 2,
  "first": true,
  "empty": false
}
Response code: 200; Time: 721ms; Content length: 425 bytes
```

现在我们可以得出结论：QuerysDSL 可以帮我们省去创建 Predicate 的过程，简化了操作流程。但是它依然存在一些局限性，比如多了一些模糊查询、范围查询、大小查询，它对这些方面的支持不是特别友好。可能未来会更新、优化。

### @DynamicUpdate & @DynamicInsert 详解

@DynamicInsert：这个注解表示 insert 的时候，会动态生产 insert SQL 语句，其生成 SQL 的规则是：只有非空的字段才能生成 SQL。代码如下：

```java
@Target( TYPE )
@Retention( RUNTIME )
public @interface DynamicInsert {
    //默认是true，如果设置成false，就表示空的字段也会生成sql语句；
    boolean value() default true;
}
```

这个注解主要是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 insert SQL 的 Columns 只包含非空的字段；如果实体中不加这个注解，默认的情况是空的，字段也会作为 insert 语句里面的 Columns。

@DynamicUpdate：和 insert 是一个意思，只不过这个注解指的是在 update 的时候，会动态产生 update SQL 语句，生成 SQL 的规则是：只有非空的字段才会生成到 update SQL 的 Columns 里面。请看代码：

```java
@Target( TYPE )
@Retention( RUNTIME )
public @interface DynamicUpdate {
    //和insert里面一个意思，默认true;
    boolean value() default true;
}
```

和上一个注解的原理类似，这个注解也是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 update SQL 的 Columns 只包含非空的字段；如果不加这个注解，默认的情况是空的字段也会作为 update 语句里面的 Columns。

# 扩展使用

## N+1 SQL 问题

想要解决一个问题，必须要知道它是什么、如何产生的，这样才能有方法、有逻辑地去解决它。下面通过一个例子来看一下什么是 N+1 的 SQL 问题。

假设一个 UserInfo 实体对象和 Address 是一对多的关系，即一个用户有多个地址，我们首先看一下一般实体里面的关联关系会怎么写。两个实体对象如下述代码所示。

```java
// UserInfo实体对象如下：
@Entity
@Table
@ToString(exclude = "addressList")//exclued防止 toString打印日志的时候死循环
public class UserInfo extends BaseEntity {
    private String name;
    private String telephone;
    // UserInfo实体对象的关联关系由Address对象里面的userInfo字段维护，默认是lazy加载模式，为了方便演示fetch取EAGER模式。此处是一对多关联关系
    @OneToMany(mappedBy = "userInfo",fetch = FetchType.EAGER)
    private List<Address> addressList;
}

// Address对象如下：
@Entity
@Table
@ToString(exclude = "userInfo")
public class Address extends BaseEntity {
    private String city;
    //维护UserInfo和Address的外键关系，方便演示也采用EAGER模式；
    @ManyToOne(fetch = FetchType.EAGER)
    @JsonBackReference //此注解防止JSON死循环
    private UserInfo userInfo;
}
```

然后，我们请求通过 UserInfoRepository 查询所有的 UserInfo 信息，方法如下面这行代码所示。

```java
userInfoRepository.findAll()
```

现在，我们的控制台将会得到四个 SQL，如下所示。

```sql
org.hibernate.SQL                        :
select userinfo0_.id                    as id1_1_,
       userinfo0_.create_time           as create_t2_1_,
       userinfo0_.create_user_id        as create_u3_1_,
       userinfo0_.last_modified_time    as last_mod4_1_,
       userinfo0_.last_modified_user_id as last_mod5_1_,
       userinfo0_.version               as version6_1_,
       userinfo0_.ages                  as ages7_1_,
       userinfo0_.email_address         as email_ad8_1_,
       userinfo0_.last_name             as last_nam9_1_,
       userinfo0_.name                  as name10_1_,
       userinfo0_.telephone             as telepho11_1_
from user_info userinfo0_ org.hibernate.SQL                        :
select addresslis0_.user_info_id          as user_inf8_0_0_,
       addresslis0_.id                    as id1_0_0_,
       addresslis0_.id                    as id1_0_1_,
       addresslis0_.create_time           as create_t2_0_1_,
       addresslis0_.create_user_id        as create_u3_0_1_,
       addresslis0_.last_modified_time    as last_mod4_0_1_,
       addresslis0_.last_modified_user_id as last_mod5_0_1_,
       addresslis0_.version               as version6_0_1_,
       addresslis0_.city                  as city7_0_1_,
       addresslis0_.user_info_id          as user_inf8_0_1_
from address addresslis0_
where addresslis0_.user_info_id = ? org.hibernate.SQL                        :
select addresslis0_.user_info_id          as user_inf8_0_0_,
       addresslis0_.id                    as id1_0_0_,
       addresslis0_.id                    as id1_0_1_,
       addresslis0_.create_time           as create_t2_0_1_,
       addresslis0_.create_user_id        as create_u3_0_1_,
       addresslis0_.last_modified_time    as last_mod4_0_1_,
       addresslis0_.last_modified_user_id as last_mod5_0_1_,
       addresslis0_.version               as version6_0_1_,
       addresslis0_.city                  as city7_0_1_,
       addresslis0_.user_info_id          as user_inf8_0_1_
from address addresslis0_
where addresslis0_.user_info_id = ? org.hibernate.SQL                        :
select addresslis0_.user_info_id          as user_inf8_0_0_,
       addresslis0_.id                    as id1_0_0_,
       addresslis0_.id                    as id1_0_1_,
       addresslis0_.create_time           as create_t2_0_1_,
       addresslis0_.create_user_id        as create_u3_0_1_,
       addresslis0_.last_modified_time    as last_mod4_0_1_,
       addresslis0_.last_modified_user_id as last_mod5_0_1_,
       addresslis0_.version               as version6_0_1_,
       addresslis0_.city                  as city7_0_1_,
       addresslis0_.user_info_id          as user_inf8_0_1_
from address addresslis0_
where addresslis0_.user_info_id = ?
```

通过 SQL 我们可以看得出来，当取 UserInfo 的时候，有多少条 UserInfo 数据就会触发多少条查询 Address 的 SQL。

那么所谓的 N+1 的 SQL，此时 1 代表的是一条 SQL 查询 UserInfo 信息；N 条 SQL 查询 Address 的信息。你可以想象一下，如果有 100 条 UserInfo 信息，可能会触发 100 条查询 Address 的 SQL，性能很差。

很简单，这就是我们常说的 N+1 SQL 问题。我们这里使用的是 EAGER 模式，当使用 LAZY 的时候也是一样的道理，只是生成 N 条 SQL 的时机是不一样的。

现在你认识了这个问题，下一步该思考，怎么解决才更合理呢？有没有什么办法可以减少 SQL 条数呢？

### 减少 N+1 SQL 的条数

最容易想到，就是有没有什么机制可以减少 N 对应的 SQL 条数呢？从原理分析会知道，不管是 LAZY 还是 EAGER 都是没有用的，因为这两个只是决定了 N 条 SQL 的触发时机，而不能减少 SQL 的条数。

不知道你是否还记得在第 20 讲（Spring JPA 中的 Hibernate 加载过程与配置项是怎么回事）中，我们介绍过的 Hibernate 的配置项有哪些，如果你回过头去看，会发现有个配置可以改变每次批量取数据的大小。
hibernate.default_batch_fetch_size 配置

hibernate.default_batch_fetch_size 配置在 AvailableSettings.class 里面，指的是批量获取数据的大小，默认是 -1，表示默认没有匹配取数据。那么我们把这个值改成 20 看一下效果，只需要在 application.properties 里面增加如下配置即可。

```properties
# 更改批量取数据的大小为20
spring.jpa.properties.hibernate.default_batch_fetch_size= 20
```

在实体类不发生任何改变的前提下，我们再执行如下两个方法，分别看一下 SQL 的生成情况。

```java
userInfoRepository.findAll();
```

还是先查询所有的 UserInfo 信息，看一下 SQL 的执行情况，代码如下所示。

```sql
org.hibernate.SQL                        :
select userinfo0_.id                    as id1_1_,
       userinfo0_.create_time           as create_t2_1_,
       userinfo0_.create_user_id        as create_u3_1_,
       userinfo0_.last_modified_time    as last_mod4_1_,
       userinfo0_.last_modified_user_id as last_mod5_1_,
       userinfo0_.version               as version6_1_,
       userinfo0_.ages                  as ages7_1_,
       userinfo0_.email_address         as email_ad8_1_,
       userinfo0_.last_name             as last_nam9_1_,
       userinfo0_.name                  as name10_1_,
       userinfo0_.telephone             as telepho11_1_
from user_info userinfo0_ org.hibernate.SQL                        :
select addresslis0_.user_info_id          as user_inf8_0_1_,
       addresslis0_.id                    as id1_0_1_,
       addresslis0_.id                    as id1_0_0_,
       addresslis0_.create_time           as create_t2_0_0_,
       addresslis0_.create_user_id        as create_u3_0_0_,
       addresslis0_.last_modified_time    as last_mod4_0_0_,
       addresslis0_.last_modified_user_id as last_mod5_0_0_,
       addresslis0_.version               as version6_0_0_,
       addresslis0_.city                  as city7_0_0_,
       addresslis0_.user_info_id          as user_inf8_0_0_
from address addresslis0_
where addresslis0_.user_info_id in (?, ?, ?)
```

我们可以看到 SQL 直接减少到两条了，其中查询 Address 的地方查询条件变成了 in(?,?,?)。

想象一下，如果我们有 20 条 UserInfo 信息，那么产生的 SQL 也是两条，此时要比 20+1 条 SQL 性能高太多了。

而 hibernate.default_batch_fetch_size 的经验参考值，可以设置成 20、30、50、100 等，太高了也没有意义。一个请求执行一次，产生的 SQL 数量为 3-5 条基本上都算合理情况，这样通过设置 default_batch_fetch_size 就可以很好地避免大部分业务场景下的 N+1 条 SQL 的性能问题了。

此时你还需要注意一点就是，在实际工作中，一定要知道我们一次操作会产生多少 SQL，有没有预期之外的 SQL 参数，这是需要关注的重点，这种情况可以利用我们之前说过的如下配置来开启打印 SQL，请看代码。

```properties
## 显示sql的执行日志，如果开了这个,show_sql就可以不用了，show_sql没有上下文，多线程情况下，分不清楚是谁打印的，所有我推荐如下配置项：
logging.level.org.hibernate.SQL=debug
```

但是这种配置也有个缺陷，就是只能全局配置，没办法针对不通过的实体管理关系配置不同的 Fetch Size 的值。

而与之类似的 Hibernate 里面也提供了一个注解 @BatchSize 可以解决此问题。

### @BatchSize 注解

@BatchSize 注解是 Hibernate 提供的用来解决查询关联关系的批量处理大小，默认无，可以配置在实体上，也可以配置在关联关系上面。此注解里面只有一个属性 size，用来指定关联关系 LAZY 或者是 EAGER 一次性取数据的大小。

我们还是将上面的例子中的 UserInfo 实体做一下改造，在里面增加两次 @BatchSize 注解，代码如下所示。

```java
@Entity
@Table
@ToString(exclude = "addressList")
@BatchSize(size = 2)//实体类上加@BatchSize注解，用来设置当被关联关系的时候一次查询的大小，我们设置成2，方便演示Address关联UserInfo的时候的效果
public class UserInfo extends BaseEntity {
   private String name;
   private String telephone;
   @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
   @BatchSize(size = 20)//关联关系的属性上加@BatchSize注解，用来设置当通过UserInfo加载Address的时候一次取数据的大小
   private List<Address> addressList;
}
```

我们通过改造 UserInfo 实体，可以直接演示 @BatchSize 应用在实体类和属性字段上的效果，所以 Address 实体可以不做任何改变，hibernate.default_batch_fetch_size 还改成默认值 -1，我们再分别执行一下两个 findAll 方法，看一下效果。

第一种：查询所有 UserInfo，代码如下面这行所示。

```java
userInfoRepository.findAll()
```

我们看一下 SQL 控制台。

```java
 org.hibernate.SQL                        :
select userinfo0_.id                    as id1_1_,
       userinfo0_.create_time           as create_t2_1_,
       userinfo0_.create_user_id        as create_u3_1_,
       userinfo0_.last_modified_time    as last_mod4_1_,
       userinfo0_.last_modified_user_id as last_mod5_1_,
       userinfo0_.version               as version6_1_,
       userinfo0_.ages                  as ages7_1_,
       userinfo0_.email_address         as email_ad8_1_,
       userinfo0_.last_name             as last_nam9_1_,
       userinfo0_.name                  as name10_1_,
       userinfo0_.telephone             as telepho11_1_
from user_info userinfo0_ org.hibernate.SQL                        :
select addresslis0_.user_info_id          as user_inf8_0_1_,
       addresslis0_.id                    as id1_0_1_,
       addresslis0_.id                    as id1_0_0_,
       addresslis0_.create_time           as create_t2_0_0_,
       addresslis0_.create_user_id        as create_u3_0_0_,
       addresslis0_.last_modified_time    as last_mod4_0_0_,
       addresslis0_.last_modified_user_id as last_mod5_0_0_,
       addresslis0_.version               as version6_0_0_,
       addresslis0_.city                  as city7_0_0_,
       addresslis0_.user_info_id          as user_inf8_0_0_
from address addresslis0_
where addresslis0_.user_info_id in (?, ?, ?)
```

和刚才设置 hibernate.default_batch_fetch_size=20 的效果一模一样，所以我们可以利用 @BatchSize 这个注解针对不同的关联关系，配置不同的大小，从而提升 N+1 SQL 的性能。

**注意事项：**

@BatchSize 的使用具有局限性，不能作用于 @ManyToOne 和 @OneToOne 的关联关系上，那样代码是不起作用的，如下所示。

```java
public class Address extends BaseEntity {
    private String city;
    @ManyToOne(cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
    @BatchSize(size = 30) //由于是@ManyToOne的关联关系所有没有作用
    private UserInfo userInfo;
}
```

因此，要注意 @BatchSize 只能作用在 @ManyToMany、@OneToMany、实体类这三个地方。

### Hibernate 中 @Fetch 数据的策略

Hibernate 提供了一个 @Fetch 注解，用来改变获取数据的策略。我们来研究一下这一注解的语法，代码如下所示。

```java
// fetch注解只能用在方法和字段上面
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Fetch {
    //注解里面，只有一个属性获取数据的模式
    FetchMode value();
}
//其中FetchMode的值有如下几种：
public enum FetchMode {
    //默认模式，就是会有N+1 sql的问题；
    SELECT,
    //通过join的模式，用一个sql把主体数据和关联关系数据一口气查出来
    JOIN,
    //通过子查询的模式，查询关联关系的数据
    SUBSELECT
}
```

需要注意的是，不要把这个注解和 JPA 协议里面的 FetchType.EAGER、FetchType.LAZY 搞混了，JPA 协议的关联关系中的 FetchTyp 解决的是取关联关系数据时机的问题，也就是说 EAGER 代表的是立即获得关联关系的数据，LAZY 是需要的时候再获得关联关系的数据。

这和 Hibernate 的 FetchMode 是两回事，FetchMode 解决的是获得数据策略的问题，也就是说，获得关联关系数据的策略有三种模式：SELECT（默认）、JOIN、SUBSELECT。下面我通过例子来分别介绍一下这三种模式有什么区别，分别起到什么作用。

#### FetchMode.SELECT

FetchMode.Select 是默认策略，加与不加是同样的效果，代表获取关系的时候新开一个 SQL 进行查询，依然会产生 N+1 的 SQL 问题。

#### FetchMode.JOIN

FetchMode.JOIN 的意思是主表信息和关联关系通过一个 SQL JOIN 的方式查出来，我们看一下例子。

首先，将 UserInfo 里面的 FetchMode 改成 JOIN 模式，关键代码如下。

```java
public class UserInfo extends BaseEntity {
    private String name;
    private String telephone;
    @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
    @Fetch(value = FetchMode.JOIN) //唯一变化的地方采用JOIN模式
    private List<Address> addressList;
}
```

然后，调用一下 userInfoRepository.findAll(); 这个方法，发现依然是这三条 SQL。

这是因为 FetchMode.JOIN 只支持通过 ID 或者联合唯一键获取数据才有效，这正是 JOIN 策略模式的局限性所在。

那么我们再调用一下 userInfoRepository.findById(id)，看看控制台的 SQL 执行情况，代码如下。

```java
select userinfo0_.id                      as id1_1_0_,
       userinfo0_.create_time             as create_t2_1_0_,
       userinfo0_.create_user_id          as create_u3_1_0_,
       userinfo0_.last_modified_time      as last_mod4_1_0_,
       userinfo0_.last_modified_user_id   as last_mod5_1_0_,
       userinfo0_.version                 as version6_1_0_,
       userinfo0_.ages                    as ages7_1_0_,
       userinfo0_.email_address           as email_ad8_1_0_,
       userinfo0_.last_name               as last_nam9_1_0_,
       userinfo0_.name                    as name10_1_0_,
       userinfo0_.telephone               as telepho11_1_0_,
       addresslis1_.user_info_id          as user_inf8_0_1_,
       addresslis1_.id                    as id1_0_1_,
       addresslis1_.id                    as id1_0_2_,
       addresslis1_.create_time           as create_t2_0_2_,
       addresslis1_.create_user_id        as create_u3_0_2_,
       addresslis1_.last_modified_time    as last_mod4_0_2_,
       addresslis1_.last_modified_user_id as last_mod5_0_2_,
       addresslis1_.version               as version6_0_2_,
       addresslis1_.city                  as city7_0_2_,
       addresslis1_.user_info_id          as user_inf8_0_2_
from user_info userinfo0_
         left outer join address addresslis1_ on userinfo0_.id = addresslis1_.user_info_id
where userinfo0_.id = ?
```

这时我们会发现，当查询 UserInfo 的时候，它会通过 left outer join 把 Address 的信息也查询出来，虽然 SQL 上会有冗余信息，但是你会发现我们之前的 N+1 的 SQL 直接变成 1 条 SQL 了。

此时我们修改 UserInfo 里面的 @OneToMany，这个 @Fetch(value = FetchMode.JOIN) 同样适用于 @ManyToOne；然后再改一下 Address 实例，用 @Fetch(value = FetchMode.JOIN) 把 Adress 里面的 UserInfo 关联关系改成 JOIN 模式；接着我们用 LAZY 获取数据的时机，会发现其对获取数据的策略没有任何影响。

这里我只是给你演示获取数据时机的不同情况，关键代码如下。

```java
@Entity
@Table
@ToString(exclude = "userInfo")
public class Address extends BaseEntity {
    private String city;
    @ManyToOne(cascade = CascadeType.PERSIST,fetch = FetchType.LAZY)
    @JsonBackReference
    @Fetch(value = FetchMode.JOIN)
    private UserInfo userInfo;
}
```

同样的道理，JOIN 对列表性的查询是没有效果的，我们调用一下 addressRepository.findById(id)，产生的 SQL 如下所示。

```sql
org.hibernate.SQL                        : 
select address0_.id                     as id1_0_0_,
       address0_.create_time            as create_t2_0_0_,
       address0_.create_user_id         as create_u3_0_0_,
       address0_.last_modified_time     as last_mod4_0_0_,
       address0_.last_modified_user_id  as last_mod5_0_0_,
       address0_.version                as version6_0_0_,
       address0_.city                   as city7_0_0_,
       address0_.user_info_id           as user_inf8_0_0_,
       userinfo1_.id                    as id1_1_1_,
       userinfo1_.create_time           as create_t2_1_1_,
       userinfo1_.create_user_id        as create_u3_1_1_,
       userinfo1_.last_modified_time    as last_mod4_1_1_,
       userinfo1_.last_modified_user_id as last_mod5_1_1_,
       userinfo1_.version               as version6_1_1_,
       userinfo1_.ages                  as ages7_1_1_,
       userinfo1_.email_address         as email_ad8_1_1_,
       userinfo1_.last_name             as last_nam9_1_1_,
       userinfo1_.name                  as name10_1_1_,
       userinfo1_.telephone             as telepho11_1_1_
from address address0_
         left outer join user_info userinfo1_ on address0_.user_info_id = userinfo1_.id
where address0_.id = ?
```

我们发现此时只会产生一个 SQL，即通过 from address left outer join user_info 一次性把所有信息都查出来，然后 Hibernate 再根据查询出来的结果组合到不同的实体里面。

也就是说 FetchMode.JOIN 对于关联关系的查询 LAZY 是不起作用的，因为 JOIN 的模式是通过一条 SQL 查出来所有信息，所以 FetchMode.JOIN 会忽略 FetchType。

#### FetchMode.SUBSELECT

这种模式很简单，就是将关联关系通过子查询的形式查询出来，我们还是结合例子来理解一下。

首先，将 UserInfo 里面的关联关系改成 @Fetch(value = FetchMode.SUBSELECT)，关键代码如下。

```java
public class UserInfo extends BaseEntity {
    @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.LAZY) //我们这里测试一下LAZY情况
    @Fetch(value = FetchMode.SUBSELECT) //唯一变化之处
    private List<Address> addressList;
}
```

接着，像上面的做法一样，执行一下 userInfoRepository.findAll()；方法，看一下控制台的 SQL 情况，如下所示。

```sql
org.hibernate.SQL                        :
select userinfo0_.id                    as id1_1_,
       userinfo0_.create_time           as create_t2_1_,
       userinfo0_.create_user_id        as create_u3_1_,
       userinfo0_.last_modified_time    as last_mod4_1_,
       userinfo0_.last_modified_user_id as last_mod5_1_,
       userinfo0_.version               as version6_1_,
       userinfo0_.ages                  as ages7_1_,
       userinfo0_.email_address         as email_ad8_1_,
       userinfo0_.last_name             as last_nam9_1_,
       userinfo0_.name                  as name10_1_,
       userinfo0_.telephone             as telepho11_1_
from user_info userinfo0_ 
org.hibernate.SQL                        :
select addresslis0_.user_info_id          as user_inf8_0_1_,
       addresslis0_.id                    as id1_0_1_,
       addresslis0_.id                    as id1_0_0_,
       addresslis0_.create_time           as create_t2_0_0_,
       addresslis0_.create_user_id        as create_u3_0_0_,
       addresslis0_.last_modified_time    as last_mod4_0_0_,
       addresslis0_.last_modified_user_id as last_mod5_0_0_,
       addresslis0_.version               as version6_0_0_,
       addresslis0_.city                  as city7_0_0_,
       addresslis0_.user_info_id          as user_inf8_0_0_
from address addresslis0_
where addresslis0_.user_info_id in (select userinfo0_.id from user_info userinfo0_)
```

这个时候会发现，查询 Address 信息是直接通过 addresslis0_.user_info_id in (select userinfo0_.id from user_info userinfo0_) 子查询的方式进行的，也就是说 N+1 SQL 变成了 1+1 的 SQL，这有点类似我们配置 @BatchSize 的效果。

FetchMode.SUBSELECT 支持 ID 查询和各种条件查询，唯一的缺点是只能配置在 @OneToMany 和 @ManyToMany 的关联关系上，不能配置在 @ManyToOne 和 @OneToOne 的关联关系上，所以我们在 Address 里面关联 UserInfo 的时候就没有办法做实验了。

总之，@Fetch 的不同模型，都有各自的优缺点：FetchMode.SELECT 默认，和不配置的效果一样；FetchMode.JOIN 只支持类似 findById(id) 的方法，只能根据 ID 查询才有效果；FetchMode.SUBSELECT 虽然不限使用方式，但是只支持 OneToMany 的关联关系。

所以你在使用 @Fetch 的时候需要注意一下它的局限性，我个人是比较推荐 @BatchSize 的方式。

## SpEL 表达式

SpEL 在 @Value 里面的用法最常见，我们通过 @Value 来了解一下。

### @Value 的应用场景

新建一个 DemoProperties 对象，用 Spring 装载，测试一下两个语法点：运算符和 Map、List。

**第一个语法：通过 @Value 展示 SpEL 里面支持的各种运算符的写法。**如下面的表格所示。
![image-20210104092551998](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20210104092552.png)

**第二个语法：@Value 展示了 SpEL 可以直接读取 Map 和 List 里面的值**，代码如下所示。

```java
//我们通过@Component加载一个类，并且给其中的List和Map附上值
@Component("workersHolder")
public class WorkersHolder {
    private List<String> workers = new LinkedList<>();
    private Map<String, Integer> salaryByWorkers = new HashMap<>();
    public WorkersHolder() {
        workers.add("John");
        workers.add("Susie");
        workers.add("Alex");
        workers.add("George");
        salaryByWorkers.put("John", 35000);
        salaryByWorkers.put("Susie", 47000);
        salaryByWorkers.put("Alex", 12000);
        salaryByWorkers.put("George", 14000);
    }
    //Getters and setters ...
}
//SpEL直接读取Map和List里面的值
@Value("#{workersHolder.salaryByWorkers['John']}") // 35000
private Integer johnSalary;
@Value("#{workersHolder.salaryByWorkers['George']}") // 14000
private Integer georgeSalary;
@Value("#{workersHolder.salaryByWorkers['Susie']}") // 47000
private Integer susieSalary;
@Value("#{workersHolder.workers[0]}") // John
private String firstWorker;
@Value("#{workersHolder.workers[3]}") // George
private String lastWorker;
@Value("#{workersHolder.workers.size()}") // 4
private Integer numberOfWorkers;
```

以上就是 SpEL 的运算符和对 Map、List、SpringBeanFactory 里面的 Bean 的调用情况。

**@Value 使用的注意事项 # 与 $ 的区别**

SpEL 表达式默认以 # 开始，以大括号进行包住，如 #{expression}。默认规则在 ParserContext 里面设置，我们也可以自定义，但是一般建议不要动。

这里注意要与 Spring 中的 Properties 进行区别，Properties 相关的表达式是以 $ 开始的大括号进行包住的，如 ${property.name}。

也就是说 @Value 的值有两类：

```
${ property**:**default_value }

#{ obj.property**? :**default_value }
```

第一个注入的是外部参数对应的 Property，第二个则是 SpEL 表达式对应的内容。

而 Property placeholders 不能包含 SpEL 表达式，但是 SpEL 表达式可以包含 Property 的引用。如 #{${someProperty} + 2}，如果 someProperty=1，那么效果将是 #{ 1 + 2}，最终的结果将是 3。

### JPA 中 @Query 的应用场景

SpEL 除了能在 @Value 里面使用外，也能在 @Query 里使用，而在 @Query 里还有一个特殊的地方，就是它可以用来取方法的参数。

通过 SpEL 取被 @Query 注解的方法参数

在 @Query 注解中使用 SpEL 的主要目的是取方法的参数，主要有三种用法，如下所示。

```java
//用法一：根据下标取方法里面的参数
@Query("select u from User u where u.age = ?#{[0]}") 
List<User> findUsersByAge(int age);
//用法二：#customer取@Param("customer")里面的参数
@Query("select u from User u where u.firstname = :#{#customer.firstname}")
List<User> findUsersByCustomersFirstname(@Param("customer") Customer customer);
//用法三：用JPA约定的变量entityName取得当前实体的实体名字
@Query("from #{#entityName}")
List<UserInfo> findAllByEntityName();
```

其中，

- 方法一可以通过 [0] 的方式，根据下标取到方法的参数；
- 方法二通过 #customer 可以根据 @Param 注解的参数的名字取到参数，必须通过 ?#{} 和 :#{} 来触发 SpEL 的表达式语法；
- 方法三通过 #{#entityName} 取约定的实体的名字。

> 要注意区别@Param 的用法:lastname这种方式。

下面我们再来看一个更复杂一点的例子，代码如下。

```java
public interface UserInfoRepository extends JpaRepository<UserInfo, Long> {
    // JPA约定的变量entityName取得当前实体的实体名字
    @Query("from #{#entityName}")
    List<UserInfo> findAllByEntityName();
    //一个查询中既可以支持SpEL也可以支持普通的:ParamName的方式
    @Modifying
    @Query("update #{#entityName} u set u.name = :name where u.id =:id")
    void updateUserActiveState(@Param("name") String name, @Param("id") Long id);
    //演示SpEL根据数组下标取参数，和根据普通的Parma的名字:name取参数
    @Query("select u from UserInfo u where u.lastName like %:#{[0]} and u.name like %:name%")
    List<UserInfo> findContainingEscaped(@Param("name") String name);
    //SpEL取Parma的名字customer里面的属性
    @Query("select u from UserInfo u where u.name = :#{#customer.name}")
    List<UserInfo> findUsersByCustomersFirstname(@Param("customer") UserInfo customer);
    //利用SpEL根据一个写死的'jack'字符串作为参数
    @Query("select u from UserInfo u where u.name = ?#{'jack'}")
    List<UserInfo> findOliverBySpELExpressionWithoutArgumentsWithQuestionmark();
    //同时SpEL支持特殊函数escape和escapeCharacter
    @Query("select u from UserInfo u where u.lastName like %?#{escape([0])}% escape ?#{escapeCharacter()}")
    List<UserInfo> findByNameWithSpelExpression(String name);
    // #entityName和#[]同时使用
    @Query("select u from #{#entityName} u where u.name = ?#{[0]} and u.lastName = ?#{[1]}")
    List<UserInfo> findUsersByFirstnameForSpELExpressionWithParameterIndexOnlyWithEntityExpression(String name, String lastName);
    //对于 native SQL同样适用，并且同样支持取pageable分页里面的属性值
    @Query(value = "select * from (" //
           + "select u.*, rownum() as RN from (" //
           + "select * from user_info ORDER BY ucase(firstname)" //
           + ") u" //
           + ") where RN between ?#{ #pageable.offset +1 } and ?#{#pageable.offset + #pageable.pageSize}", //
           countQuery = "select count(u.id) from user_info u", //
           nativeQuery = true)
    Page<UserInfo> findUsersInNativeQueryWithPagination(Pageable pageable);
}
```

我个人比较推荐使用 @Param 的方式，这样语义清晰，参数换位置了也不影响执行结果。

### SpEL 在 @Cacheable 中的应用场景

我们在实际工作中还有一个经常用到 SpEL 的场景，就是在 Cache 的时候，也就是 Spring Cache 的相关注解里面，如 @Cacheable、@CachePut、@CacheEvict 等。我们还是通过例子来体会一下，代码如下所示。

```java
//缓存key取当前方法名，判断一下只有返回结果不为null或者非empty才进行缓存
@Cacheable(value = "APP", key = "#root.methodName", cacheManager = "redis.cache", unless = "#result == null || #result.isEmpty()")
@Override
public Map<String, Map<String, String>> getAppGlobalSettings() {}
//evict策略的key是当前参数customer里面的name属性
@Caching(evict = {
    @CacheEvict(value="directory", key="#customer.name") })
public String getAddress(Customer customer) {...}
//在condition里面使用，当参数里面customer的name属性的值等于字符串Tom才放到缓存里面
@CachePut(value="addresses", condition="#customer.name=='Tom'")
public String getAddress(Customer customer) {...}
//用在unless里面，利用SpEL的条件表达式判断，排除返回的结果地址长度小于64的请求
@CachePut(value="addresses", unless="#result.length()<64")
public String getAddress(Customer customer) {...}
```

**Spring Cache 中 SpEL 支持的上下文语法**

Spring Cache 提供了一些供我们使用的 SpEL 上下文数据，如下表所示（摘自 Spring 官方文档）。

![img](https://raw.githubusercontent.com/ZongweiBai/note-image/main/note/20210104093149.png)

## Spring Data JPA 单元测试最佳实践

测试用例写法主要依赖@DataJpaTest注解：

```java
@DataJpaTest
public class AddressRepositoryTest {
    @Autowired
    private AddressRepository addressRepository;
    //测试一下保存和查询
    @Test
    public  void testSave() {
        Address address = Address.builder().city("shanghai").build();
        addressRepository.save(address);
        List<Address> address1 = addressRepository.findAll();
        address1.stream().forEach(address2 -> System.out.println(address2));
    }
}
```

通过上面的测试用例可以看到，我们直接添加了 @DataJpaTest 注解，然后利用 Spring 的注解 @Autowired，引入了 spring context 里面管理的 AddressRepository 实例。换句话说，我们在这里面使用了集成测试，即直接连接的数据库来完成操作。

@DataJpaTest 注解帮我们做了很多事情：

1. 加载 Spring Data JPA 所需要的上下文，即数据库，所有的 Repository；
2. 启用默认集成数据库 h2，完成集成测试。

### 什么是单元测试

通俗来讲，就是不依赖本类之外的任何方法完成本类里面的所有方法的测试，也就是我们常说的依赖本类之外的，都通过 Mock 的方式进行。那么在单元测试的模式下，我们一起看看 Service 层的单元测试应该怎么写。

**Service 层单元测试**

单元测试写法如下。

```java
@ExtendWith(SpringExtension.class)//通过这个注解利用Spring的容器
@Import(UserInfoServiceImpl.class)//导入要测试的UserInfoServiceImpl
public class UserInfoServiceTest {
    @Autowired //利用spring的容器，导入要测试的UserInfoService
    private UserInfoService userInfoService;
    @MockBean //里面@MockBean模拟我们service中用到的userInfoRepository，这样避免真实请求数据库
    private UserInfoRepository userInfoRepository;
    // 利用单元测试的思想，mock userInfoService里面的UserInfoRepository，这样Service层就不用连接数据库，就可以测试自己的业务逻辑了
    @Test
    public void testGetUserInfoDto() {
        //利用Mockito模拟当调用findById(1)的时候，返回模拟数据
        Mockito.when(userInfoRepository.findById(1L)).thenReturn(java.util.Optional.ofNullable(UserInfo.builder().name("jack").id(1L).build()));
        UserInfoDto userInfoDto = userInfoService.findByUserId(1L);
        //经过一些service里面的逻辑计算，我们验证一下返回结果是否正确
        Assertions.assertEquals("jack",userInfoDto.getName());
    }
}
```

这样就可以完成了我们的 Service 层的测试了。

其中 @ExtendWith(SpringExtension.class) 是 spring boot 与 Junit 5 结合使用的时候，当利用 Spring 的 TesatContext 进行 mock 测试时要使用的。有的时候如果们做一些简单 Util 的测试，就不一定会用到 SpringExtension.class。

在 service 的单元测试中，主要用到的知识点有四个。

1. 通过 @ExtendWith(SpringExtension.class) 加载 Spring 的测试框架及其 TestContext；
2. 通过 @Import(UserInfoServiceImpl.class) 导入具体要测试的类，这样 SpringTestContext 就不用加载项目里面的所有类，只需要加载 UserInfoServiceImpl.class 就可以了，这样可以大大提高测试用例的执行速度；
3. 通过 @MockBean 模拟 UserInfoSerceImpl 依赖的 userInfoRepository，并且自动注入 Spring test context 里面，这样 Service 里面就自动有依赖了；
4. 利用 Mockito.when().thenReturn() 的机制，模拟测试方法。

这样我们就可以通过 Assertions 里面的断言来测试 serice 方法里面的逻辑是否符合预期了。

**Controller 层单元测试**

完整的测试用例，代码如下所示。

```java
package com.example.jpa.demo;

@WebMvcTest(UserInfoController.class)
public class UserInfoControllerTest {
    @Autowired
    private MockMvc mvc;
    @MockBean
    private UserInfoService userInfoService;
    //单元测试mvc的controller的方法
    @Test
    public void testGetUserDto() throws Exception {
        //利用@MockBean，当调用 userInfoService的findByUserId(1)的时候返回一个模拟的UserInfoDto数据
        Mockito.when(userInfoService.findByUserId(1L)).thenReturn(UserInfoDto.builder().name("jack").id(1L).build());
        //利用mvc验证一下Controller里面的解决是否OK
        MockHttpServletResponse response = mvc
            .perform(MockMvcRequestBuilders
                     .get("/user/1/")//请求的path
                     .accept(MediaType.APPLICATION_JSON)//请求的mediaType，这里面可以加上各种我们需要的Header
                    )
            .andDo(print())//打印一下
            .andExpect(status().isOk())
            .andExpect(MockMvcResultMatchers.jsonPath("$.name").value("jack"))
            .andReturn().getResponse();
        System.out.println(response);
    }
}
```

其中我们主要利用了 @WebMvcTest 注解，来引入我们要测试的 Controller。

当通过 @WebMvcTest(UserInfoController.class) 导入我们需要测试的 Controller 之后，就可以再通过 MockMvc 请求到我们加载的 Contoller 里面的 path 了，并且可以通过 MockMvc 提供的一些方法发送请求，验证 Controller 的响应结果。

下面概括一下 Contoller 层单元测试主要用到的三个知识点。

1. 利用 @WebMvcTest 注解，加载我们要测试的 Controller，同时生成 mvc 所需要的 Test Context；
2. 利用 @MockBean 默认 Controller 里面的依赖，如 Service，并通过 Mockito.when().thenReturn()；的语法 mock 依赖的测试数据；
3. 利用 MockMvc 中提供的方法，发送 Controller 的 Rest 风格的请求，并验证返回结果和状态码。

那么单元测试我们先介绍这么多，下面看一下什么是集成测试。

### 什么是集成测试

顾名思义，就是指多个模块放在一起测试，和单元测试正好相反，并非采用 mock 的方式测试，而是通过直接调用的方式进行测试。也就是说我们依赖 spring 容器进行开发，所有的类之间直接调用，模拟应用真实启动时候的状态。

**Service 层的集成测试用例写法**

我们还用刚才的例子，看一下 UserInfoService 里面的 findByUserId 通过集成测试如何进行。测试用例的写法如下。

```java
@DataJpaTest
@ComponentScan(basePackageClasses= UserInfoServiceImpl.class)
public class UserInfoServiceIntegrationTest {
    @Autowired
    private UserInfoService userInfoService;
    @Autowired
    private UserInfoRepository userInfoRepository;
    @Test
    @Rollback(false)//如果我们事务回滚设置成false的话，数据库可以真实看到这条数据
    public void testIntegtation() {
        UserInfo u1 = UserInfo.builder().name("jack-db").ages(20).id(1L).telephone("1233456").build();
        //数据库真实加一条数据
        userInfoRepository.save(u1);//数据库里面真实保存一条数据
        UserInfoDto userInfoDto =  userInfoService.findByUserId(1L);
        userInfoDto.getName();
        Assertions.assertEquals(userInfoDto.getName(),u1.getName()+"_HELLO");
    }
}
```

这时你会发现数据已经不再回滚，也会正常地执行 SQL，而不是通过 Mock 的方式测试。Service 的集成测试相对来说还比较简单，那么我们看下 Controller 层的集成测试用例应该怎么写。

**Controller 层的集成测试用例的写法**

我们用集成测试把刚才 UserInfoCotroller 写的 user/1/ 接口测试一下，将集成测试的代码做如下改动。

```java
@SpringBootTest(classes = DemoApplication.class,
                webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) //加载DemoApplication，指定一个随机端口
public class UserInfoControllerIntegrationTest {
    @LocalServerPort //获得模拟的随机端口
    private int port;
    @Autowired //我们利用RestTemplate，发送一个请求
    private TestRestTemplate restTemplate;
    @Test
    public void testAllUserDtoIntegration() {
        UserInfoDto userInfoDto = this.restTemplate
            .getForObject("http://localhost:" + port + "/user/1", UserInfoDto.class);//真实请求有一个后台的API
        Assertions.assertNotNull(userInfoDto);
    }
}
```

我们再看日志的话，会发现此次的测试用例会在内部启动一个 tomcat 容器，然后再利用 TestResTemplate 进行真实请求，返回测试结果进行测试。

而其中会涉及一个注解 @SpringBootTest，它用来指定 Spring 应用的类是哪个，也就是我们真实项目的 Application 启动类；然后会指定一个端口，此处必须使用随机端口，否则可能会有冲突（如果我们启动的集成测试有点多的情况）。

### Junit 4 和 Junit 5 在 Spring Boot 中的区别

第一，Spring Boot 2.2+ 以上的版本默认导入的是 Junit 5 的 jar 包依赖，以下的版本默认导入的是 Junit 4 的 jar 包依赖的版本，所以你在使用不同版本的 Spring Boot 的时候需要注意一下依赖的 jar 包是否齐全。

第二，org.junit.junit.Test 变成了 org.junit.jupiter.api.Test。

第三，一些注解发生了变化：

- @Before 变成了 @BeforeEach
- @After 变成了 @AfterEach
- @BeforeClass 变成了 @BeforeAll
- @AfterClass 变成了 @AfterAll
- @Ignore 变成了 @Disabled
- @Category 变成了 @Tag
- @Rule 和 @ClassRule 没有了，用 @ExtendWith 和 @RegisterExtension 代替

第四，引用 Spring 的上下文 @RunWith(SpringRunner.class) 变成了 @ExtendWith(SpringExtension.class)。

第五，org.junit.Assert 下面的断言都移到了org.junit.jupiter.api.Assertions 下面，所以一些断言的写法会发生如下变化：

```java
//junit4断言的写法
Assert.assertEquals(200, result.getStatusCodeValue());
Assert.assertEquals(true, result.getBody().contains("employeeList"));
//junit5断言的写法
Assertions.assertEquals(400, ex.getRawStatusCode());
Assertions.assertEquals(true, ex.getResponseBodyAsString().contains("Missing request header"));
```

第六，Junit 5 提供 @DisplayName("Test MyClass") 用来标识此次单元测试的名字。

## Spring Data ElasticSearch

Spring Data 和 Elasticsearch 结合的时候，唯一需要注意的是版本之间的兼容性问题，Elasticsearch 和 Spring Boot 是同时向前发展的，而 Elasticsearch 的大版本之间还存在一定的 API 兼容性问题，所以我们必须要知道这些版本之间的关系，我整理了一个表格，如下。

| **Spring Data Release Train**                                | **Spring Data Elasticsearch**                                | **Elasticsearch** | **Spring Boot**                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------- | ------------------------------------------------------------ |
| 2020.0.0[[1](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1)] | 4.1.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1)] | 7.9.3             | 2.4.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1)] |
| Neumann                                                      | 4.0.x                                                        | 7.6.2             | 2.3.x                                                        |
| Moore                                                        | 3.2.x                                                        | 6.8.12            | 2.2.x                                                        |
| Lovelace                                                     | 3.1.x                                                        | 6.2.2             | 2.1.x                                                        |
| Kay[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] | 3.0.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] | 5.5.0             | 2.0.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] |
| Ingalls[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] | 2.1.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] | 2.4.0             | 1.5.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] |

现在你对这些版本之间的关联关系有了一定印象，由于版本越新越便利，所以一般情况下我们直接采用最新的版本。

首先参考官方文档：https://github.com/elastic/helm-charts/tree/master/elasticsearch 安装ES，完成安装后就可以开始测试了。

**第一步：引入 `spring-boot-starter-data-elasticsearch` 依赖。**

**第二步：在 application.properties 里面新增 es 的连接地址，连接本地的 Elasticsearch**。

```properties
spring.data.elasticsearch.client.reactive.endpoints=127.0.0.1:9200
```

**第三步：新增一个 ElasticSearchConfiguration 的配置文件，主要是为了开启扫描的包**。

```java
package com.example.data.es.demo.es;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.elasticsearch.repository.config.EnableElasticsearchRepositories;
//利用@EnableElasticsearchRepositories注解指定Elasticsearch相关的Repository的包路径在哪里
@EnableElasticsearchRepositories(basePackages = "com.example.data.es.demo.es")
@Configuration
public class ElasticSearchConfiguration {
}
```

**第四步：我们新增一个 Topic 的 Document，它类似 JPA 里面的实体，用来保存和读取 Topic 的数据**，代码如下所示。

```java
package com.example.data.es.demo.es;

@Document(indexName = "topic")
//论坛主题信息
public class Topic {
    @Id
    private Long id;
    private String title;
    @Field(type = FieldType.Nested, includeInParent = true)
    private List<Author> authors;
}

package com.example.data.es.demo.es;
//作者信息
public class Author {
    private String name;
}
```

**第五步：新建一个 Elasticsearch 的 Repository，用来对 Elasticsearch 索引的增删改查**，代码如下所示。

```java
package com.example.data.es.demo.es;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import java.util.List;
//类似JPA一样直接操作Topic类型的索引
public interface TopicRepository extends ElasticsearchRepository<Topic,Long> {
    List<Topic> findByTitle(String title);
}
```

**第六步: 新建一个 Controller，对 Topic 索引进行查询和添加。**

```java
@RestController
public class TopicController {
    @Autowired
    private TopicRepository topicRepository;
    //查询topic的所有索引
    @GetMapping("topics")
    public List<Topic> query(@Param("title") String title) {
        return topicRepository.findByTitle(title);
    }
    //保存 topic索引
    @PostMapping("topics")
    public Topic create(@RequestBody Topic topic) {
        return topicRepository.save(topic);
    }
}
```

**第七步：发送一个添加和查询的请求测试一下**。

我们发送三个 POST 请求，添加三条索引，代码如下所示。

```json
POST /topics HTTP/1.1
Host: 127.0.0.1:8080
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: d9cc1f6c-24dd-17ff-f2e8-3063fa6b86fc
{
    "title":"jack",
    "id":2,
    "authors":[{
        "name":"jk1"
    },{
        "name":"jk2"
    }]
}
```

然后发送一个 get 请求，获得标题是 jack 的索引，如下面这行代码所示。

```
GET http://127.0.0.1:8080/topics?title=jack
```

得到如下结果。

```json
GET http://127.0.0.1:8080/topics?title=jack
HTTP/1.1 200 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Wed, 30 Dec 2020 15:12:16 GMT
Keep-Alive: timeout=60
Connection: keep-alive
  {
    "id": 1,
    "title": "jack",
    "authors": [
      {
        "name": "jk1"
      },
      {
        "name": "jk2"
      }
    ]
  },
  {
    "id": 3,
    "title": "jack",
    "authors": [
      {
        "name": "jk1"
      },
      {
        "name": "jk2"
      }
    ]
  },
  {
    "id": 2,
    "title": "jack",
    "authors": [
      {
        "name": "jk1"
      },
      {
        "name": "jk2"
      }
    ]
  }
]
Response code: 200; Time: 348ms; Content length: 199 bytes
Cannot preserve cookies, cookie storage file is included in ignored list:
> /Users/jack/Company/git_hub/spring-data-jpa-guide/2.3/elasticsearch-data/.idea/httpRequests/http-client.cookies
```

这时，一个完整的 Spring Data Elasticsearch 的例子就演示完了。其实你会发现，我们使用 Spring Data Elasticsearch 来操作 ES 相关的 API 的话，比我们直接写 Http 的 client 要简单很多，因为这里面帮我们封装了很多基础逻辑，省去了很多重复造轮子的过程。