# Yapi简介

YApi 是一个可本地部署的、打通前后端及QA的、可视化的接口管理平台,因本人目前所在团队经常因接口问题管理不当导致各类生产问题，目前急需一款专业的接口管理工具。
 Yapi是去哪儿网推出的一款开源API管理工具，当我第一次打开，说实话惊叹其小清新的界面



![img](https:////upload-images.jianshu.io/upload_images/15690304-9c3d59864df89e3b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

微信图片_20200306111858.jpg

# Yapi 搭建环境简介

> nodejs>7.6
>  mongodb>2.6
>  git

但是因为本人所在公司内外网分离，目前工具都无法在Centos上使用yum源或wget下载，所以需做以下准备：
 1.连接外网服务器下载好nodejs，传输至内网安装
 2.连接外网服务器下载好mongodb，传输至内网安装
 3.连接外网服务器下载好Yapi，并打包好，传输至内网安装
 4.连接外网服务器下载好PM2，并打包好，传输至内网安装
 **其中，外网服务器也许安装nodejs**
 我这边在网上找到了相关的资源下载包合集，可供大家下载

> 链接：[https://pan.baidu.com/s/1SuE4sMFIL19m0bhrGWkwDQ](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1SuE4sMFIL19m0bhrGWkwDQ)
>  提取码：gope

# Yapi安装过程

## 内外网服务器安装nodejs



```bash
mkdir nodejs
tar -zxvf node-v12.13.0-linux-x64.tar.xz
mv node-v12.13.0-linux-x64/* /usr/local/nodejs

#添加软链接到/usr/local/bin目录或配置环境变量
ln -s /usr/local/nodejs/bin/npm /usr/local/bin
ln -s /usr/local/nodejs/bin/node /usr/local/bin

# 配置环境变量在  /etc/profile文件中加入以下语句或添加软连接
# Nodejs

export NODEJS_HOME=/usr/local/bin/node/bin
export PATH=$NODEJS_HOME:$PATH

# 生效环境变量
source /etc/profile

# 检测环境是否生效，能显示版本号即说明安装成功
node -v  
npm -v
```

## 内网服务器安装mongodb



```bash
mkdir mongodb
tar -zxvf mongodb-linux-x86_64-3.0.6.tgz -C mongodb
mv mongodb-linux-x86_64-3.0.6/* /usr/local/mongodb

# 增加mongodb环境变量
# mongodb

export MONGODB_HOME=/usr/local/mongodb/bin
export PATH=$MONGODB_HOME:$PATH


# 生效环境变量
source /etc/profile

#检查mongodb环境变量是否生效,能显示版本号即说明安装成功
mongo --version

#配置mongodb配置文件信息
cd mongodb
mkdir data
touch mongo.log
vim mongodb.cnf


#配置信息详情
# 指定数据存储目录 需要提前创建
dbpath=/usr/local/mongodb/data/
# 指定日志文件
logpath=/usr/local/mongodb/mongo.log   
# 日志追加写    
logappend=true 
# 创建后台子进程
fork=true
# 指定端口号
port=27017


#进入bin目录，启动mongodbserver
./mongod -f /usr/local/mongodb/mongdb.cnf

#资源不足使用
./mongod --smallfiles --config /usr/local/mongodb/mongodb.conf

#连接本机的mongodb
cd /usr/local/mongodb/bin/
mongo

#当前所有数据库
show dbs

#创建用户名/密码
db.createUser({user:'root',pwd:'xxx', roles:[{role:'userAdminAnyDatabase', db:'admin'}]})
```

## 外网服务器安装yapi



```bash
mkdir yapi
cd yapi
git clone https://github.com/YMFE/yapi.git vendors
cp vendors/config_example.json ./config.json
cd vendors
npm install --production
# 将创建的yapi文件夹打成压缩包得到yapi.tar.gz(其目录下有config.json和vendors)
tar -czf yapi.tar.gz yapi

## 外网服务器安装PM2
npm install -S pm2
tar -czf PM2.tar.gz PM2
```

至此，外网过程做的准备工作已经完成，可以将打包好的yapi.tar.gz和PM2.tar.gz 传输回内网

## 内网服务器安装Yapi



```bash
tar -zxvf yapi.tar
cd yapi
# 配置config.json
{
  "port": "3000",
  "adminAccount": "admin@admin.com",
  "db": {
    "servername": "127.0.0.1",
    "DATABASE": "yapi",
    "port": 27017,
    "user": "root",
    "pass": "xxx",
    "authSource": "admin"
  },
  "mail": {
    "enable": false,
    "host": "smtp.exmail.qq.com",
    "port": 465,
    "from": "xxx@xxx.cn",
    "auth": {
      "user": "xxx@xxx.cn",
      "pass": "xxx"
    }
  }
}
# 初始化数据库
cd vendors
npm run install-server
# 启动yapi server
node server/app.js
```

*浏览器访问 ip:3000 yapi接口管理平台*

*默认的管理员为[admin@admin.com](https://links.jianshu.com/go?to=mailto%3Aadmin%40admin.com) 密码[ymfe.org](https://links.jianshu.com/go?to=http%3A%2F%2Fymfe.org)*

## 后台运行

1. \# 安装 screen

    yum install screen -y

2. \# 新建一个名为 yapi 的进程

   screen -S yapi

   node server/app.js







## 离线安装PM2



```bash
#查看服务器的npm默认安装目录 
npm config get prefix

#如果目录是 /usr/local/nodejs
cd /usr/local/nodejs/lib/node_modules/

#拷贝 pm2.tar.gz 到该目录后解压
tar xvf pm2.tar.gz

#添加软链接
ln -s /usr/local/nodejs/lib/node_modules/pm2/bin/pm2  /usr/local/bin
```

## 用pm2启动和重启YApi



```bash
#启动 --watch参数，意味着当你的express应用代码发生变化时，pm2会帮你重启服务
pm2 start /usr/local/yapi/vendors/server/app.js --watch
#重启
pm2 restart /usr/local/yapi/vendors/server/app.js
```

# Yapi使用

关于Yapi的使用，目前去哪儿网上有详细的demo，可阅读使用



