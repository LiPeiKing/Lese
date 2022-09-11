# nginx

# 安装

略

### 启动

两种方法：

1） 直接双击该目录下的"nginx.exe"，即可启动nginx服务器；

2） 命令行进入该文件夹，执行start nginx命令，也会直接启动nginx服务器。

### 验证

开浏览器，输入地址：http://localhost:80，访问页面，出现如下页面表示访问成功。

### Nginx Windows基本操作命令

启动服务：start nginx
退出服务：nginx -s quit
强制关闭服务：nginx -s stop
重载服务：nginx -s reload　　（重载服务配置文件，类似于重启，服务不会中止）
验证配置文件：nginx -t
使用配置文件：nginx -c "配置文件路径"
使用帮助：nginx -h

### Nginx Linux基本操作指令

启动服务：nginx
退出服务：nginx -s quit
强制关闭服务：nginx -s stop
重载服务：nginx -s reload　　（重载服务配置文件，类似于重启，但服务不会中止）
验证配置文件：nginx -t
使用配置文件：nginx -c "配置文件路径"
使用帮助：nginx -h



## 配置

### 配置介绍

在项目使用中，使用最多的三个核心功能是静态服务器、反向代理和负载均衡。

这三个不同的功能的使用，都跟Nginx的配置密切相关，Nginx服务器的配置信息主要集中在"nginx.conf"这个配置文件中，并且所有的可配置选项大致分为以下几个部分.

```properties
main                                # 全局配置
 
events {                            # 工作模式配置
 
}
 
http {                              # http设置
    ....
 
    server {                        # 服务器主机配置（虚拟主机、反向代理等）
        ....
        location {                  # 路由配置（虚拟目录等）
            ....
        }
 
        location path {
            ....
        }
 
        location otherpath {
            ....
        }
    }
 
    server {
        ....
 
        location {
            ....
        }
    }
 
    upstream name {                  # 负载均衡配置
        ....
    }
}
```

#### main模块

+ user    用来指定nginx worker进程运行用户以及用户组，默认nobody账号运行
+ worker_processes    指定nginx要开启的子进程数量，运行过程中监控每个进程消耗内存(一般几M~几十M不等)根据实际情况进行调整，通常数量是CPU内核数量的整数倍
+ error_log    定义错误日志文件的位置及输出级别【debug / info / notice / warn / error / crit】
+ pid    用来指定进程id的存储文件的位置
+ worker_rlimit_nofile    用于指定一个进程可以打开最多文件数量的描述
+ ...

#### event模块

+ worker_connections    指定最大可以同时接收的连接数量，这里一定要注意，最大连接数量是和worker processes共同决定的。
+ multi_accept    配置指定nginx在收到一个新连接通知后尽可能多的接受更多的连接
+ use epoll    配置指定了线程轮询的方法，如果是linux2.6+，使用epoll，如果是BSD如Mac请使用Kqueue
+ ...

#### http模块

作为web服务器，http模块是nginx最核心的一个模块，配置项也是比较多的，项目中会设置到很多的实际业务场景，需要根据硬件信息进行适当的配置。

##### 1）基础配置

+ sendfile on：配置on让sendfile发挥作用，将文件的回写过程交给数据缓冲去去完成，而不是放在应用中完成，这样的话在性能提升有有好处
+ tcp_nopush on：让nginx在一个数据包中发送所有的头文件，而不是一个一个单独发
+ tcp_nodelay on：让nginx不要缓存数据，而是一段一段发送，如果数据的传输有实时性的要求的话可以配置它，发送完一小段数据就立刻能得到返回值，但是不要滥用哦
+ keepalive_timeout 10：给客户端分配连接超时时间，服务器会在这个时间过后关闭连接。一般设置时间较短，可以让nginx工作持续性更好
+ client_header_timeout 10：设置请求头的超时时间
+ client_body_timeout 10:设置请求体的超时时间
+ send_timeout 10：指定客户端响应超时时间，如果客户端两次操作间隔超过这个时间，服务器就会关闭这个链接
+ limit_conn_zone $binary_remote_addr zone=addr:5m ：设置用于保存各种key的共享内存的参数，
+ limit_conn addr 100: 给定的key设置最大连接数
+ server_tokens：虽然不会让nginx执行速度更快，但是可以在错误页面关闭nginx版本提示，对于网站安全性的提升有好处哦
+ include /etc/nginx/mime.types：指定在当前文件中包含另一个文件的指令
+ default_type application/octet-stream：指定默认处理的文件类型可以是二进制
+ type_hash_max_size 2048：混淆数据，影响三列冲突率，值越大消耗内存越多，散列key冲突率会降低，检索速度更快；值越小key，占用内存较少，冲突率越高，检索速度变慢

##### 2）日志配置

+ access_log logs/access.log：设置存储访问记录的日志
+ error_log logs/error.log：设置存储记录错误发生的日志

##### 3）SSL证书配置

+ ssl_protocols：指令用于启动特定的加密协议，nginx在1.1.13和1.0.12版本后默认是ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2，TLSv1.1与TLSv1.2要确保OpenSSL >= 1.0.1 ，SSLv3 现在还有很多地方在用但有不少被攻击的漏洞。
+ ssl prefer server ciphers：设置协商加密算法时，优先使用我们服务端的加密套件，而不是客户端浏览器的加密套件

##### 4）压缩配置

+ gzip 是告诉nginx采用gzip压缩的形式发送数据。这将会减少我们发送的数据量。

+ gzip_disable 为指定的客户端禁用gzip功能。我们设置成IE6或者更低版本以使我们的方案能够广泛兼容。

