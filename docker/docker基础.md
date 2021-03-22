## docker基础

> 首先，我们必须添加一个外部存储库以获得Docker CE。我们将使用官方的Docker CE CentOS存储

### 环境安装

#### 安装centos7

...

#### 查看内核版本

```shell
uname -r
```

#### docker安装步骤

##### 准备

1. 服务器可连接外网

2. 安装gcc

   ```shell
   yum install gcc
   yum install gcc-c++
   ```

3. 卸载旧版本

   ```shell
   yum remove docker \ docker-client \ docker-client-latest \ docker-common \ docker-latest \ docker-latest-logrotate \ docker-logrotate \ docker-selinux \ docker-engine-selinux \ docker-engine
   ```

##### 安装docekr

```
curl https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo

yum install https://download.docker.com/linux/Fedora/30/x86_64/stable/Packages/containerd.io-1.2.6-3.3.fc30.x86_64.rpm

yum install docker-ce --allowerasing

```

##### 启动docker

```
systemctl status  docker
```

##### 配置国内docker镜像源

```
sudo mkdir -p /etc/docker
sudo vi /etc/docker/daemon.json

Docker中国官方镜像加速
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}

网易163镜像加速
{
"registry-mirrors": ["http://hub-mirror.c.163.com"]
}

中科大镜像加速
{
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]     
}
```

### 常用命令

#### 容器信息

```shell
##查看docker容器版本
docker version
##查看docker容器信息
docker info
##查看docker容器帮助
docker --help
```

#### 镜像操作

1. 查看镜像

   ```shell
   ##列出本地images 
   docker images 
   ##含中间映像层 
   docker images -a
   ##只显示镜像ID
   docker images -q
   ##含中间映像层
   docker images -qa 
   ##显示镜像摘要信息(DIGEST列)
   docker images --digests
   ##显示镜像完整信息
   docker images --no-trunc
   ##显示指定镜像的历史创建；参数：-H 镜像大小和日期，默认为true；--no-trunc  显示完整的提交记录；-q  仅列出提交记录ID
   docker history -H redis
   ```

2. 镜像搜索

   ```shell
   ##搜索仓库MySQL镜像
   docker search mysql
   ## --filter=stars=600：只显示 starts>=600 的镜像
   docker search --filter=stars=600 mysql
   ## --no-trunc 显示镜像完整 DESCRIPTION 描述
   docker search --no-trunc mysql
   ## --automated ：只列出 AUTOMATED=OK 的镜像
   docker search  --automated mysql
   ```

   

3. 镜像下载

   ```shell
   ##下载Redis官方最新镜像，相当于：docker pull redis:latest
   docker pull redis
   ##下载仓库所有Redis镜像
   docker pull -a redis
   ##下载私人仓库镜像
   docker pull bitnami/redis
   ```

   

4. 镜像删除

   ```shell
   ##单个镜像删除，相当于：docker rmi redis:latest
   docker rmi redis
   ##强制删除(针对基于镜像有运行的容器进程)
   docker rmi -f redis
   ##多个镜像删除，不同镜像间以空格间隔
   docker rmi -f redis tomcat nginx
   ##删除本地全部镜像
   docker rmi -f $(docker images -q)
   ```

   

5. 镜像构建

   ```sh
   ##编写dockerfile
   cd /docker/dockerfile
   vim mycentos
   ##构建docker镜像
   docker build -f /docker/dockerfile/mycentos -t mycentos:1.1
   ```

   

#### 容器操作

1. 容器启动

   ```java
   ##新建并启动容器，参数：-i  以交互模式运行容器；-t  为容器重新分配一个伪输入终端；--name  为容器指定一个名称
   docker run -i -t --name mycentos
   ##后台启动容器，参数：-d  已守护方式启动容器
   docker run -d mycentos
   ```

   注意：此时使用"docker ps -a"会发现容器已经退出。这是docker的机制：要使Docker容器后台运行，就必须有一个前台进程。解决方案：将你要运行的程序以前台进程的形式运行。

   ```shell
   ##启动一个或多个已经被停止的容器
   docker start redis
   ##重启容器
   docker restart redis
   ```

   

2. 容器进程

   ```shell
   ##top支持 ps 命令参数，格式：docker top [OPTIONS] CONTAINER [ps OPTIONS]
   ##列出redis容器中运行进程
   docker top redis
   ##查看所有运行容器的进程信息
   for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done
   ```

   

3. 容器日志

   ```shell
   ##查看redis容器日志，默认参数
   docker logs rabbitmq
   ##查看redis容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志；
   docker logs -f -t --tail=20 redis
   ##查看容器redis从2019年05月21日后的最新10条日志。
   docker logs --since="2019-05-21" --tail=10 redis
   ```

   

4. 容器的进入与退出

   ```shell
   ##使用run方式在创建时进入
   docker run -it centos /bin/bash
   ##关闭容器并退出
   exit
   ##仅退出容器，不关闭
   快捷键：Ctrl + P + Q
   ##直接进入centos 容器启动命令的终端，不会启动新进程，多个attach连接共享容器屏幕，参数：--sig-proxy=false  确保CTRL-D或CTRL-C不会关闭容器
   docker attach --sig-proxy=false centos 
   ##在 centos 容器中打开新的交互模式终端，可以启动新进程，参数：-i  即使没有附加也保持STDIN 打开；-t  分配一个伪终端
   docker exec -i -t  centos /bin/bash
   ##以交互模式在容器中执行命令，结果返回到当前终端屏幕
   docker exec -i -t centos ls -l /tmp
   ##以分离模式在容器中执行命令，程序后台运行，结果不会反馈到当前终端
   docker exec -d centos  touch cache.txt 
   ```

   

