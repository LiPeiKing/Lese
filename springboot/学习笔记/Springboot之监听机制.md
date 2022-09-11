# Springboot之监听机制

## ApplicationListener 扩展

`ApplicationListener` 其实是 `spring` 事件通知机制中核心概念；在java的事件机制中，一般会有三个概念：

- event object : 事件对象
- event source ：事件源，产生事件的地方
- event listener ：监听事件并处理

`ApplicationListener` 继承自 `java.util.EventListener` ，提供了对于`Spring`中事件机制的扩展。

`ApplicationListener` 在实际的业务场景中使用的非常多，比如我一般喜欢在容器初始化完成之后来做一些资源载入或者一些组件的初始化。这里的容器指的就是`Ioc`容器，对应的事件是`ContextRefreshedEvent` 。

```java
@Component
public class StartApplicationListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent
    contextRefreshedEvent) {
       //初始化资源文件
       //初始化组件 如：cache
    }
}
```

上面这段代码会在容器刷新完成之后来做一些事情。下面通过自定义事件来看看怎么使用，在看具体的`demo`之前，先来了解下一些关注点。

日常工作了，如果要使用 `Spring` 事件传播机制，我们需要关注的点有以下几点：

- 事件类，这个用来描述事件本身一些属性，一般继承`ApplicationEvent`
- 监听类，用来监听具体的事件并作出响应。需要实现 `ApplicationListener` 接口
- 事件发布类，需要通过这个类将时间发布出去，这样才能被监听者监听到，需要实现`ApplicationContextAware`接口。
- 将事件类和监听类交给`Spring`容器。

那么下面就按照这个思路来看下`demo`的具体实现。

### 事件类：UserRegisterEvent

`UserRegisterEvent` ，用户注册事件；这里作为事件对象，继承自 `ApplicationEvent`。

```java
public class UserRegisterEvent extends ApplicationEvent {

    public String name;

    public UserRegisterEvent(Object o) {
        super(o);
    }

    public UserRegisterEvent(Object o, String name) {
        super(o);
        this.name=name;
    }
}
```

### 事件发布类：UserService

用户注册服务，这里需要在用户注册时将注册事件发布出去，所以通过实现`ApplicationEventPublisherAware`接口，使`UserService`具有事件发布能力。

> ApplicationEventPublisherAware:发布事件，也就是把某个事件告诉的所有与这个事件相关的监听器。

```java
@Service
public class UserService implements ApplicationEventPublisherAware {

    private ApplicationEventPublisher applicationEventPublisher;

    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }

    public void register(String name) {
        System.out.println("用户：" + name + " 已注册！");
        applicationEventPublisher.publishEvent(new UserRegisterEvent(name));
    }
}
```

这里的`UserService`实际上是作为事件源存在的，通过`register`将用户注册事件传播出去。那么下面就是需要定义如何来监听这个事件，并且将事件进行消费处理掉，这里就是通过`ApplicationListener`来完成。

### 监听类：BonusServerListener

当用户触发注册操作时，向积分服务发送消息，为用户初始化积分。

```java
@Component
public class BonusServerListener implements ApplicationListener<UserRegisterEvent> {
    public void onApplicationEvent(UserRegisterEvent event) {
        System.out.println("积分服务接到通知，给 " + event.getSource() + " 增加积分...");
    }
}
```



`Spring` 的事件传播机制是基于观察者模式（`Observer`）实现的，它可以将 `Spring Bean`的改变定义为事件 `ApplicationEvent`，通过 `ApplicationListener` 监听 `ApplicationEvent` 事件，一旦`Spring Bean` 使用 `ApplicationContext.publishEvent( ApplicationEvent event )`发布事件后，`Spring` 容器会通知注册在 容器中所有 `ApplicationListener` 接口的实现类，最后 `ApplicationListener` 接口实现类判断是否处理刚发布出来的 `ApplicationEvent` 事件。

## ApplicationContextAware 扩展

