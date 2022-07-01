# ![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB4AAAAeCAYAAAA7MK6iAAAAAXNSR0IArs4c6QAABH1JREFUSA3tVl1oHFUUPmdmd2ltklqbpJDiNnXFmgbFktho7YMPNiJSSZM0+CAYSkUELVhM6YuwIPpgoOKDqOBDC0XE2CQoNtQXBUFTTcCi+Wlh1V2TQExsUzcltd3M9Tt3ZjZzZ2fT+OJTL8yeM+eee757fmeJbq//KQL8X3DUSFOcfr7cRsRtxNQMWueeVzOkaITIGqQHNg5y8+jNW9ldM7A6nTpAjuolUikAwq7CE3WcM2RRDz+XGVgN3FptU/aUSlvq9Pa3iZ1+sgAqJyyAFqkipd9dqiwHF3P65YycLWc/6sqGrvoEoIp6DOFaX5h6+dnfjkWprwqsPk0dUGq5vySwDImC10KxFHgGL1SWoc92O3eVht09qdXNH11I2SsTsJYqMWzihqGMi+A+Garf3BAuuLI5oGlULyNfyB/HYNujwktOfRrMr5t77NmevqaUopx0grnKAyvVpmwUDB4x6FPXuGvYLTDwWsejwgtgkYKPqRJg8SV6xaiZ3ZTppGneS4yfH5/66fZSDHv+QZci/+h5c5UHtpy67JUqGppM0sh0Nc1dW6/N1W5Yoqat8/TU/VnadmdeW2PLLSyh0cvxBs3KbqTmwYPpxN4do/mzE8nEpvX/UMu2Wbp74zUAK5q6WkHns7V0eWkdPbPzd3rxkTGybadYySumVzhcaJFbs5UrEkQ/+CK8gF5dnh/6ciIZ73gwQ927L1IitoxKLXYP3SjYdOrHHfTZhRRlFyrorafPk20B3HPD1y2G3qKZME5Jcf3t/HUC13/8tSd++vqFveMUTwAUxSUFI1QekR1+bIze3D9MF2aq6cPvG72CgnldWCFqyRw3lwH8ZMerjTD9ElRO7Gv44wNpC90aASqGfVlz/Rx17srQ57/UU26hkhQqUB7dBR71WmzQhHUnblGmVOEw0jhbV1n9OlXUDCIRGaNV5Jp43N516fN7JmnTHdfp7Hgy0luO4aMhtkLL8Bi3bUWYvzh5Mn1dTxrL6QmGuRhGL/TiTTxRoEdTszSaq9GR0NGA3KdkOz3hqSV3MIDhQ5IVX/Ivx3umBti2es2h4eZby7x8br1rkf7Mo90AqC8aQ3sJeNzqFRu+vSANAQe3PL7l0HGOAdwDCeZYvNKeoZp1Qfs6Aipndh86HmFRi0LAnEO47wsqM6cdfjh3jBPUzhZy7nvlUfFsamED1VQt6aISHVymXZ/B2aCtIG8AI8xfobj2d3en1wWVhOeHELKmLQ1s211s88comkv4UCwWyF787mJdYXtNfhKAXVqnKTq8QZvGAGGOfaTo5pGZ/PwbUCr5+DPr/1J92JNHr9aOl/F3iI5+O1nfybsGxoimvZ3ViWSluDITw3P37mypheDIPY0tw7+O/5ApbkYw+zpfaUVu32Pi98+defdUhEpZkRFq0aqyNh9FuL9hpYbEm6iwi0z2REd09ZmyENEbuhjDWzKvZXTqKYaBIr3tt5kuPtQBZFvEUwHt60vfCNu41XsksH9Ij1BMMz1Y0OOunHNShFIP5868g5zeXmuLwL9T4b6Q2+KejgAAAABJRU5ErkJggg==)Jenkins docker部署及初始配置原创

> 关于docker的安装部署，这里暂先略掉。

## 1，先下载Jenkins的镜像。

```shell
[root@localhost ~]$docker pull jenkinsci/blueocean
Using default tag: latest
latest: Pulling from jenkinsci/blueocean
ff3a5c916c92: Pull complete
5de5f69f42d7: Pull complete
fd869c8b9b59: Pull complete
09b77ebbb585: Pull complete
edaab8c638eb: Pull complete
74718d164167: Pull complete
6d6dab0396d5: Pull complete
f6a4487e3af7: Pull complete
4b44a66deffd: Pull complete
925e68272890: Pull complete
428de9991230: Pull complete
f4b1af398075: Pull complete
425d68489478: Pull complete
e2f8bb400606: Pull complete
251d043c8226: Pull complete
9a6a8fe90c4f: Downloading [==============================================>    ]  62.19MB/66.73MB
9a6a8fe90c4f: Pull complete
Digest: sha256:df6669eab61f76cba7b51847ef24623f65902499925b4d5a2516d155dc95c0c5
Status: Downloaded newer image for jenkinsci/blueocean:latest
```

