# Springboot之Bean初始化的顺序

## Bean定义

```java
package com.example.demo.beandemo;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class BeanInitOrder implements InitializingBean, DisposableBean, BeanNameAware, BeanFactoryAware, ApplicationContextAware {

    private String name;

    public BeanInitOrder() {
        System.out.println("执行构造方法");
    }

    public void initMethod() {
        System.out.println("执行bean定义的initMethod方法");
    }

    public void destroyMethod() {
        System.out.println("执行bean定义的destroyMethod方法");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("执行PostConstruct方法");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("执行PreDestroy方法");
    }

    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("注入BeanFactory");
    }

    public void setBeanName(String s) {
        System.out.println("注入BeanName");
    }

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("注入ApplicationContext");
    }


    public void destroy() throws Exception {
        System.out.println("执行量DisposableBean的destroy方法");
    }

    public void afterPropertiesSet() throws Exception {
        System.out.println("执行InitializingBean的afterPropertiesSet方法");
    }

    public void setName(String name){
        this.name=name;
    }

    public String getName() {
        return name;
    }

}
```

## 后置处理器BeanPostProcessor

```java
package com.example.demo.beandemo;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;


public class InitBeanHandleBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("getBeanInitOrder")) {
            System.out.println("执行BeanPostProcessor的postProcessBeforeInitialization方法");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("getBeanInitOrder")) {
            System.out.println("执行BeanPostProcessor的postProcessAfterInitialization方法");
        }
        return bean;
    }
}
```

## 属性处理器BeanFactoryPostProcessor 

```java
package com.example.demo.beandemo;

import org.springframework.beans.BeansException;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;

public class InitBeanHandleBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {BeanDefinition getBeanInitOrder = beanFactory.getBeanDefinition("getBeanInitOrder");
        MutablePropertyValues propertyValues = getBeanInitOrder.getPropertyValues();
        propertyValues.addPropertyValue("name", "mateEgg");
        System.out.println("执行BeanFactoryPostProcessor的postProcessBeanFactory方法");
    }
}
```

## 配置类

```css
package com.example.demo.config;

import com.example.demo.beandemo.BeanInitOrder;
import com.example.demo.beandemo.InitBeanHandleBeanPostProcessor;
import com.example.demo.beandemo.InitBeanHandleBeanFactoryPostProcessor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;

@Configuration
public class BeanInitOrderConfig {


    @Bean(initMethod = "initMethod" ,destroyMethod = "destroyMethod")
    public BeanInitOrder getBeanInitOrder(){
        return new BeanInitOrder();
    }

    @Bean
    public InitBeanHandleBeanPostProcessor getInitBeanHandle(){
        return new InitBeanHandleBeanPostProcessor();
    }

    @Bean
    public static InitBeanHandleBeanFactoryPostProcessor getInitBeanHandleBeanFactoryPostProcessor(){
        return new InitBeanHandleBeanFactoryPostProcessor();
    }

}
```

## 启动类

```swift
package com.example.demo;

import com.example.demo.bean.Car;
import com.example.demo.beandemo.BeanInitOrder;
import com.sun.corba.se.spi.orb.ParserImplBase;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.web.bind.annotation.InitBinder;

import javax.annotation.PostConstruct;

@SpringBootApplication
public class DemoApplication {

    @Autowired
    private BeanInitOrder beanInitOrder;

    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(DemoApplication.class, args);
    }

    @PostConstruct
    public void test(){
        System.out.println(beanInitOrder.getName());
    }
 }
```

## 输出结果



```bash
执行BeanFactoryPostProcessor的postProcessBeanFactory方法
2019-01-24 21:59:57.822  INFO 8118 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'beanInitOrderConfig' of type [com.example.demo.config.BeanInitOrderConfig$$EnhancerBySpringCGLIB$$3f223dec] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-01-24 21:59:58.137  INFO 8118 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8082 (http)
2019-01-24 21:59:58.159  INFO 8118 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-01-24 21:59:58.160  INFO 8118 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.14]
2019-01-24 21:59:58.167  INFO 8118 --- [           main] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/Users/mac/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.]
2019-01-24 21:59:58.246  INFO 8118 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-01-24 21:59:58.246  INFO 8118 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1493 ms
执行构造方法
注入BeanName
注入BeanFactory
注入ApplicationContext
执行BeanPostProcessor的postProcessBeforeInitialization方法
执行PostConstruct方法
执行InitializingBean的afterPropertiesSet方法
执行bean定义的initMethod方法
执行BeanPostProcessor的postProcessAfterInitialization方法
mateEgg
```