`ApplicationContextAware`中只有一个`setApplicationContext`方法。实现了`ApplicationContextAware`接口的类，可以在该`Bean`被加载的过程中获取`Spring`的应用上下文`ApplicationContext`，通过`ApplicationContext`可以获取 `Spring`容器内的很多信息。

这种一般在需要手动获取`Bean`的注入实例对象时会使用到。下面通过一个简单的`demo`来了解下。

`GlmapperApplicationContext` 持有`ApplicationContext`对象，通过实现 `ApplicationContextAware`接口来给`ApplicationContext`做赋值。

```java
@Component
public class GlmapperApplicationContext implements ApplicationContextAware {

    private  ApplicationContext applicationContext;
    public void setApplicationContext(ApplicationContext
    applicationContext) throws BeansException {
            this.applicationContext=applicationContext;
    }

    public ApplicationContext getApplicationContext(){
        return applicationContext;
    }
}
```

需要手动获取的`bean`:

```java
@service
public class HelloService {
    public void sayHello(){
        System.out.println("Hello Glmapper");
    }
}
```

客户端类调用：

```java
public class MainTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        
        HelloService helloService = (HelloService) context.getBean("helloService");
        helloService.sayHello();

        //这里通过实现ApplicationContextAware接口的类来完成bean的获取
        GlmapperApplicationContext glmapperApplicationContext = (GlmapperApplicationContext) context.getBean("glmapperApplicationContext");
        
        ApplicationContext applicationContext = glmapperApplicationContext.getApplicationContext();
        
        HelloService glmapperHelloService = (HelloService) applicationContext.getBean("helloService");
        
        glmapperHelloService.sayHello();
    }
}
```

## BeanFactoryAware 扩展

我们知道`BeanFactory`是整个`Ioc`容器最顶层的接口，它规定了容器的基本行为。实现`BeanFactoryAware`接口就表明当前类具体`BeanFactory`的能力。

`BeanFactoryAware`接口中只有一个`setBeanFactory`方法。实现了`BeanFactoryAware`接口的类，可以在该`Bean`被加载的过程中获取加载该`Bean`的`BeanFactory`，同时也可以获取这个`BeanFactory`中加载的其它`Bean`。

来想一个问题，我们为什么需要通过`BeanFactory`的`getBean`来获取`Bean`呢？Spring已经提供了很多便捷的注入方式，那么通过`BeanFactory`的`getBean`来获取`Bean`有什么好处呢？来看一个场景。

现在有一个`HelloService`，这个`HelloService`就是打招呼，我们需要通过不同的语言来实现打招呼，比如用中文，用英文。一般的做法是：

```java
public interface HelloService {
    void sayHello();
}

//英文打招呼实现
@Service
public class GlmapperHelloServiceImpl implements HelloService {
    public void sayHello() {
        System.out.println("Hello Glmapper");
    }
}

//中文打招呼实现
@Service
public class LeishuHelloServiceImpl implements HelloService {
    public void sayHello() {
        System.out.println("你好，磊叔");
    }
}
```

客户端类来调用务必会出现下面的方式：

```java
if (condition=="英文"){
    glmapperHelloService.sayHello();
}
if (condition=="中文"){
    leishuHelloService.sayHello();
}
```

如果有一天，老板说我们要做国际化，要实现全球所有的语言来问候。你是说好的，还是控制不住要动手呢？

那么有没有什么方式可以动态的去决定我的客户端类到底去调用哪一种语言实现，而不是用过if-else方式来罗列呢？是的，对于这些需要动态的去获取对象的场景，`BeanFactoryAware`就可以很好的搞定。OK，来看代码改造：

引入`BeanFactoryAware`：