##  2，查看镜像。

```shell
[root@localhost ~]$docker images
REPOSITORY           TAG             IMAGE ID            CREATED             SIZE
jenkinsci/blueocean  latest          52e299133c9c        20 hours ago        438MB

```

##  3，将镜像启动成容器。

```shell
# 容器内不启动docker
docker run -d  \
--name Jenkins -u root \
-p 9090:8080  \
-v /var/jenkins_home:/var/jenkins_home  \
jenkinsci/blueocean

# 容器内启动docker
docker run \
  -d \
  --rm \
  --name jk -u root \
  -p 9091:8080 \
  -v /var/jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  jenkinsci/blueocean

```

说明：

> - 1，–name 是指定生成的容器名称。
> - 2，最好使用root启动，以免有权限问题而启动失败。
> - 3，-p是端口的映射，冒号前边是宿主机的端口，冒号后边的是容器的端口。
> - 4，-v将Jenkins容器的Jenkins_home映射到宿主机的目录中，实现数据持续化。
> - 5, -rm 命令选项，等价于在容器退出后，执行`docker rm -v`

##  4，查看启动的容器。

```shell
[root@localhost ~]$docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                               NAMES
19e670dc09b0        jenkinsci/blueocean   "/sbin/tini -- /usr/…"   4 seconds ago       Up 3 seconds        50000/tcp, 0.0.0.0:9090->8080/tcp   jenkins
```

启动之后就可以直接通过宿主机ip+映射的端口进行访问了。

##  5，初入Jenkins。

注意刚进入之后会有一个坑。

