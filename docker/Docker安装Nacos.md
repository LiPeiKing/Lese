# Docker安装Nacos

## 1、拉取镜像

在命令行窗口输入以下命令，我这里是指定了版本号的；不指定版本号默认下载最新的。

```shell
docker pull nacos/nacos-server
```



## 2、启动容器

```shell
docker  run \
--name nacos -d \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v /data/nacos/logs:/home/nacos/logs \
nacos/nacos-server
```



## 3、测试

在浏览器上输入访问地址：`http://ip:8848/nacos`测试是否安装成功

用户名：nacos

密码：nacos