```java
/**
 * @description: 实现BeanFactoryAware ，让当前bean本身具有 BeanFactory 的能力
 *
 * 实现 BeanFactoηAware 接口的 bean 可以直接访问 Spring 容器，被容器创建以后，
 * 它会拥有一个指向 Spring
 容器的引用，可以利用该bean根据传入参数动态获取被spring工厂加载的bean
 *
 */
public class GlmapperBeanFactory implements BeanFactoryAware {

    private BeanFactory beanFactory;

    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory=beanFactory;
    }
    /**
     * 提供一个execute 方法来实现不同业务实现类的调度器方案。
     * @param beanName
     */
    public void execute(String beanName){
        HelloService helloService=(HelloService) beanFactory.getBean(beanName);
        helloService.sayHello();
    }

}
```

这里为了逻辑方便理解，再加入一个`HelloFacade` 类,这个类的作用就是持有一个`BeanFactoryAware`的实例对象，然后通过`HelloFacade`实例对象的方法来屏蔽底层`BeanFactoryAware`实例的实现细节。

```java
public class HelloFacade {
    private GlmapperBeanFactory glmapperBeanFactory;
    //调用glmapperBeanFactory的execute方法
    public void sayHello(String beanName){
        glmapperBeanFactory.execute(beanName);
    }
    public void setGlmapperBeanFactory(GlmapperBeanFactory beanFactory){
        this.glmapperBeanFactory = beanFactory;
    }
}
```

客户端类:

```java
public class MainTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        
        HelloFacade helloFacade = (HelloFacade) context.getBean("helloFacade");

        GlmapperBeanFactory glmapperBeanFactory = (GlmapperBeanFactory) context.getBean("glmapperBeanFactory");
        
        //这里其实可以不通过set方法注入到helloFacade中，
        //可以在helloFacade中通过autowired
        //注入；这里在使用main方法来执行验证，所以就手动set进入了
        helloFacade.setGlmapperBeanFactory(glmapperBeanFactory);

        //这个只需要传入不同HelloService的实现类的beanName，
        //就可以执行不同的业务逻辑
        helloFacade.sayHello("glmapperHelloService");
        helloFacade.sayHello("leishuHelloService");

    }
}
```

可以看到在调用者（客户端）类中，只需要通过一个`beanName`就可以实现不同实现类的切换，而不是通过一堆if-else来判断。另外有的小伙伴可能会说，程序怎么知道用哪个`beanName`呢？其实这个也很简单，这个参数我们可以通过一些途径来拼接得到，比如使用一个`prefix`用来指定语言，`prefix`+`HelloService`就可以确定唯一的`beanName`。





## 事件机制

> 在 SpringBoot 的启动过程中，会通过 SPI 机制去加载 spring.factories 下面的一些类，这里面就包括了事件相关的类。

- SpringApplicationRunListener

```properties
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

- ApplicationListener

```properties
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
```

`SpringApplicationRunListener` 类是 `SpringBoot` 中新增的类。`SpringApplication` 类中使用它们来间接调用 `ApplicationListener`。另外还有一个新增的类是`SpringApplicationRunListeners`，`SpringApplicationRunListeners` 中包含了多个 `SpringApplicationRunListener`。



### 三种监听器的关系

ApplicationListener、SpringApplicationRunListeners、SpringApplicationRunListener的关系：

- SpringApplicationRunListeners类和SpringApplicationRunListener类是SpringBoot中新增的类。ApplicationListener是spring中框架的类。
- 在SpringBoot（SpringApplication类）中，使用SpringApplicationRunListeners、SpringApplicationRunListener来间接调用ApplicationListener。
- SpringApplicationRunListeners是SpringApplicationRunListener的封装，SpringApplicationRunListeners中包含多个SpringApplicationRunListener，是为了批量执行的封装，SpringApplicationRunListeners与SpringApplicationRunListener生命周期相同，调用每个周期的各个SpringApplicationRunListener。然后广播相应的事件到Spring框架的ApplicationListener。

通过上面的3个特点可以看出SpringApplicationRunListener就是一个ApplicationListener的代理。springboot启动的几个主要过程的监听通知都是通过他来进行回调。

### SpringApplicationRunListener

从命名我们就可以知道它是一个监听者，那纵观整个启动流程我们会发现，它其实是用来在整个启动流程中接收不同执行点事件通知的监听者，SpringApplicationRunListener接口规定了SpringBoot的生命周期，在各个生命周期广播相应的事件，调用实际的ApplicationListener类。

源码如下：

```java
public interface SpringApplicationRunListener {
	//当run方法首次启动时立即调用。可用于非常早期的初始化。对应事件ApplicationStartingEvent
	void starting();
    
