# Spring Boot Validator

### 前言

JSR(Java Specification Requests） 是一套 JavaBean 参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在我们 JavaBean 的属性上面，这样就可以在需要校验的时候进行校验了，非常方便！

校验的时候我们实际用的是 Hibernate Validator 框架。Hibernate Validator 是 Hibernate 团队最初的数据校验框架，Hibernate Validator 4.x 是 Bean Validation 1.0（JSR 303）的参考实现，Hibernate Validator 5.x 是 Bean Validation 1.1（JSR 349）的参考实现，目前最新版的 Hibernate Validator 6.x 是 Bean Validation 2.0（JSR 380）的参考实现。

SpringBoot 项目的 spring-boot-starter-web 依赖中已经有 hibernate-validator 包，不需要引用相关依赖。

使用 Spring Boot 程序的话只需要spring-boot-starter-web 就够了，它的子依赖包含了我们所需要的东西。非 SpringBoot 项目需要自行引入相关依赖包

但有一点需要注意，在更新版本的SpringBoot中，默认移除了Bean Validtion相关依赖。具体的对应关系可以参照如下表格：
![img](https://img2020.cnblogs.com/blog/1822926/202102/1822926-20210207093754623-2017740170.png)

maven地址

    <!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-validator -->
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>6.0.16.Final</version>
    </dependency>

### 字段注解

```java
@NotEmpty 被注释的字符串的不能为 null 也不能为空
@NotBlank 被注释的字符串非 null，并且必须包含一个非空白字符
@Null 被注释的元素（数字）必须为 null
@NotNull 被注释的元素（数字）必须不为 null
@AssertTrue 被注释的元素必须为 true
@AssertFalse 被注释的元素必须为 false
@Pattern(regex=,flag=)被注释的元素必须符合指定的正则表达式
@Email 被注释的元素必须是 Email 格式。
@Min(value)被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@Max(value)被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@DecimalMin(value)被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@DecimalMax(value) 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@Size(max=, min=) 被注释的元素（集合）的大小必须在指定的范围内
@Digits (integer, fraction)被注释的元素必须是一个数字，其值必须在可接受的范围内
@Past被注释的元素必须是一个过去的日期
@Future 被注释的元素必须是一个将来的日期
@Length(min=,max=) 被注释的字符串的大小必须在指定的范围内
@Range(min=,max=,message=) 被注释的元素必须在合适的范围内
```

### 参数校验

**一定一定不要忘记在类上加上 Validated 注解了，这个参数可以告诉 Spring 去校验方法参数。否则验证注解不生效**
Path Variables 和 Request Parameters

```java
@RestController
@RequestMapping("/api")
@Validated
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
        return ResponseEntity.ok().body(id);
    }

```

### 实体校验 @Valid

    @ApiOperation(value = "测试")
    @PostMapping()
    public CommonResult test(@Valid @RequestBody Test test) {}

### 嵌套校验

对象内部包含另一个对象作为属性，属性上加 @Valid，可以验证作为属性的对象内部的验证。

```java
@Data
public class ValidDemoQuery {

    private String name;

    @Valid
    private RoleDemoQuery role;

    @Min(value = 1, message = "年龄必须大于1岁！")
    private String age;

    @Data
    public class RoleDemoQuery {

        @Pattern(regexp = "admin|member", message = "角色必须为admin或member")
        private String role;

    }

}
```





### 分组校验

两个接口的DTO相同，校验项不同，加入分组groups

#### 方法一

```java
import javax.validation.groups.Default;

public interface AddGroupextends Default {
}

import javax.validation.groups.Default;

public interface UpdateGroupextends Default{
}

```

```java
@ApiModel(value="测试")
@Data
public class TestDTO {

    @NotNull(message = "id不为空")
    @Max(value = 1,message ="最大值不超过1" )
    private Long id;

    @NotBlank(message = "name不为空！",groups = {AddGroup.class, UpdateGroup.class})
    private String name;

    @NotBlank(message = "task 不为空！",groups = {UpdateGroup.class})
    private String task;

    @Email(message = "邮箱格式错误!")//只校验格式，不校验null
    @NotBlank(message = "邮箱 不为空！")
    private String email;
}

```

**再在需要校验的地方@Validated声明校验组**此处失败会触发ConstraintViolationException 异常。

```java
 /**
     * 走参数校验注解的 groups 组合校验
     *
     * @param userDTO
     * @return
     */
    @PostMapping("/update/groups")
    public RspDTO update(@RequestBody @Validated(UpdateGroup.class) TestDTO test) {
      
        return RspDTO.success();
 }

```

注意:在声明分组的时候尽量加上 extend javax.validation.groups.Default否则,在你声明@Validated(Update.class)的时候,就会出现你在默认没添加groups = {}的时候的校验组@Email(message = "邮箱格式不对"),会不去校验,因为默认的校验组是 groups = {Default.class}

#### 方法二(推荐)

```java
@ApiOperation(value = "测试")

    @PostMapping()

    public CommonResult test(@Valid @RequestBody Test test) {

        //校验未加group的和加了group = UpdateGroup
        //获取java默认的校验器。在spring项目中可以设为单例bean然后注入，本demo直接获取
        Validator validator = Validation.buildDefaultValidatorFactory().getValidator();
        //获得校验不通过的结果，保存到set
        Set<ConstraintViolation<Object>> set = validator.validate(test, UpdateGroup.class);
        if (set != null && set.size() > 0){
            throw new VisualException(ResultCode.FAILED.getCode(),set.iterator().next().getMessage());
        }
        return CommonResult.success();
 }  

```

### 自定义注解

原有注解不能满足校验需求时，需要引入自定义注解

```java
@Documented
@Constraint(validatedBy = PhoneNumberValidator.class)
@Target({FIELD, PARAMETER})
@Retention(RUNTIME)
public@interface PhoneNumber {
    String message() default "Invalid phone number";
    Class[] groups() default {};
    Class[] payload() default {};
}

```

```java
publicclass PhoneNumberValidator implements ConstraintValidator<PhoneNumber,String> {

    @Override
    public boolean isValid(String phoneField, ConstraintValidatorContext context) {
        if (phoneField == null) {
            // can be null
            returntrue;
        }
        return phoneField.matches("^1(3[0-9]|4[57]|5[0-35-9]|8[0-9]|70)\\d{8}$") && phoneField.length() > 8 && phoneField.length() < 14;
    }
}

```

搞定，我们现在就可以使用这个注解了。

```java
@PhoneNumber(message = "phoneNumber 格式不正确")
@NotNull(message = "phoneNumber 不能为空")
private String phoneNumber;

```

### 校验模式

**普通模式（默认）**：在普通模式下，会校验完所有的属性，然后返回所有验证失败信息。

**快速失败模式**：只要有一个验证失败，则返回。切换为快速失败模式可参考下面代码：

```java
@Configuration
public class ValidatorConfiguration {
    @Bean
    public Validator validator(){
        ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
                .configure()
                .addProperty( "hibernate.validator.fail_fast", "true" )
                .buildValidatorFactory();
        Validator validator = validatorFactory.getValidator();
        return validator;
    }
}
```



### 统一异常处理

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // 捕捉参数校验异常
    @ExceptionHandler(BindException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public R<String> bindException(HttpServletRequest request, BindException ex) {
        return new R<>(getStatus(request).value(), ex.getBindingResult().getAllErrors().get(0).getDefaultMessage());
    }

    // 捕捉参数校验异常
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public R<String> constraintViolationException(HttpServletRequest request, ConstraintViolationException e) {
        return new R<>(getStatus(request).value(),
                e.getConstraintViolations().stream().findAny().isPresent()
                        ? e.getConstraintViolations().stream().findAny().get().getMessageTemplate() : "参数有误！");
    }

    // 捕捉其他所有异常
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public R<String> globalException(HttpServletRequest request, Throwable ex) {
        return new R<>(getStatus(request).value(), ex.getMessage());
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        return HttpStatus.valueOf(statusCode);
    }

}
```



### 总结

```java
/**
     * @Valid 只校验未分组的,带group的不校验
     * @Validated 只校验未分组的,带group的不校验
     * @Validated(AddGroup.class) 只校验AddGroup的
     * @Valid@Validated(AddGroup.class) 只校验未分组的,带group的不校验
     * 只有@Max (value = 1,message ="最大值不超过1" )  不校验
     * 方法上 @Validated  @Max (value = 1,message ="最大值不超过1" )  不校验
     * @Validated 类上 @Max (value = 1,message ="最大值不超过1" ) 成功校验
     */

```

![img](https://img2020.cnblogs.com/blog/1822926/202102/1822926-20210207094540403-499749728.png)