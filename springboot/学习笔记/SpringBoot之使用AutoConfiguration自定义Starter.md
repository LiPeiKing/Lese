# SpringBoot之使用AutoConfiguration自定义Starter

## SpringBoot内置条件注解

有关`@ConditionalOnXxx`相关的注解这里要系统的说下，因为这个是我们配置的关键，根据名称我们可以理解为`具有Xxx条件`，当然它实际的意义也是如此，条件注解是一个系列，下面我们详细做出解释

`@ConditionalOnBean`：当`SpringIoc`容器内存在指定`Bean`的条件
`@ConditionalOnClass`：当`SpringIoc`容器内存在指定`Class`的条件
`@ConditionalOnExpression`：基于SpEL表达式作为判断条件
`@ConditionalOnJava`：基于`JVM`版本作为判断条件
`@ConditionalOnJndi`：在JNDI存在时查找指定的位置
`@ConditionalOnMissingBean`：当`SpringIoc`容器内不存在指定`Bean`的条件
`@ConditionalOnMissingClass`：当`SpringIoc`容器内不存在指定`Class`的条件
`@ConditionalOnNotWebApplication`：当前项目不是Web项目的条件
`@ConditionalOnProperty`：指定的属性是否有指定的值
`@ConditionalOnResource`：类路径是否有指定的值
`@ConditionalOnSingleCandidate`：当指定`Bean`在`SpringIoc`容器内只有一个，或者虽然有多个但是指定首选的`Bean`
`@ConditionalOnWebApplication`：当前项目是Web项目的条件

以上注解都是元注解`@Conditional`演变而来的，根据不用的条件对应创建以上的具体条件注解。

到目前为止我们还没有完成自动化配置`starter`，我们需要了解`SpringBoot`运作原理后才可以完成后续编码。

## Starter自动化运作原理

在注解`@SpringBootApplication`上存在一个开启自动化配置的注解`@EnableAutoConfiguration`来完成自动化配置，注解源码如下所示：

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Import;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

在`@EnableAutoConfiguration`注解内使用到了`@import`注解来完成导入配置的功能，而`EnableAutoConfigurationImportSelector`内部则是使用了`SpringFactoriesLoader.loadFactoryNames`方法进行扫描具有`META-INF/spring.factories`文件的jar包。我们可以先来看下`spring-boot-autoconfigure`包内的`spring.factories`文件内容，如下所示：

```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnClassCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
.....省略
```

可以看到配置的结构形式是`Key`=>`Value`形式，多个`Value`时使用`,`隔开，那我们在自定义`starter`内也可以使用这种形式来完成，我们的目的是为了完成自动化配置，所以我们这里`Key`则是需要使用`org.springframework.boot.autoconfigure.EnableAutoConfiguration`

## 构建项目

https://segmentfault.com/a/1190000011433487









































