	//在准备好环境后，但在创建ApplicationContext之前调用。对应事件ApplicationEnvironmentPreparedEvent
	void environmentPrepared(ConfigurableEnvironment environment);
    
	//在创建和准备好ApplicationContext之后，但在加载源之前调用。对应事件ApplicationContextInitializedEvent
	void contextPrepared(ConfigurableApplicationContext context);
    
	//在加载应用程序上下文后但刷新之前调用。对应事件ApplicationPreparedEvent
	void contextLoaded(ConfigurableApplicationContext context);
    
	//上下文已刷新，应用程序已启动，但尚未调用commandlinerunner和applicationrunner。对应事件ApplicationStartedEvent
	void started(ConfigurableApplicationContext context);
    
	//在运行应用程序时发生故障时调用。2.0 才有。对应事件ApplicationFailedEvent
	void failed(ConfigurableApplicationContext context, Throwable exception);
}
```

### SpringApplicationRunListeners

上面提到，`SpringApplicationRunListeners` 是`SpringApplicationRunListener`的集合，里面包括了很多`SpringApplicationRunListener`实例；`SpringApplication` 类实际上使用的是 `SpringApplicationRunListeners` 类，与 `SpringApplicationRunListener` 生命周期相同，调用各个周期的 `SpringApplicationRunListener` 。然后广播相应的事件到 `ApplicationListener`。

> 代码详见：[SpringApplicationRunListeners](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fspring-projects%2Fspring-boot%2Fblob%2Fmaster%2Fspring-boot-project%2Fspring-boot%2Fsrc%2Fmain%2Fjava%2Forg%2Fspringframework%2Fboot%2FSpringApplicationRunListeners.java).

#### 实现类 EventPublishingRunListener类

`EventPublishingRunListener` 类是 `SpringApplicationRunListener`接口的实现类 ，它具有广播事件的功能。其内部使用 `ApplicationEventMulticaster`在实际刷新上下文之前发布事件。

```java
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {

    private final SpringApplication application;

    private final String[] args;

    private final SimpleApplicationEventMulticaster initialMulticaster;

    public EventPublishingRunListener(SpringApplication application, String[] args) {
        this.application = application; 
        this.args = args;        //新建立广播器
        this.initialMulticaster = new SimpleApplicationEventMulticaster();
        for (ApplicationListener<?> listener : application.getListeners()) {
            this.initialMulticaster.addApplicationListener(listener);
        }
    }
    
    @Override
	public void starting(ConfigurableBootstrapContext bootstrapContext) {
		this.initialMulticaster
				.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
	}

	@Override
	public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
			ConfigurableEnvironment environment) {
		this.initialMulticaster.multicastEvent(
				new ApplicationEnvironmentPreparedEvent(bootstrapContext, this.application, this.args, environment));
	}

	@Override
	public void contextPrepared(ConfigurableApplicationContext context) {
		this.initialMulticaster
				.multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
	}

	@Override
	public void contextLoaded(ConfigurableApplicationContext context) {
		for (ApplicationListener<?> listener : this.application.getListeners()) {
			if (listener instanceof ApplicationContextAware) {
				((ApplicationContextAware) listener).setApplicationContext(context);
			}
			context.addApplicationListener(listener);
		}
		this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
	}

	@Override
	public void started(ConfigurableApplicationContext context, Duration timeTaken) {
		context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
		AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
	}

	@Override
	public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
		context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context, timeTaken));
		AvailabilityChangeEvent.publish(context, ReadinessState.ACCEPTING_TRAFFIC);
	}

	@Override
	public void failed(ConfigurableApplicationContext context, Throwable exception) {
		ApplicationFailedEvent event = new ApplicationFailedEvent(this.application, this.args, context, exception);
		if (context != null && context.isActive()) {
			// Listeners have been registered to the application context so we should
			// use it at this point if we can
			context.publishEvent(event);
		}
		else {
			// An inactive context may not have a multicaster so we use our multicaster to
			// call all of the context's listeners instead
			if (context instanceof AbstractApplicationContext) {
				for (ApplicationListener<?> listener : ((AbstractApplicationContext) context)
						.getApplicationListeners()) {
					this.initialMulticaster.addApplicationListener(listener);
				}
			}
			this.initialMulticaster.setErrorHandler(new LoggingErrorHandler());
			this.initialMulticaster.multicastEvent(event);
		}
	}