![image](http://t.eryajf.net/imgs/2021/09/c902710b00a7750f.jpg)

这里看到指定了一个路径，去到var下去找，但是这个目录下并没有这个目录，可能这是镜像默认的一个路径，但是事实却并非如此。

那么我们就要用一下find 大法了（其实在/var/lib/docker/volumes/c727610d35c19516dc279ffb9cc8ef68a3ff9e8ef4798a4e2730a1223f22d372/secrets/initialAdminPassword之中。）。

![image](http://t.eryajf.net/imgs/2021/09/48b197e8e245c58e.jpg)

这里是我之前起了几次容器才会有多个文件，根据时间判断是哪个就行了。

接下来去到容器当中看看。

```shell
[root@localhost mnt]$docker exec -it jk bash
bash-4.4#
bash-4.4# ls
bin  etc   lib	  mnt	root  sbin  sys  usr
dev  home  media  proc	run   srv   tmp  var
bash-4.4#
```

`注意这里容器中使用apk命令来管理应用的安装配置，类似于系统中的yum命令。`

## 6，基础软件安装。

### 1，maven。

```shell
bash-4.4# apk add maven
OK: 402 MiB in 107 packages
bash-4.4# apk search ansible
ansible-2.4.1.0-r0
ansible-doc-2.4.1.0-r0
bash-4.4# apk add ansible
OK: 402 MiB in 107 packages
bash-4.4# apk add ansible-doc
OK: 402 MiB in 107 packages
```

接着查看一下这些工具在容器中是安在了什么地方。

```shell
bash-4.4# apk info -L maven
maven-3.8.3-r0 contains:
etc/mavenrc
usr/bin/mvn
usr/bin/mvnDebug
usr/bin/mvnyjp
usr/share/java/maven-3/bin/m2.conf
usr/share/java/maven-3/bin/mvn
usr/share/java/maven-3/bin/mvnDebug
usr/share/java/maven-3/bin/mvnyjp
```

安装路径：`usr/share/java/maven-3`

###  2，git。

```shell
bash-4.4# apk info -L git
git-2.34.2-r0 contains:
usr/bin/git
usr/bin/git-receive-pack
usr/bin/git-shell
usr/bin/git-upload-archive
usr/bin/git-upload-pack
```

安装路径：`usr/bin/`

### 3，jdk。

```shell
bash-4.4# echo $JAVA_HOME
/opt/java/openjdk
```

安装路径：`/opt/java/openjdk`

## 7，Jenkins中的基础配置

我们来到系统设置–》全局工具配置页面。

依然是一些工具的配置。

刚才已经全部看过了，，现在配置如下。

![image](http://t.eryajf.net/imgs/2021/09/dd61b6a41087807c.jpg)

![image](http://t.eryajf.net/imgs/2021/09/6f41d18d34ddc37f.jpg)

上边都是一些基本工作，必须要做的，不然下边的事儿都免谈了。











## 附录

### apk命令的用法

```shell
bash-4.4# apk -h
apk-tools 2.8.2, compiled for x86_64.
usage: apk COMMAND [-h|--help] [-p|--root DIR]
           [-X|--repository REPO] [-q|--quiet]
           [-v|--verbose] [-i|--interactive]
           [-V|--version] [-f|--force]
           [--force-binary-stdout]
           [--force-broken-world]
           [--force-non-repository]
           [--force-old-apk] [--force-overwrite]
           [--force-refresh] [-U|--update-cache]
           [--progress] [--progress-fd FD]
           [--no-progress] [--purge]
           [--allow-untrusted] [--wait TIME]
           [--keys-dir KEYSDIR]
           [--repositories-file REPOFILE]
           [--no-network] [--no-cache]
           [--cache-dir CACHEDIR] [--arch ARCH]
           [--print-arch] [ARGS]...
 
The following commands are available:
  add       Add PACKAGEs to 'world' and install
            (or upgrade) them, while ensuring
            that all dependencies are met
  del       Remove PACKAGEs from 'world' and
            uninstall them
  fix       Repair package or upgrade it without
            modifying main dependencies
  update    Update repository indexes from all
            remote repositories
  info      Give detailed information about
            PACKAGEs or repositories
  search    Search package by PATTERNs or by
            indexed dependencies
  upgrade   Upgrade currently installed packages
            to match repositories
  cache     Download missing PACKAGEs to cache
            and/or delete unneeded files from
            cache
  version   Compare package versions (in
            installed database vs. available) or
            do tests on literal version strings
  index     Create repository index file from
            FILEs
  fetch     Download PACKAGEs from global
            repositories to a local directory
  audit     Audit the directories for changes
  verify    Verify package integrity and
            signature
  dot       Generate graphviz graphs
  policy    Show repository policy for packages
  stats     Show statistics about repositories
            and installations
  manifest  Show checksums of package contents
 
Global options:
  -h, --help              Show generic help or
                          applet specific help
  -p, --root DIR          Install packages to DIR
  -X, --repository REPO   Use packages from REPO
  -q, --quiet             Print less information
  -v, --verbose           Print more information
                          (can be doubled)
  -i, --interactive       Ask confirmation for
                          certain operations
  -V, --version           Print program version
                          and exit
  -f, --force             Enable selected
                          --force-* (deprecated)
  --force-binary-stdout   Continue even if binary
                          data is to be output
  --force-broken-world    Continue even if
                          'world' cannot be
                          satisfied
  --force-non-repository  Continue even if
                          packages may be lost on
                          reboot
  --force-old-apk         Continue even if
                          packages use
                          unsupported features
  --force-overwrite       Overwrite files in
                          other packages
  --force-refresh         Do not use cached files
                          (local or from proxy)
  -U, --update-cache      Update the repository
                          cache
  --progress              Show a progress bar
  --progress-fd FD        Write progress to fd
  --no-progress           Disable progress bar
                          even for TTYs
  --purge                 Delete also modified
                          configuration files
                          (pkg removal) and
                          uninstalled packages
                          from cache (cache
                          clean)
  --allow-untrusted       Install packages with
                          untrusted signature or
                          no signature
  --wait TIME             Wait for TIME seconds
                          to get an exclusive
                          repository lock before
                          failing
  --keys-dir KEYSDIR      Override directory of
                          trusted keys
  --repositories-file REPOFILE Override
                          repositories file
  --no-network            Do not use network
                          (cache is still used)
  --no-cache              Do not use any local
                          cache path
  --cache-dir CACHEDIR    Override cache
                          directory
  --arch ARCH             Use architecture with
                          --root
  --print-arch            Print default arch and
                          exit
 
This apk has coffee making abilities.
```



### apk国内镜像源进行加速

采用国内阿里云的源，文件内容为：/etc/apk/repositories

https://mirrors.aliyun.com/alpine/v3.6/main/
https://mirrors.aliyun.com/alpine/v3.6/community/

命令：`sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories`





### jenkins初始化提示“No such plugin: cloudbees-folder”：

https://www.jb51.cc/docker/1041355.html



























