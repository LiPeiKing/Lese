# SpringBoot之统一异常处理

### 前言

之前用springboot的时候，只知道捕获异常使用try{}catch，一个接口一个try{}catch，这也是大多数开发人员异常处理的常用方式，虽然屡试不爽，但会造成一个问题，就是一个Controller下面，满屏幕的try{}catch，看着一点都不优雅，一点都不符合小明的气质，憋了这么久，小明今天终于决定对所有异常实施统一处理的方案。

### 技术支持

通过 Spring 的 AOP 特性就可以很方便的实现异常的统一处理：使用@ControllerAdvice、@RestControllerAdvice配合@ExceptionHandler(value = {Exception.class}) 捕获运行时异常。

`@ControllerAdvice`和`@RestControllerAdvice`都可以指向控制器的一个子集：

```
// 指向所有带有注解@RestController的控制器
@ControllerAdvice(annotations = RestController.class)
public class AnnotationAdvice {}

// 指向所有指定包中的控制器
@ControllerAdvice("org.example.controllers")
public class BasePackageAdvice {}

// 指向所有带有指定签名的控制器
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class AssignableTypesAdvice {}
```

### 通用异常处理

**举个例子:**

假如我们需要针对NullException（空指针异常，是Java程序员最痛恨的异常，没有之一）进行全局处理（如下所示）。

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
   * 处理空指针的异常
   * @param req
   * @param e
   * @return
   */
  @ExceptionHandler(value =NullPointerException.class)
  public BaseResponseFacade exceptionHandler(HttpServletRequest req, NullPointerException e){
    log.error("发生空指针异常！原因是:",e);
    return ResponseUtil.error(ResponseCode.ERROR);
  }
}
```



### 自定义异常处理

自定义异常处理

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.experimental.Accessors;

/**
 * @Description 自定义异常
 * @Date 2019-08-05 15:49
 * @Created by 程序员小明
 */
@Data
@AllArgsConstructor
@Accessors(chain = true)
public class BizException extends RuntimeException {
  /**
   * 错误码
   */
  protected Integer errorCode;
  /**
   * 错误信息
   */
  protected String errorMsg;
}
```

定义过之后，我们就可以和之前处理NullException方式一样处理我们自定义的异常。

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

  /**
   * 处理自定义的业务异常
   * @param req
   * @param e
   * @return
   */
  @ExceptionHandler(value = BizException.class)
  public BaseResponseFacade bizExceptionHandler(HttpServletRequest req, BizException e){
    log.error("发生业务异常！原因是：{}",e.getErrorMsg());
    return ResponseUtil.error(e.getErrorCode(),e.getErrorMsg());
  }

  /**
   * 处理空指针的异常
   * @param req
   * @param e
   * @return
   */
  @ExceptionHandler(value =NullPointerException.class)
  public BaseResponseFacade exceptionHandler(HttpServletRequest req, NullPointerException e){
    log.error("发生空指针异常！原因是:",e);
    return ResponseUtil.error(ResponseCode.ERROR);
  }


  /**
   * 处理其他异常
   * @param req
   * @param e
   * @return
   */
  @ExceptionHandler(value =Exception.class)
  public BaseResponseFacade exceptionHandler(HttpServletRequest req, Exception e){
    log.error("未知异常！原因是:",e);
    return ResponseUtil.error(ResponseCode.INTERNAL_SERVER_ERROR);
  }
}
```

