```

从上面代码可以看出，它使用了Spring广播器SimpleApplicationEventMulticaster。它把监听的过程封装成了SpringApplicationEvent事件并让内部属性(属性名为multicaster)ApplicationEventMulticaster接口的实现类SimpleApplicationEventMulticaster广播出去，广播出去的事件对象会被SpringApplication中的listeners属性进行处理。

**补充：SimpleApplicationEventMulticaster**

类结构分析:     

![img](https://pic3.zhimg.com/80/v2-7d50fc9268375c5549888f0de5c1bfbe_720w.jpg)

`SimpleApplicationEventMulticaster`是 `Spring` 默认的事件广播器。实现了 ApplicationEventMulticaster 应用事件广播器接口, 并继承 AbstractApplicationEventMulticaster 抽象应用事件广播器类.

先分析：AbstractApplicationEventMulticaster 实现了三个接口，分别为 ApplicationEventMulticaster， BeanClassLoaderAware， BeanFactoryAware 会初始化类加载器及bean容器。

```java
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
    for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
        Executor executor = getTaskExecutor();
        if (executor != null) {
            executor.execute(() -> invokeListener(listener, event));
        }
        else {
            invokeListener(listener, event);
        }
    }
} 
protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
    ErrorHandler errorHandler = getErrorHandler();
    if (errorHandler != null) {
        try {
            doInvokeListener(listener, event);
        }
        catch (Throwable err) {
            errorHandler.handleError(err);
        }
    }
    else {
        doInvokeListener(listener, event);
    }
}
private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
		try {
            // 调用实现类中的方法
			listener.onApplicationEvent(event);
		}
		catch (ClassCastException ex) {
			String msg = ex.getMessage();
			if (msg == null || matchesClassCastMessage(msg, event.getClass()) ||
					(event instanceof PayloadApplicationEvent &&
							matchesClassCastMessage(msg, ((PayloadApplicationEvent) event).getPayload().getClass()))) {
				// Possibly a lambda-defined listener which we could not resolve the generic event type for
				// -> let's suppress the exception.
				Log loggerToUse = this.lazyLogger;
				if (loggerToUse == null) {
					loggerToUse = LogFactory.getLog(getClass());
					this.lazyLogger = loggerToUse;
				}
				if (loggerToUse.isTraceEnabled()) {
					loggerToUse.trace("Non-matching event type for listener: " + listener, ex);
				}
			}
			else {
				throw ex;
			}
		}
	}
