# [SpringBoot之Bean之指定初始化顺序的若干姿势](https://segmentfault.com/a/1190000020873720)

本文将介绍几种可行的方式来控制 bean 之间的加载顺序

- 构造方法依赖
- @DependOn 注解
- BeanPostProcessor 扩展

## 环境搭建

我们的测试项目和上一篇博文公用一个项目环境，当然也可以建一个全新的测试项目，对应的配置如下：（文末有源码地址）

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.7</version>
    <relativePath/> <!-- lookup parent from update -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

## 初始化顺序指定

### 1. 构造方法依赖

这种可以说是最简单也是最常见的使用姿势，但是在使用时，需要注意循环依赖等问题

我们知道 bean 的注入方式之中，有一个就是通过构造方法来注入，借助这种方式，我们可以解决有优先级要求的 bean 之间的初始化顺序

比如我们创建两个 Bean，要求 CDemo2 在 CDemo1 之前被初始化，那么我们的可用方式

```java
@Component
public class CDemo1 {

    private String name = "cdemo 1";

    public CDemo1(CDemo2 cDemo2) {
        System.out.println(name);
    }
}

@Component
public class CDemo2 {

    private String name = "cdemo 1";

    public CDemo2() {
        System.out.println(name);
    }
}
```

实测输出结果如下，和我们预期一致

![img](https://segmentfault.com/img/remote/1460000020873724)

虽然这种方式比较直观简单，但是有几个限制

- 需要有注入关系，如 CDemo2 通过构造方法注入到 CDemo1 中，如果需要指定两个没有注入关系的 bean 之间优先级，则不太合适（比如我希望某个 bean 在所有其他的 Bean 初始化之前执行）
- 循环依赖问题，如过上面的 CDemo2 的构造方法有一个 CDemo1 参数，那么循环依赖产生，应用无法启动

另外一个需要注意的点是，在构造方法中，不应有复杂耗时的逻辑，会拖慢应用的启动时间

### 2. @DependOn 注解

这是一个专用于解决 bean 的依赖问题，当一个 bean 需要在另一个 bean 初始化之后再初始化时，可以使用这个注解

使用方式也比较简单了，下面是一个简单的实例 case

```java
@DependsOn("rightDemo2")
@Component
public class RightDemo1 {
    private String name = "right demo 1";

    public RightDemo1() {
        System.out.println(name);
    }
}

@Component
public class RightDemo2 {
    private String name = "right demo 2";

    public RightDemo2() {
        System.out.println(name);
    }
}
```

上面的注解放在 `RightDemo1` 上，表示`RightDemo1`的初始化依赖于`rightDemo2`这个 bean

![img](https://segmentfault.com/img/remote/1460000020873725)

在使用这个注解的时候，有一点需要特别注意，它能控制 bean 的实例化顺序，但是 bean 的初始化操作（如构造 bean 实例之后，调用`@PostConstruct`注解的初始化方法）顺序则不能保证，比如我们下面的一个实例，可以说明这个问题

```java
@DependsOn("rightDemo2")
@Component
public class RightDemo1 {
    private String name = "right demo 1";

    @Autowired
    private RightDemo2 rightDemo2;

    public RightDemo1() {
        System.out.println(name);
    }

    @PostConstruct
    public void init() {
        System.out.println(name + " _init");
    }
}

@Component
public class RightDemo2 {
    private String name = "right demo 2";

    @Autowired
    private RightDemo1 rightDemo1;

    public RightDemo2() {
        System.out.println(name);
    }

    @PostConstruct
    public void init() {
        System.out.println(name + " _init");
    }
}
```

注意上面的代码，虽然说有循环依赖，但是通过`@Autowired`注解方式注入的，所以不会导致应用启动失败，我们先看一下输出结果

![img](https://segmentfault.com/img/remote/1460000020873726)

有意思的地方来了，我们通过`@DependsOn`注解来确保在创建`RightDemo1`之前，先得创建`RightDemo2`；

所以从构造方法的输出可以知道，先实例 RightDemo2, 然后实例 RightDemo1；

然后从初始化方法的输出可以知道，在上面这个场景中，虽然 RightDemo2 这个 bean 创建了，但是它的初始化代码在后面执行

> 题外话：
> 有兴趣的同学可以试一下把上面测试代码中的`@Autowired`的依赖注入删除，即两个 bean 没有相互注入依赖，再执行时，会发现输出顺序又不一样

### 3. BeanPostProcessor

最后再介绍一种非典型的使用方式，如非必要，请不要用这种方式来控制 bean 的加载顺序

先创建两个测试 bean

```java
@Component
public class HDemo1 {
    private String name = "h demo 1";

    public HDemo1() {
        System.out.println(name);
    }
}

@Component
public class HDemo2 {
    private String name = "h demo 2";

    public HDemo2() {
        System.out.println(name);
    }
}
```

我们希望 HDemo2 在 HDemo1 之前被加载，借助 BeanPostProcessor，我们可以按照下面的方式来实现

```java
@Component
public class DemoBeanPostProcessor extends InstantiationAwareBeanPostProcessorAdapter implements BeanFactoryAware {
    private ConfigurableListableBeanFactory beanFactory;
    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
        if (!(beanFactory instanceof ConfigurableListableBeanFactory)) {
            throw new IllegalArgumentException(
                    "AutowiredAnnotationBeanPostProcessor requires a ConfigurableListableBeanFactory: " + beanFactory);
        }
        this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
    }

    @Override
    @Nullable
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        // 在bean实例化之前做某些操作
        if ("HDemo1".equals(beanName)) {
            HDemo2 demo2 = beanFactory.getBean(HDemo2.class);
        }
        return null;
    }
}
```

请将目标集中在`postProcessBeforeInstantiation`，这个方法在某个 bean 的实例化之前，会被调用，这就给了我们控制 bean 加载顺序的机会

![img](https://segmentfault.com/img/remote/1460000020873727)

看到这种骚操作，是不是有点蠢蠢欲动，比如我有个 bean，希望在应用启动之后，其他的 bean 实例化之前就被加载，用这种方式是不是也可以实现呢?

下面是一个简单的实例 demo，重写`DemoBeanPostProcessor`的`postProcessAfterInstantiation`方法，在 application 创建之后，就加载我们的 FDemo 这个 bean

```java
@Override
public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
    if ("application".equals(beanName)) {
        beanFactory.getBean(FDemo.class);
    }

    return true;
}


@DependsOn("HDemo")
@Component
public class FDemo {
    private String name = "F demo";

    public FDemo() {
        System.out.println(name);
    }
}

@Component
public class HDemo {
    private String name = "H demo";

    public HDemo() {
        System.out.println(name);
    }
}
```

从下图输出可以看出，`HDemo`, `FDemo`的实例化顺序放在了最前面了

![img](https://segmentfault.com/img/remote/1460000020873728)

### 4. 小结

在小结之前，先指出一下，一个完整的 bean 创建，在本文中区分了两块顺序

- 实例化 （调用构造方法）
- 初始化 （注入依赖属性，调用`@PostConstruct`方法）

本文主要介绍了三种方式来控制 bean 的加载顺序，分别是

- 通过构造方法依赖的方式，来控制有依赖关系的 bean 之间初始化顺序，但是需要注意循环依赖的问题
- `@DependsOn`注解，来控制 bean 之间的实例顺序，需要注意的是 bean 的初始化方法调用顺序无法保证
- BeanPostProcessor 方式，来手动控制 bean 的加载顺序

## 其他