# 基于Docker的YApi搭建

### YApi简介

旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API。

#### 权限管理

YApi 成熟的团队管理扁平化项目权限配置满足各类企业的需求

#### 可视化接口管理

基于 websocket 的多人协作接口编辑功能和类 postman 测试工具，让多人协作成倍提升开发效率

#### Mock Server

易用的 Mock Server，再也不用担心 mock 数据的生成了

#### 自动化测试

完善的接口自动化测试,保证数据的正确性

#### 数据导入

支持导入 swagger, postman, har 数据格式，方便迁移旧项目

#### 插件机制

强大的插件机制，满足各类业务需求

### Docker的搭建与配置

请参考之前写的文章：
Docker搭建

### 搭建YApi

#### 1. 由于YApi依赖于MongoDB，所以我们需要下载并启动MongoDB。

```java
docker run -d --name mongo-yapi mongo
```

![图片.png](https://segmentfault.com/img/remote/1460000022685999)

#### 2. 下载YApi镜像

```java
docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi
```

![图片.png](https://segmentfault.com/img/remote/1460000022686000)

#### 3. 初始化MongoDB数据库索引及创建管理员账号

```java
  docker run -it --rm \
  --link mongo-yapi:mongo \
  --entrypoint npm \
  --workdir /api/vendors \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  run install-server
```

![图片.png](https://segmentfault.com/img/remote/1460000022686001)
上图可知管理员账号和密码为：
++初始化管理员账号成功,账号名："admin@admin.com"，密码："ymfe.org"++

#### 4. 启动yapi服务

```java
  docker run -d \
  --name yapi \
  --link mongo-yapi:mongo \
  --workdir /api/vendors \
  -p 3000:3000 \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  server/app.js
```

![图片.png](https://segmentfault.com/img/remote/1460000022686002)

#### 5. 登陆YApi

访问 ：[http://你的ip地址](https://link.segmentfault.com/?enc=3%2FwELKtXrjIVvtHGCorP8A%3D%3D.V3Rysj1GE%2FLYiDfpTYshRDoYwqWwyg9twW5RAiGTL3AtbGrJwi9U6vhbNbs28wyW):3000
账号 ：admin@admin.com
密码 ：ymfe.org
![图片.png](https://segmentfault.com/img/remote/1460000022686004)
登陆之后：
![图片.png](https://segmentfault.com/img/remote/1460000022686003)
好了，尽情开始吧！

### 个人网站链接