```

从上面的代码段可以看出，它是通过遍历注册的每个监听器，并启动来调用每个监听器的 `onApplicationEvent` 方法。

下面来看下 `EventPublishingRunListener` 类生命周期对应的事件。



##### ApplicationStartingEvent

`ApplicationStartingEvent` 是 `SpringBoot` 启动开始的时候执行的事件，在该事件中可以获取到 `SpringApplication` 对象，可做一些执行前的设置，对应的调用方法是 `starting()`。

##### ApplicationEnvironmentPreparedEvent

`ApplicationEnvironmentPreparedEvent` 是`SpringBoot` 对应 `Enviroment` 已经准备完毕时执行的事件，此时上下文 `context` 还没有创建。在该监听中获取到 `ConfigurableEnvironment` 后可以对配置信息做操作，例如：修改默认的配置信息，增加额外的配置信息等。对应的生命周期方法是 `environmentPrepared(environment)`；`SpringCloud` 中，引导上下文就是在这时初始化的。

##### ApplicationContextInitializedEvent

当 `SpringApplication` 启动并且准备好 `ApplicationContext`，并且在加载任何 `bean` 定义之前调用了 `ApplicationContextInitializers` 时发布的事件。对应的生命周期方法是`contextPrepared()`

##### ApplicationPreparedEvent

`ApplicationPreparedEvent` 是`SpringBoot`上下文 `context` 创建完成是发布的事件；但此时 `spring` 中的 `bean` 还没有完全加载完成。这里可以将上下文传递出去做一些额外的操作。但是在该监听器中是无法获取自定义 `bean` 并进行操作的。对应的生命周期方法是 `contextLoaded()`。

##### ApplicationStartedEvent

这个事件是在 2.0 版本才引入的；具体发布是在应用程序上下文刷新之后，调用任何 `ApplicationRunner` 和 `CommandLineRunner` 运行程序之前。

##### ApplicationReadyEvent

这个和 `ApplicationStartedEvent` 很类似，也是在应用程序上下文刷新之后之后调用，区别在于此时`ApplicationRunner` 和 `CommandLineRunner`已经完成调用了，也意味着 `SpringBoot` 加载已经完成。

##### ApplicationFailedEvent

`SpringBoot` 启动异常时执行的事件，在异常发生时，最好是添加虚拟机对应的钩子进行资源的回收与释放，能友善的处理异常信息。

 

所以说SpringApplicationRunListener和ApplicationListener之间的关系是通过ApplicationEventMulticaster广播出去的SpringApplicationEvent所联系起来的。更多的spring事件知识见《[ApplicationEvent事件机制源码分析](https://www.cnblogs.com/duanxz/p/4341651.html)》

![img](https://img2018.cnblogs.com/blog/285763/201907/285763-20190725102547250-682035405.png)

### **如何扩展**

对于开发者来说，基本没有什么常见的场景要求我们必须实现一个自定义的SpringApplicationRunListener，即使是SpringBoot中也只默认实现了一个`org.springframework.boot.context.eventEventPublishingRunListener`, 用来在SpringBoot的整个启动流程中的不同时间点发布不同类型的应用事件(SpringApplicationEvent)。那些对这些应用事件感兴趣的ApplicationListener可以接受并处理(这也解释了为什么在SpringApplication实例化的时候加载了一批ApplicationListener，但在run方法执行的过程中并没有被使用)。

　　如果我们真的在实际场景中自定义实现SpringApplicationRunListener,有一个点需要注意：**任何一个SpringApplicationRunListener实现类的构造方法都需要有两个构造参数，一个参数的类型就是我们的org.springframework.boot.SpringApplication,另外一个参数就是args参数列表的String[]**:

```
package com.dxz.controller;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringApplicationRunListener;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;

public class SampleSpringApplicationRunListener implements SpringApplicationRunListener {
    private final SpringApplication application;
    private final String[] args;

    public SampleSpringApplicationRunListener(SpringApplication sa, String[] args) {
        this.application = sa;
        this.args = args;
    }

    @Override
    public void starting() {
        System.out.println("自定义starting");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        System.out.println("自定义environmentPrepared");
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("自定义contextPrepared");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("自定义contextLoaded");
    }

    @Override
    public void finished(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("自定义finished");
    }
}
```

接着，我们还要满足SpringFactoriesLoader的约定，在当前SpringBoot项目的classpath下新建META-INF目录，并在该目录下新建spring.fatories文件，文件内容如下:

```properties
org.springframework.boot.SpringApplicationRunListener=\
    com.dxz.SampleSpringApplicationRunListener
