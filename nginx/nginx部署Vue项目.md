# nginx部署Vue项目

### 上传Vue项目到指定目录

### 修改nginx默认配置

`vim /etc/nginx/conf.d/default.conf`

```shell
server {
    listen       9999;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /realtime/5g_app/network/html;
        index  index.html index.htm;
        try_files $uri $uri/ @router;
    }
	location @router {
		# last ：相当于Apache里德(L)标记，【表示完成rewrite ！important】【将得到的路径重新进行一次路径匹配】，浏览器地址栏【URL地址不变】
		# break；本条规则匹配完成后，【终止匹配  ！important】，【不再匹配后面的规则】，浏览器地址栏【URL地址不变】 一般不用这个选项！
		# redirect： 返回302临时重定向，浏览器地址【会显示跳转后的URL地址】
		# permanent：返回301永久重定向，浏览器地址栏【会显示跳转后的URL地址】
		# 1.静态资源，去掉项目名，进行定向 \为转义， nginx 中的 / 不代表正则，所以不需要转义
		rewrite (static/.*)$ /$1   redirect;
		# 2.非静态资源，直接定向index.html
		rewrite ^.*$   /index.html  last;
	}
	# 后端接口，反向代理  
	location /api {
		#  反向代理
		rewrite  ^.+api/?(.*)$ /$1 break;
		include  uwsgi_params;
		proxy_pass http://127.0.0.1:9300;
	}

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
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

```

### 问题汇总

### 1.查看错误日志

```shell
cat /var/log/nginx/error.log
```

### 2.403 forbidden

#### 查看nginx的启动用户

`ps aux | grep “nginx: worker process” | awk’{print $1}’`

nginx.conf配置用户需和启动用户保持一致

`vim /etc/nginx/nginx.cong `  

![在这里插入图片描述](https://img-blog.csdnimg.cn/c77bc00213594358b0a92c7ebbadc332.png)



### 查看是否缺少index.html或者index.php文件



### 权限问题，如果nginx没有web目录的操作权限，也会出现403错误

```bash
chmod -R 777 /data/www/
```

### SELinux设置为开启状态

+ 查看当前 selinux 的状态

```bash
/usr/sbin/sestatus
运行命令getenforce，验证SELinux状态。
返回状态如果是enforcing，表明SELinux已开启。
```

+ 将SELINUX=enforcing 修改为 SELINUX=disabled 状态。

```bash
vim /etc/selinux/config

#SELINUX=enforcing
ELINUX=disabled
```

+ 重启生效。reboot。



执行命令`setenforce 0`临时关闭SELinux。