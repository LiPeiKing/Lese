# ![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB4AAAAeCAYAAAA7MK6iAAAAAXNSR0IArs4c6QAABGpJREFUSA3tVVtoXFUU3fvOI53UlmCaKIFmwEhsE7QK0ipFEdHEKpXaZGrp15SINsXUWvBDpBgQRKi0+KKoFeJHfZA+ED9KKoIU2gYD9UejTW4rVIzm0VSTziPzuNu1z507dibTTjL4U/DAzLn3nL3X2o91ziX6f9wMFdh6Jvbm9nNSV0msViVO6tN1Rm7NMu2OpeJ9lWBUTDxrJbYTS0hInuwciu9eLHlFxCLCZEk3MegsJmZ5K/JD6t7FkFdEvGUo1g7qJoG3MHImqRIn8/nzY1K9UPKKiJmtnUqHVE3Gbuay6vJE/N2FEmuxFjW2nUuE0yQXRRxLiTUAzs36zhZvOXJPdX850EVnnLZkB8prodQoM5JGj7Xk2mvC7JB8tG04Ef5PiXtG0UtxupRQSfTnBoCy554x18yJHI6I+G5Eru4LHmPJZEQsrvPUbMiA8G/WgMK7w7I+ez7++o2ANfbrjvaOl1tFMs+htG3IrZH9/hDX1Pr8Tc0UvH8tcX29KzAgIGcEkINyW5BF9x891hw6VYqgJHEk0huccS7vh3C6gTiODL+26huuBtbct8eZnqLML8PkxGYpuPZBqtqwkSjgc4mB5gbgig5i+y0UDK35LMxXisn9xQtK+nd26gTIHsHe/oblK/b29fUmN/8Y+9jAQrnBp56m1LcDlDp9irKTExSKduXJVWSqdBMA08pEJnEIOB3FPPMybu/oeV8zFeYN3xx576Q6RH+VmplE4ncQV5v+5rzSoyOU7PuEAg8g803PwBJ0CExno/jcMbN8tONYeOmHiuUNryvm3fRUy4tMPVLdAGkUhNWuggGrJcXPv+ouCjz0MKUHz1J2/E8IC9nqTabcxgaBYM0hPhD5Y65FsbxRQKxCQrDjDctW7PUM3HuZunFyifSAqEfuzCp48Il24luWUWZoyJCaPR82jE0+kFA643wRFVni4RYSq3ohJO2pZ7B5dO4xkDWbEpossJPLSrPjYID8rS2UHTlvyNxqIGsg674XJJ7vnh5L7PNwC4hh2sjCI96mzszOTpxLF0T7l88Yz7lAuK6OnL8gXLOnTvpzSb22YG8W7us3jSebFHeeqnXRG1vt+MoUM84LQIBmMsCTAcOauTh0T0l0neQK7m2bLMt2mGxU3HYssS0J2cdv5wljlPsrIuZLAG/2DOZIXgCYT8uMGZN+e2kSirfxZOPCsC0f24nTZzspnVn9VePS1Z5vubmAGGXG8ZFno9Hel0yfA5ZPhF7Dh972BQJ2qCpgH67lmWtBYbvk6sz02wjky2vXyz0XErP/kFB619js1BtwfOV4OPRqOQBjy3Qbk18vigUPPSD5ceHnwck7W9bhAqZdd7SuG7w4/P2F/GaJh8c7e9qgow+Q7cGBo+98WsLkuktFqiZabtXuQTu/Y5ETbR0v7tNSFnvrmu6pjdoan2KjMu8q/Hmj1EfCO2ZGfEIbIXKUlw8qaX9/b2oeSJmFksSeT/Fn0V3nSypChh4Gjh74ybO9aeZ/AN2dwciu2/MhAAAAAElFTkSuQmCC)Jenkins根目录详解

Jenkins启动之后，/var/jenkins_home目录就是Jenkins的家目录。当然这个目录的位置也是可以更改的，具体更改的办法，随便百度一下就会有结果。

先来看看长什么样子。