5. 查看容器

   ```shell
   ##查看正在运行的容器
   docker ps
   ##查看正在运行的容器的ID
   docker ps -q
   ##查看正在运行+历史运行过的容器
   docker ps -a
   ##显示运行容器总文件大小
   docker ps -s
   ##显示最近创建容器
   docker ps -l
   ##显示最近创建的3个容器
   docker ps -n 3
   ##不截断输出
   docker ps --no-trunc
   ##获取镜像redis的元信息
   docker inspect redis
   ##获取正在运行的容器redis的 IP
   docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis
   ```

   

6. 容器的停止与删除

   ```shell
   ##停止一个运行中的容器
   docker stop redis
   ##杀掉一个运行中的容器
   docker kill redis
   ##删除一个已停止的容器
   docker rm redis
   ##删除一个运行中的容器
   docker rm -f redis
   ##删除多个容器
   docker rm -f $(docker ps -a -q)
   docker ps -a -q | xargs docker rm
   ## -l 移除容器间的网络连接，连接名为 db
   docker rm -l db 
   ## -v 删除容器，并删除容器挂载的数据卷
   docker rm -v redis
   ```

   

7. 生成镜像

   ```shell
   ##基于当前redis容器创建一个新的镜像；参数：-a 提交的镜像作者；-c 使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
   docker commit -a="DeepInThought" -m="my redis" [redis容器ID]  myredis:v1.1
   ```

   

8. 容器与主机间的数据拷贝

   ```shell
   ##将rabbitmq容器中的文件copy至本地路径
   docker cp rabbitmq:/[container_path] [local_path]
   ##将主机文件copy至rabbitmq容器
   docker cp [local_path] rabbitmq:/[container_path]/
   ##将主机文件copy至rabbitmq容器，目录重命名为[container_path]（注意与非重命名copy的区别）
   docker cp [local_path] rabbitmq:/[container_path]
   ```

   

### docker镜像构建

> Docker镜像构建分为两种，一种是手动构建，另一种是Dockerfile（自动构建）



案例：基于centos镜像进行构建，制作包含vim命令的centos镜像

#### 手动构建docker镜像

1. 查看已有镜像

   ```shell
   docker images
   ```

   ![image-20210228145648810](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210228145648810.png)

    

2. 拉取centos镜像

   ```shell
   docker pull centos
   ```

   ![image-20210228145728419](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210228145728419.png)

3. 交互式运行centos镜像

   ```shell
   docker run -it centos
   ```

   ![image-20210228145814871](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210228145814871.png)

4. centos容器中安装vim工具

   ```shell
   yum install vim
   ```

   ![image-20210228145855544](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210228145855544.png)

5. 退出centos交互式程序

   ```shell
   exit
   ```

   

6. 根据自定义的centos容器生成镜像

   ```shell
   docker commit a68c0 chanmufeng/centos-vim
   ```

   ![image-20210228145947166](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210228145947166.png)

#### Dockerfile创建镜像

> 使用Dockerfile是更推荐的方式，这样可以让使用者更清晰地看到这个镜像的制作细节

1. 创建对应目录

   ```shell
   mkdir centos-vim
   ```

   

2. 编写Dockerfile文件

   ```shell
   FROM centos:7
   RUN yum install -y vim
   ```

   

3. docker build

   ```shell
   docker build -t lipeiking/centos-vim2 .
   ```

   ![image-20210228150704909](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210228150704909.png)

##### 指令解析

- FROM 和 RUN 指令的作用

  **FROM**：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

  **RUN**：用于执行后面跟着的命令行命令。有以下俩种格式：

  shell 格式：

  ```
  RUN <命令行命令>
  # <命令行命令> 等同于，在终端操作的 shell 命令。
  ```

  **注意**：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：

  ```shell
  FROM centos
  RUN yum install wget
  RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
  RUN tar -xvf redis.tar.gz
  以上执行会创建 3 层镜像。可简化为以下格式：
  FROM centos
  RUN yum install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz
  ```

  如上，以 **&&** 符号连接命令，这样执行后，只会创建 1 层镜像。

- COPY
- ADD
- CMD
- ENV
- ARG
- VOLUME
- EXPOSE



### 安装mariadb

#### 拉取镜像

```
docker pull mariadb
```

#### 查看镜像

```
docker images
```

#### 创建一个目录作为和容器的映射目录

```
mkdir -p /data/mariadb/data
```

#### 启动镜像

```
docker run --name mariadb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /data/mariadb/data:/var/lib/mysql -d mariadb

	-name启动容器设置容器名称为mariadb

　　-p设置容器的3306端口映射到主机3306端口

　　-e MYSQL_ROOT_PASSWORD设置环境变量数据库root用户密码为输入数据库root用户的密码

　　-v设置容器目录/var/lib/mysql映射到本地目录/data/mariadb/data

　　-d后台运行容器mariadb并返回容器id

```

#### 查看容器是否运行

```
docker ps -a
```

#### 修改容器为自启动

```
docker container update --restart=always 容器id
```

#### 进入容器

```
docker exec -it 容器Id bash
```

#### 启动容器

```
docker start 容器id
```

#### 停止容器

```
docker stop 容器id
```









