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