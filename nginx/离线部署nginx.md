# Linux 中离线安装 Nginx

### **下载并安装依赖包**  

该压缩包内包含了 Nginx-1.18.0以及 Nginx所需要的依赖库。依赖库主要为：

- 编译 Nginx 的GCC 编译器；
- 未来使用 C++ 来编写 Nginx 的 G++ 编译器；
- Perl 正则表达式（Nginx HTTP 模块依赖库）；
- zlib （网络数据包 gzip压缩依赖库）；
- openssl （提供HTTPS 支持以及 MD5、SHA1 等加密算法实现）。

执行`sudo rpm -Uvh *.rpm --nodeps –force`进行安装：

执行后，接着验证安装是否成功： `gcc -v`

### **安装Nginx**  

```shell
rpm -ivh nginx-1.20.2-1.el7.ngx.x86_64.rpm
```

#### 查看nginx版本

`nginx -v`