+ gzip_static 告诉nginx在压缩资源之前，先查找是否有预先gzip处理过的资源。这要求你预先压缩你的文件（在这个例子中被注释掉

  了），从而允许你使用最高压缩比，这样nginx就不用再压缩这些文件了（想要更详尽的gzip_static的信息，请点击这里）。   

+ gzip_proxied 允许或者禁止压缩基于请求和响应的响应流。我们设置为any，意味着将会压缩所有的请求。
+ gzip_min_length 设置对数据启用压缩的最少字节数。如果一个请求小于1000字节，我们最好不要压缩它，因为压缩这些小的数据会降低处理此请求的所有进程的速度。
+ gzip_comp_level 设置数据的压缩等级。这个等级可以是1-9之间的任意数值，9是最慢但是压缩比最大的。我们设置为4，这是一个比较折中的设置。
+ gzip_type 设置需要压缩的数据格式。上面例子中已经有一些了，你也可以再添加更多的格式。

##### 5）文件缓存配置

+ open_file_cache 打开缓存的同时也指定了缓存最大数目，以及缓存的时间。我们可以设置一个相对高的最大时间，这样我们可以在它们不活动超过20秒后清除掉。
+ open_file_cache_valid 在open_file_cache中指定检测正确信息的间隔时间。
+ open_file_cache_min_uses 定义了open_file_cache中指令参数不活动时间期间里最小的文件数。
+ open_file_cache_errors 指定了当搜索一个文件时是否缓存错误信息，也包括再次给配置中添加文件。我们也包括了服务器模块，这些是在不同文件中定义的。如果你的服务器模块不在这些位置，你就得修改这一行来指定正确的位置。

#### sever模块

srever模块配置是http模块中的一个子模块，用来定义一个虚拟访问主机，也就是一个虚拟服务器的配置信息。

+ erver：一个虚拟主机的配置，一个http中可以配置多个server
+ server_name：用来指定ip地址或者域名，多个配置之间用空格分隔
+ charset：用于设置www/路径中配置的网页的默认编码格式
+ access_log：用于指定该虚拟主机服务器中的访问记录日志存放路径
+ error_log：用于指定该虚拟主机服务器中访问错误日志的存放路径
  

#### location模块

location模块是Nginx配置中出现最多的一个配置，主要用于配置路由访问信息。

在路由访问信息配置中关联到反向代理、负载均衡等等各项功能，所以location模块也是一个非常重要的配置模块。

##### 1）基本配置

- location /：表示匹配访问根目录
- root：用于指定访问根目录时，访问虚拟主机的web目录
- index：在不指定访问具体资源时，默认展示的资源文件列表

##### 2）反向代理配置

通过反向代理代理服务器访问模式，通过proxy_set配置让客户端访问透明化。

location / {
    proxy_pass http://localhost:8888;
    proxy_set_header X-real-ip $remote_addr;
    proxy_set_header Host $http_host;
}

#### 负载均衡模块（upstream）

upstream模块主要负责负载均衡的配置，通过默认的轮询调度方式来分发请求到后端服务器。简单的配置方式如下。

```xml
upstream name {
    ip_hash;
    server 192.168.1.100:8000;
    server 192.168.1.100:8001 down;
    server 192.168.1.100:8002 max_fails=3;
    server 192.168.1.100:8003 fail_timeout=20s;
    server 192.168.1.100:8004 max_fails=3 fail_timeout=20s;
}
```



+ ip_hash：指定请求调度算法，默认是weight权重轮询调度，可以指定

+ server host:port：分发服务器的列表配置

  ​	-- down：表示该主机暂停服务
  ​	-- max_fails：表示失败最大次数，超过失败最大次数暂停服务
  ​	-- fail_timeout：表示如果请求受理失败，暂停指定的时间之后重新发起请求

### Nginx常用配置

#### 静态Http服务器配置

首先，Nginx是一个HTTP服务器，可以将服务器上的静态文件（如HTML、图片）通过HTTP协议展现给客户端。
配置：

```pr
server {
    listen 80; 　　# 端口
    server_name localhost  192.168.1.100; 　　# 域名   
    location / {　　　　　　　　　　　　　# 代表这是项目根目录
        root /usr/share/nginx/www; 　 # 虚拟目录
    }
}
```

#### 反向代理服务器配置

```properties
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://192.168.0.112:8080; 　　# 应用服务器HTTP地址
    }
}
```

#### 负载均衡配置

```properties
upstream myapp {
    server 192.168.0.111:8080; 　　# 应用服务器1
    server 192.168.0.112:8080; 　　# 应用服务器2
}
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://myweb;
    }
}
```

#### 虚拟主机配置

```properties
upstream  testapi {  
	server www.huzd.info;
}
server {
    listen 9090;
    server_name localhost;
    location /www/ {
        proxy_pass http://testapi/; 斜杠不能少
    }
}
server {
    listen 9091;
    server_name localhost;
    location /www/ {
        proxy_pass http://testapi/; 斜杠不能少
    }
}
```







```properties

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
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
	upstream pagefault {
        server www.pagefault.info;
    }
	
	upstream  testapi {
        server www.huzd.info;
    }
	

    server {
        listen       8090;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
	server {
        listen 8099;
        server_name localhost;

		location /www {
			proxy_pass http://pagefault/;
		}

    }
	
	server {
        listen       8093;
        server_name  localhost;

		location /www/ {
			proxy_pass http://testapi/;
		}
		location /pro/ {
			proxy_pass http://testapi/;
		}
    }


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
	

}

```























