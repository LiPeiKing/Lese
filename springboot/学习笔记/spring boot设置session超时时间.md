# spring boot设置session超时时间

>一下顺序即是springboot的优先级   

### 一、启动类中配置

```java
package com.mycenter;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
import org.springframework.boot.web.servlet.ServletComponentScan;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
@MapperScan(value = "com.mycenter.mapper")
@ServletComponentScan
public class MycenterApplication {

    public static void main(String[] args) {
        SpringApplication.run(MycenterApplication.class, args);
    }


    @Bean
    public EmbeddedServletContainerCustomizer containerCustomizer(){
        return container -> {
            container.setSessionTimeout(7200);/*单位为S*/
        };
    }
}
```

### 二、配置文件

#### 1、数字

```yaml
server:
  port: 8080
  servlet:
    session:
      timeout: 7200s
```

#### 2、Duration

```yaml
server:
  port: 80
  servlet:
    session:
      timeout: PT20M
```

Duration转换字符串方式，默认为正，负以-开头，紧接着P，（字母不区分大小写）D ：天 T：天和小时之间的分隔符 H ：小时  M：分钟  S：秒 每个单位都必须是数字，且时分秒顺序不能乱。例如PT20M，就是设置为20分钟，可以按照自己的需求。

可以通过`int sessionTimeout = session.getServletContext().getSessionTimeout();`打印出自己的session时间

举例：

```java
P30D		30天
PT24H		24小时
PT20.345S	20.345秒
P2DT3H4M	2天3小时4分钟
    