# EhCache Monitor的使用

##### 1、在[地址](http://ehcache.org/documentation/monitor.html#Installation_And_Configuration)下载ehcache-monitor-kit-1.0.0-distribution.tar.gz包

##### 2.解压缩到目录下，复制ehcache-monitor-kit-1.0.0\lib\ehcache-probe-1.0.0.jar包到application的web-inf/lib目录下

##### 3.将以下配置copy的ehcache.xml文件的ehcache标签中，注：上述链接中说的配置少写了个probe包名。

```xml
<cacheManagerPeerListenerFactory 
	class="org.terracotta.ehcachedx.monitor.probe.ProbePeerListenerFactory"
	properties="monitorAddress=localhost, monitorPort=9889" />
```

##### 4.在\ehcache-monitor-kit-1.0.0\etc\ehcache-monitor.conf中可以配置监控的ip和端口号。

##### 5.启动被监控的web application和ehcache-monitor-kit-1.0.0\bin目录下的startup.bat（在windows环境下）

注意，使用的时候将startup.bat文件中的jetty配置文件删除

##### 6.在浏览器中输入 http://localhost:9889/monitor/即可开始监控。

![image-20210930174416610](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20210930174416610.png)

