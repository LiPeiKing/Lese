# MySQL借助DBeaver工具数据导入导出

## 摘要：

DBeaver是一款数据库管理软件，小巧易用，最主要其官方版就可以满足平常得任务需求。对于力主使用正版软件工具的公司和单位来说，它是操作MySQL数据库的比较好得选择。本次仅介绍MySQL的数据迁移的工作，将一个DB服务器上的约200万条数据迁移到另外一台DB服务器。


## 下载MySQL组件

为什么需要下载MySQL组件呢？因为我们的MySQL安装在服务器上，本地只有DBeaver，而DBeaver导入导出数据需要MySQL Client的支持，所以需要下载之。若本地装有MySQL，只需要找到MySQL安装包bin下面的几个.exe执行文件。

在官网下载，这里需要注意的是MySQL现在已经发展到8.0了，需要下载与MySQL服务器尽量大版本相一致的，如服务器装的是5.*的版本，这里也选择一个5.*的版本，否则就会遇到一些问题（文章最后出现的问题）。[MySql官网下载](https://dev.mysql.com/downloads/mysql/) 。将下载的压缩包解压，待用。

![image-20211221094422546](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211221094422546.png)

![image-20211221094545616](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211221094545616.png)

## 配置DBeaver

DBeaver需要配置MySQL Client，这一步骤是最主要的。如果新建连接，只需要在连接设置中选择Local Client 的位置，就是我们上一步的解压缩文件。若是已经建立的连接，只需要编辑连接设置一下即可， 具体操作看图。

![image-20211221094642077](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211221094642077.png)   

![image-20211221094709674](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211221094709674.png)

这一步骤可以通过远程下载，也可以添加本地下载好的文件

![image-20211221094835960](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211221094835960.png)



点击取消添加本地下载好的文件

![image-20211221094907990](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211221094907990.png)





## 执行导出-导入

![image-20211221103606275](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211221103606275.png)