```

### demo 及各个事件的执行顺序

下面的各个事件对应的demo及打印出来的执行顺序。

- GlmapperApplicationStartingEventListener

```typescript
public class GlmapperApplicationStartingEventListener implements ApplicationListener<ApplicationStartingEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("execute ApplicationStartingEvent ...");
    }
}
复制代码
```

- GlmapperApplicationEnvironmentPreparedEvent

```typescript
public class GlmapperApplicationEnvironmentPreparedEvent implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {
    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent applicationEnvironmentPreparedEvent) {
        System.out.println("execute ApplicationEnvironmentPreparedEvent ...");
    }
}
复制代码
```

- GlmapperApplicationContextInitializedEvent

```typescript
public class GlmapperApplicationContextInitializedEvent implements ApplicationListener<ApplicationContextInitializedEvent> {
    @Override
    public void onApplicationEvent(ApplicationContextInitializedEvent applicationContextInitializedEvent) {
        System.out.println("execute applicationContextInitializedEvent ...");
    }
}
复制代码
```

- GlmapperApplicationPreparedEvent

```typescript
public class GlmapperApplicationPreparedEvent implements ApplicationListener<ApplicationPreparedEvent> {
    @Override
    public void onApplicationEvent(ApplicationPreparedEvent applicationPreparedEvent) {
        System.out.println("execute ApplicationPreparedEvent ...");
    }
}
复制代码
```

- GlmapperApplicationStartedEvent

```typescript
public class GlmapperApplicationStartedEvent implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent applicationStartedEvent) {
        System.out.println("execute ApplicationStartedEvent ...");
    }
}
复制代码
```

- GlmapperApplicationReadyEvent

```typescript
public class GlmapperApplicationReadyEvent implements ApplicationListener<ApplicationReadyEvent> {
    @Override
    public void onApplicationEvent(ApplicationReadyEvent applicationReadyEvent) {
        System.out.println("execute ApplicationReadyEvent ...");
    }
}
复制代码
```

- 执行结果



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/1680822d81776a67~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)







## 事件体系的初始化

事件体系的初始化对应在 `SpringBoot`启动过程的 `refreshContext`这个方法；`refreshContext`具体调用 AbstractApplicationContext.refresh()方法，最后调用 initApplicationEventMulticaster() 来完成事件体系的初始化,代码如下：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/1680835fb264f84a~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)



用户可以为容器定义一个自定义的事件广播器，只要实现 `ApplicationEventMulticaster` 就可以了，`Spring` 会通过 反射的机制将其注册成容器的事件广播器，如果没有找到配置的外部事件广播器，`Spring` 就是默认使用 `SimpleApplicationEventMulticaster` 作为事件广播器。

## 事件注册

事件注册是在事件体系初始化完成之后做的事情，也是在 `AbstractApplicationContext.refresh()` 方法中进行调用的。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/16808437ea8623d2~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

这里干了三件事：



- 首先注册静态指定的 `listeners`；这里包括我们自定义的那些监听器。
- 调用 `DefaultListableBeanFactory` 中 `getBeanNamesForType` 得到自定义的 `ApplicationListener` `bean` 进行事件注册。
- 广播早期的事件

## 事件广播

事件发布伴随着 `SpringBoot` 启动的整个生命周期。不同阶段对应发布不同的事件，上面我们已经对各个事件进行了分析，下面就具体看下发布事件的实现：

> org.springframework.context.support.AbstractApplicationContext#publishEvent
>
> ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/168084a716c01856~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

> earlyApplicationEvents 中的事件是广播器未建立的时候保存通知信息，一旦容器建立完成，以后都是直接通知。

广播事件最终还是通过调用 `ApplicationEventMulticaster` 的 `multicastEvent` 来实现。而 `multicastEvent` 也就就是事件执行的方法。



## SpringBoot 启动过程中的事件阶段

这里回到 `SpringApplication`的`run`方法，看下 `SpringBoot` 在启动过程中，各个事件阶段做了哪些事情。

### starting -> ApplicationStartingEvent

这里 `debug` 到 `starting` 方法，追踪到 `multicastEvent`，这里 `type`为 `ApplicationStartingEvent`；对应的事件如下：

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/16808657f6db440c~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)



- LoggerApplicationListener：配置日志系统。使用`logging.config`环境变量指定的配置或者缺省配置
- BackgroundPreinitializer：尽早触发一些耗时的初始化任务，使用一个后台线程
- DelegatingApplicationListener：监听到事件后转发给环境变量`context.listener.classes`指定的那些事件监听器
- LiquibaseServiceLocatorApplicationListener：使用一个可以和 `SpringBoot` 可执行`jar`包配合工作的版本替换 `liquibase ServiceLocator`

### listeners.environmentPrepared->ApplicationEnvironmentPreparedEvent



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/16808703538f4265~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)



- AnsiOutputApplicationListener：根据`spring.output.ansi.enabled`参数配置`AnsiOutput`

- ConfigFileApplicationListener：`EnvironmentPostProcessor`，从常见的那些约定的位置读取配置文件，比如从以下目录读取`application.properties`,`application.yml`等配置文件：

  - classpath:
  - file:.
  - classpath:config
  - file:./config/

  也可以配置成从其他指定的位置读取配置文件。

- ClasspathLoggingApplicationListener：对环境就绪事件`ApplicationEnvironmentPreparedEvent`/应用失败事`件ApplicationFailedEvent`做出响应，往日志`DEBUG`级别输出`TCCL(thread context class loader)`的 `classpath`。

- FileEncodingApplicationListener：如果系统文件编码和环境变量中指定的不同则终止应用启动。具体的方法是比较系统属性`file.encoding`和环境变量`spring.mandatory-file-encoding`是否相等(大小写不敏感)。

### listeners.contextPrepared->ApplicationContextInitializedEvent



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/1680877bed3cfbbd~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

相关监听器参考上面的描述。



### listeners.contextLoaded->ApplicationPreparedEvent



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/168087a73a7666bd~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

相关监听器参考上面的描述。



### refresh->ContextRefreshedEvent



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/168087bc9054d773~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)



- ConditionEvaluationReportLoggingListener：实际上实现的是 `ApplicationContextInitializer`接口，其目的是将 `ConditionEvaluationReport` 写入到日志，使用`DEBUG`级别输出。程序崩溃报告会触发一个消息输出，建议用户使用调试模式显示报告。它是在应用初始化时绑定一个`ConditionEvaluationReportListener`事件监听器，然后相应的事件发生时输出`ConditionEvaluationReport`报告。
- ClearCachesApplicationListener：应用上下文加载完成后对缓存做清除工作，响应事件`ContextRefreshedEvent`。
- SharedMetadataReaderFactoryContextInitializer： 向`context`注册了一个`BeanFactoryPostProcessor`：`CachingMetadataReaderFactoryPostProcessor`实例。
- ResourceUrlProvider：`handling mappings`处理

### started->ApplicationStartedEvent



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/168088a3a77dae6d~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

相关监听器参考上面的描述。



### running->ApplicationReadyEvent



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/1/168088b04deff509~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

相关监听器参考上面的描述。



### BackgroundPreinitializer&DelegatingApplicationListener

这两个贯穿了整个过程，这里拎出来单独解释下：

- BackgroundPreinitializer：对于一些耗时的任务使用一个后台线程尽早触发它们开始执行初始化，这是`SpringBoot`的缺省行为。这些初始化动作也可以叫做预初始化。可以通过设置系统属性`spring.backgroundpreinitializer.ignore`为`true`可以禁用该机制。该机制被禁用时，相应的初始化任务会发生在前台线程。
- DelegatingApplicationListener：监听应用事件，并将这些应用事件广播给环境属性`context.listener.classes`指定的那些监听器。

