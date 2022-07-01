# Jenkins+Docker+Gitee+SpringBoot自动化部署

## 1， 搭建Jenkins平台

> 见前序文档

## 2， Jenkins平台配置

> 见前序文档

### 安装GItee插件





## 3，Vue项目

略



并在根目录下新建docker文件夹，在docker文件夹下新建Dockefile文件，内容如下。

```dockerfile
# 基础镜像 不指定版本则默认最新
FROM nginx
# author
MAINTAINER Elwin

# 挂载目录
VOLUME /usr/share/nginx/html

# 指定路径 该路径我是自己提前创建好的
WORKDIR /usr/share/nginx/html
# 复制conf文件到路径 ./conf/nginx.conf和上面的mysql一个道理 nginx.conf下面会贴出来
COPY /opt/nginx.conf /etc/nginx/nginx.conf
# 复制html文件到路径 dist中存放前端项目的打包文件，ngnix默认路径存在权限问题无法复制
COPY ./dist/ /usr/share/nginx/html
```

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

![image-20220606184837886](C:/Users/elwin/AppData/Roaming/Typora/typora-user-images/image-20220606184837886.png)

shell如下：

```shell
#!/bin/bash -il
cnpm install
cnpm run build:reduction

docker rm -f jenkins_vue
sleep 1
docker rmi -f jenkins_vue:1.0
sleep 1
docker build -t jenkins_vue:1.0 -f ./Dockerfile .
sleep 1
docker run -d -p 9092:9092 --name jenkins_vue -v /opt/ngnix/html:/usr/share/nginx/html jenkins_vue:1.0
```



