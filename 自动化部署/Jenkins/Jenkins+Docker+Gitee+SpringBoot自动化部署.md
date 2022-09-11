# Jenkins+Docker+Gitee+SpringBoot自动化部署

当我们使用传统的开发方式开发后台系统时，每写完一个功能点就需要重新运行一下项目，然后进行测试，如果是项目比较小还可以，但是如果项目比较大的话，由于涉及的人员比较多，这种开发方式就比较麻烦。基于此，我们就需要使用Jenkins配合Gitee搭建一个自动化部署平台，并将代码托管到服务器上，这样减轻了本地的电脑压力，也解放了部署的流程。

## 1， 搭建Jenkins平台

> 见前序文档

## 2， Jenkins平台配置

> 见前序文档

### 安装GItee插件





## 3，创建SpringBoot应用

首先，我们创建一个简单的SpringBoot应用进行测试，控制器代码如下。

```java
@RestController
@Slf4j
public class IndexController {

    @GetMapping("/index")
    public String index() {
        log.info("index==================");
        return "index";
    }

    @GetMapping("/test")
    public String test() {
        log.info("index==================");
        return "test";
    }
}
```

然后在配置文件application.yml中添加：

```text
server:
  port: 8080
```

并在main下新建docker文件夹，在docker文件夹下新建Dockefile文件，内容如下。

```shell
# 指定是基于哪个基础镜像
FROM java:8

# 作者信息
MAINTAINER Elwin

# 挂载点声明
VOLUME /opt/logs

# 将本地的一个文件或目录，拷贝到容器的文件或目录里
ADD /target/jenkins-0.0.1-SNAPSHOT.jar /opt/jenkins-0.0.1-SNAPSHOT.jar

##shell脚本
#RUN bash -c 'touch /jenkins-0.0.1-SNAPSHOT.jar'

# 将容器的8000端口暴露，给外部访问。
EXPOSE 8000

#添加进入docker容器后的目录
WORKDIR /opt/
# 当容器运行起来时执行使用运行jar的指令
ENTRYPOINT ["java", "-jar", "/opt/jenkins-0.0.1-SNAPSHOT.jar"]
```

需要注意的是ADD指令的编写，当SpringBoot应用打包完成后，其jar包会被放在target目录下，/为项目根目录非`src`。   

![image-20220601203043643](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203043643.png)



所以需要指定该文件的位置，使用ADD指令将其放入待构建的容器中，接着在Gitee中新建一个仓库，并将代码推送到仓库中。

![image-20220601203120622](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203120622.png)

## 4，Gitee配置

#### 1) 全局配置Git

![image-20220601203536335](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203536335.png)

![image-20220601203553880](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203553880.png)

#### 2)局部配置Git

![image-20220601203625438](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203625438.png)



## 5，构建触发器

![image-20220601203757706](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203757706.png)

![image-20220601203812313](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203812313.png)



![image-20220601203834269](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601203834269.png)

## 6，构建docker镜像并运行

![image-20220601204108547](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601204108547.png)   



![image-20220601204210162](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220601204210162.png)  



shell如下：

```shell
#!/bin/bash -il
docker rm -rf jenkins_docker
sleep 1
docker rmi -rf jenkins_docker:1.0
sleep 1
mvn clean install -Dmaven.test.skip=true
sleep 1
docker build -t jenkins_docker:1.0 -f ./src/main/docker/Dockerfile .
sleep 1
# 挂在日志目录
docker run -d -p 8000:8080 --name jenkins_docker -v /opt/logs/:/opt/logs/ jenkins_docker:1.0
```

## 7，打包测试

最后点击保存，部署任务就创建完成了，我们来测试一下有没有问题。

点击立即构建，Jenkins会立马进行一次构建，查看控制台输出。

最后，我们打开默认的地址即可。



## 8，镜像导出与导入

```shell
[root@elwin-centos8] docker commit jenkins_docker demo/test:1.0
# 将指定镜像jenkins_docker保存成 tar归档文件demo/test:1.0

[root@elwin-centos8] docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
demo/test             1.0       0829ce8ee46f   8 seconds ago    663MB

[root@elwin-centos8] docker save -o demo.tar demo/test
# 保存为demo名称tar文件


#上传至生产
[root@elwin-centos8] docker load < demo.tar
[root@elwin-centos8] docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
demo/test             1.0       0829ce8ee46f   4 minutes ago    663MB


[root@elwin-centos8] docker run -d -p 8000:8080 --name demo -v /opt/logs/:/opt/logs/ 0829ce8ee46f
```

