# 函数式接口@FunctionalInterface

## 什么是函数式接口

所谓的函数式接口，当然首先是一个接口，然后就是在这个接口里面只能有一个抽象方法。

这种类型的接口也称为SAM接口，即Single Abstract Method interfaces

## 特点

- 接口有且仅有一个抽象方法
- 允许定义静态方法
- 允许定义默认方法
- 允许java.lang.Object中的public方法
- 该注解不是必须的，如果一个接口符合"函数式接口"定义，那么加不加该注解都没有影响。加上该注解能够更好地让编译器进行检查。如果编写的不是函数式接口，但是加上了@FunctionInterface，那么编译器会报错

## 例子



```java
// 正确的函数式接口
@FunctionalInterface
public interface TestInterface {
 
    // 抽象方法
    public void sub();
 
    // java.lang.Object中的public方法
    public boolean equals(Object var1);
 
    // 默认方法
    public default void defaultMethod(){
    
    }
 
    // 静态方法
    public static void staticMethod(){
 
    }
}

// 错误的函数式接口(有多个抽象方法)
@FunctionalInterface
public interface TestInterface2 {

    void add();
    
    void sub();
}
```

## 用法

1. Runnable的用法
2. 公司组内的用法介绍



```tsx
/**
 * 真正的业务处理
 */
@FunctionalInterface
public static interface RequestExecutor {
    Object doExecute() throws BusinessException;
}

/**
 * 请求模板
 */
public void templateRequest(RequestExecutor executor) {
    // 获取response对象
    HttpServletResponse response =
        ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();
    
    try {
        
        Object result = executor.doExecute();
        writeAjaxJSONResponse(ResultMessageBuilder.build(true, "success!", result), response);
        
    } catch (Exception e) {
    
        logger.error("请求失败", e);
        writeAjaxJSONResponse(ResultMessageBuilder.build(false, 500, "系统异常"), response);
        
    }
}

/**
 * controller部分代码
 */
@PostMapping(path = "/pageQuery")
public void pageQuery(@RequestBody Query query) {
    templateRequest(() -> service.pageQuery(query));
}

/**
 * service部分代码
 */
public PagedResult<VO> pageQuery(Query query);
```

## JDK中的函数式接口举例

java.lang.Runnable,

java.awt.event.ActionListener,

java.util.Comparator,

java.util.concurrent.Callable

java.util.function包下的接口，如Consumer、Predicate、Supplier等