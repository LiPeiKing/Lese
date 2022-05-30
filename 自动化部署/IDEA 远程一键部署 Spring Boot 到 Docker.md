# IDEA 远程一键部署 Spring Boot 到 Docker

## 一、开发前准备

### 1. Docker的安装可以参考：[docker安装](https://docs.docker.com/install/)

### 2. 配置docker远程连接端口

```shell
  vi /usr/lib/systemd/system/docker.service
```

找到 **ExecStart**，在最后面添加 **-H tcp://0.0.0.0:2375**，如下图所示

![image-20220513184254765](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184254765.png) 


### **3. 重启docker**

```shell
 systemctl daemon-reload
 systemctl start docker
```

### **4. 开放端口**

```shell
firewall-cmd --zone=public --add-port=2375/tcp --permanent
```

### **5. Idea安装插件,重启**

![image-20220513184345513](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184345513.png)  

### **6. 连接远程docker**

### **(1) 编辑配置**

![image-20220513184425448](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184425448.png)   



### **(2) 填远程docker地址**

<img src="C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184455215.png" alt="image-20220513184455215" style="zoom: 50%;" />   

### **(3) 连接成功，会列出远程docker**[**容器**](https://cloud.tencent.com/product/tke?from=10680)**和镜像**

![image-20220513184627716](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184627716.png)   



# **二、新建项目**

### **1. 创建springboot项目**

项目结构图

<img src="C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184648295.png" alt="image-20220513184648295" style="zoom: 80%;" />  

### **(1) 配置pom文件**

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.king.docker</groupId>
    <artifactId>com.king.docker</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>com.king.docker</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>

            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <dockerDirectory>src/main/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <includeSystemScope>true</includeSystemScope>
                    <mainClass>com.king.docker.Application</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>1.8</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <configuration>
                            <target>
                                <copy todir="src/main/docker" overwrite="true">
                                    <fileset dir="target/">
                                        <include name="${project.artifactId}-${project.version}.jar"/>
                                    </fileset>
                                </copy>
                            </target>
                        </configuration>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

### **(2) 在src/main目录下创建docker目录，并创建Dockerfile文件**

```shell
FROM openjdk:8-jdk-alpine
ADD *.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

### **(3) 在resource目录下创建application.yml文件**

```yaml
server:
    port: 8090
spring:
    application:
        name: com.king.docker

```

### **(4) 创建DockerApplication文件**

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

###  **(5) 创建DockerController文件**

```java
@RestController
public class DockerController {

    @GetMapping("/docker")
    public String docker() {

        System.out.println("ceshi111");
        return "docker";
    }
}
```

### **(6) 增加配置**

<img src="C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184844799.png" alt="image-20220513184844799" style="zoom:80%;" />   

![image-20220513184755927](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513184755927.png)   

命令解释 

**Image tag :**指定镜像名称和**tag**，镜像名称为 **docker-demo**，**tag**为**1.1** 

**Bind ports :**绑定[宿主机](https://cloud.tencent.com/product/cdh?from=10680)端口到容器内部端口。格式为[宿主机端口]:[容器内部端口] 

**Bind mounts :**将宿主机目录挂到到容器内部目录中。格式为[宿主机目录]:[容器内部目录]。这个springboot项目会将日志打印在容器 **/home/developer/app/logs/**目录下，将宿主机目录挂载到容器内部目录后，那么日志就会持久化容器外部的宿主机目录中。

### **(7) Maven打包**

<img src="C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513185005739.png" alt="image-20220513185005739" style="zoom:80%;" />  

### **(8) 运行**

![image-20220513185048142](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513185048142.png)   



![image-20220513185151311](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513185151311.png)  

这里我们可以看到镜像名称为:docker-boot，镜像已推动至远端

### **(9) 运行成功**

![image-20220513185300304](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513185300304.png)



### **(10) 浏览器访问**

![image-20220513185322614](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20220513185322614.png)  









