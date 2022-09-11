# docker安装nginx并部署一个html静态网站

## 1.搜索安装的 nginx 镜像

```javascript
docker search nginx
```

## 2.在docker hub 中选择合适的版本后进行 镜像拉取

```javascript
 docker pull nginx
```

## 3.拉取完成后运行 nginx 容器

 使用 xftp 上传静态页面到服务器的/usr/html 目录下

```javascript
docker run -di --name=nginx -p 90:80 -v /usr/html:/usr/share/nginx/html nginx
# -d 后台运行
# -i 交互方式运行
# --name 自定义容器名称
# -p 端口号映射 90 自定义为外部访问端口：80 为nginx容器对外暴露的端口
# -v 目录挂载  冒号前为 外部目录，冒号后为 容器内目录；相当于外部目录中的内容会映射同步到容器内
```

## 4.访问运行好的容器

```javascript
 ip:90          ip为当前服务器ip地址
```

## 5.进入到容器命令

```javascript
docker exec -it container-id/container-name bash
# container-id     容器id
# container-name   自定义容器名称
```

## 6.进入到容器的指定位置查看配置

```javascript
cd /etc/nginx/conf.d/
```

可以看到默认的配置文件：

```javascript
cat default.conf 
server {
    # 默认监听 80 端口
    listen       80;
    # localhost 为外部访问该地址的域名   域名解析指向---> NGINX 配置文件所在服务器
    server_name  localhost;
    
    # 这里为本地代理，当外部访问 server_name 域名的时候 就会转到以下代理地址
    #1ocation / {
    #     proxy_pass http://192.168.0.243：8778;
	#}

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    # nginx 的默认访问文件夹为 root  /usr/share/nginx/html
    # nginx 的默认访问页面为  index  index.html  index.htm
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```