```shell
[root@elwin-centos8 jenkins_home]# ll
总用量 188
-rw-r--r--   1 root root   789 6月   1 10:08 com.dabsquared.gitlabjenkins.connection.GitLabConnectionConfig.xml
-rw-r--r--   1 root root   366 6月   1 10:08 com.dabsquared.gitlabjenkins.GitLabPushTrigger.xml
-rw-r--r--   1 root root   888 6月   1 09:21 com.gitee.jenkins.connection.GiteeConnectionConfig.xml
-rw-r--r--   1 root root   424 6月   1 09:21 com.gitee.jenkins.trigger.GiteePushTrigger.xml
-rw-r--r--   1 root root  1842 6月   1 09:21 config.xml
-rw-r--r--   1 root root 10145 6月   1 09:21 copy_reference_file.log
-rw-r--r--   1 root root  1054 5月  31 17:36 credentials.xml
drwxr-xr-x   3 root root  4096 5月  31 17:40 fingerprints
-rw-r--r--   1 root root   156 6月   1 09:21 hudson.model.UpdateCenter.xml
-rw-r--r--   1 root root  1270 5月  31 11:26 hudson.plugins.emailext.ExtendedEmailPublisher.xml
-rw-r--r--   1 root root   376 5月  31 18:04 hudson.plugins.git.GitTool.xml
-rw-r--r--   1 root root   173 5月  31 18:04 hudson.plugins.gradle.Gradle.xml
-rw-r--r--   1 root root   159 5月  31 18:04 hudson.tasks.Ant.xml
-rw-r--r--   1 root root   327 5月  31 18:04 hudson.tasks.Maven.xml
-rw-------   1 root root  1712 5月  31 11:08 identity.key.enc
-rw-r--r--   1 root root     7 6月   1 09:21 jenkins.install.InstallUtil.lastExecVersion
-rw-r--r--   1 root root     7 5月  31 11:20 jenkins.install.UpgradeWizard.state
-rw-r--r--   1 root root   184 5月  31 11:20 jenkins.model.JenkinsLocationConfiguration.xml
-rw-r--r--   1 root root   247 5月  31 18:04 jenkins.mvn.GlobalMavenConfig.xml
-rw-r--r--   1 root root   357 5月  31 11:23 jenkins.security.apitoken.ApiTokenPropertyConfiguration.xml
-rw-r--r--   1 root root   169 5月  31 11:23 jenkins.security.QueueItemAuthenticatorConfiguration.xml
-rw-r--r--   1 root root   162 5月  31 11:23 jenkins.security.UpdateSiteWarningsConfiguration.xml
-rw-r--r--   1 root root   171 5月  31 11:08 jenkins.telemetry.Correlator.xml
drwxr-xr-x   3 root root  4096 5月  31 13:45 jobs
drwxr-xr-x   4 root root  4096 5月  31 13:51 logs
-rw-r--r--   1 root root   907 6月   1 09:21 nodeMonitors.xml
drwxr-xr-x   2 root root  4096 5月  31 11:08 nodes
-rw-r--r--   1 root root   256 5月  31 18:04 org.jenkinsci.plugins.gitclient.JGitApacheTool.xml
-rw-r--r--   1 root root   244 5月  31 18:04 org.jenkinsci.plugins.gitclient.JGitTool.xml
drwxr-xr-x 130 root root 24576 6月   1 10:08 plugins
-rw-r--r--   1 root root   130 6月   1 13:54 queue.xml
-rw-r--r--   1 root root   129 6月   1 09:21 queue.xml.bak
-rw-r--r--   1 root root    64 5月  31 11:08 secret.key
-rw-r--r--   1 root root     0 5月  31 11:08 secret.key.not-so-secret
drwx------   2 root root  4096 5月  31 17:41 secrets
drwxr-xr-x   2 root root  4096 5月  31 11:19 updates
drwxr-xr-x   2 root root  4096 5月  31 11:08 userContent
drwxr-xr-x   4 root root  4096 6月   1 09:36 users
drwxr-xr-x  11 root root  4096 5月  31 11:08 war
drwxr-xr-x   2 root root  4096 5月  31 11:08 workflow-libs
drwxr-xr-x   4 root root  4096 5月  31 17:40 workspace
```

这里该说不说的，就捡一些比较常用（其实所谓不常用的就是自己不懂的，哈哈）的来做一些说明。

## 1，config.xml

这个厉害了，初始时里边定义Jenkins的版本，用户等各种信息，此文件不要动，如果随意更改里边的东西，很有可能会使Jenkins web界面处受到创伤。等项目各种编辑之后，详细的用户信息，权限，以及标头，视图等等都写入到了这里。

## 2，credentials.xml

存储Git拉取的证书信息。

## 3，Fingerprints

其中定义了通过秘钥所拉取的项目记录。

## 4，Jobs

这个重要了，要详细说明一下。一句话说明这是项目的配置保存目录已经构建历史等信息存储目录。

首先进到目录中，随便进到一个项目里。看到一些文件。

```sh
drwxr-xr-x 7 root root  200 Apr 17 17:24 builds
-rw------- 1 root root 3146 Apr 17 17:24 config.xml
lrwxrwxrwx 1 root root   22 Apr 17 17:23 lastStable -> builds/lastStableBuild
lrwxrwxrwx 1 root root   26 Apr 17 17:23 lastSuccessful -> builds/lastSuccessfulBuild
-rw------- 1 root root    3 Apr 17 17:23 nextBuildNumber
```

![image](http://t.eryajf.net/imgs/2021/09/8fa8575ab1719875.jpg)  

其中，builds里边保存着构建历史记录，走，进去看一下    

![image](http://t.eryajf.net/imgs/2021/09/aef1d44fe09e68e1.jpg)  

前边一串数字是什么呢，很简单，再看张图就知道了。  

![image](http://t.eryajf.net/imgs/2021/09/5dfac79a3b47d309.jpg)  

就是这些，每个数字对应构建历史数，然后里边保存着那次构建的详细信息。那么为什么是5次呢，其配置信息在项目配置当中定义：

![image](http://t.eryajf.net/imgs/2021/09/ed921652ed2b9be9.jpg)

就是这里，可以根据实际情况对构建历史进行保留，不建议过大，因为每次构建所有的日志等都在这里保存着，时间长了容易憋坏！！！

那么回到上级，builds旁边的config.xml就是这个项目的配置文件信息。

## 5，Logs

这里边的东西没太多意义，不用管。

## 6，Nodes

是一些节点管理的信息，Jenkins配置主从之后会在这里记录。

## 7，Plugins

不用说，插件集中营。

## 8，Users

所有用户信息都在这里保存。

## 9，Workspace

所有代码存储目录，或者叫项目工作目录，一般情况下，使用Jenkins结合服务器脚本进行一下构建部署操作的时候，都会使用到这个目录，$WORKSPACE，就是表示对应的项目根目录，注意要大写。