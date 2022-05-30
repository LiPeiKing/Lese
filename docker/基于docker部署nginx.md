# 基于docker部署nginx

#### 拉取镜像

```undefined
docker pull nginx
```

#### 创建挂载目录

```shell
mkdir html
mkdir nginx
```

##### 在/nginx下面创建nginx.conf

```shell
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
        server {
            listen       80;
            server_name  localhost;

            #charset koi8-r;

            #access_log  logs/host.access.log  main;

            location / {
                 root   /etc/nginx/html/;
                 index  index.html index.htm;
            }
        }
}
```

##### 把静态页面放入html目录下



##### 启动

```shell
docker run --name hotmap -d -p 8099:80  -v $PWD/nginx/nginx.conf:/etc/nginx.conf -v $PWD/html:/etc/nginx/html -v $PWD/logs:/etc/nginx/log/ nginx

# -d 后台运行
# -i 交互方式运行
# --name 自定义容器名称
# -p 端口号映射 90 自定义为外部访问端口:80 为nginx容器对外暴露的端口
# -v 目录挂载冒号前为外部目录，冒号后为容器内目录；相当于外部目录中的内容会映射同步到容器内
```

+ 注意：nginx配置中root指向目录需和容器内路径一致



#### 进入到容器命令

```shell
docker exec -it container-id/container-name bash
# container-id     容器id
# container-name   自定义容器名称
```



##### 进入到容器的指定位置查看配置

```shell
cd /etc/nginx/conf.d/
cat default.conf 
```

/etc/nginx/conf.d/default.conf为默认读取配置，该配置若挂载则不许修改

