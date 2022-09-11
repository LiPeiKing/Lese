# Springboot 之 Fat Jar

#### 前言

springboot FatJar 的设计,打破了标准jar的结构,在jar包内携带了其所依赖的jar包,通过jar中的main方法创建自己的类加载器,来识别加载运行其不规范的目录下的代码和依赖。

##### 初识jar包结构

解压Springboot项目之后，目录是这样的，只展开了2级目录(详细的请看附录):

```css
├── BOOT-INF
│   ├── classes
│   └── lib
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
└── org
    └── springframework
```

再看一个普通的jar，也就是非Spring Boot的jar是什么样子的

```css
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
└── com
    └── guofeng
```

Spring Boot的jar只比普通的jar多出了一个BOOT-INF目录，其他的东西在结构上是一样的。也就是说，首先Spring Boot的jar肯定是兼容普通的jar，普通jar有的东西，它都有，这样，它才能用`java -jar`运行。普通的jar，依赖都是在外部的，Spring Boot的fatjar，依赖都在Jar包内，还有就是把工程代码换了下位置。

我们知道在jar运行之前，会首先从MANIFEST.MF文件中，查找jar的相关信息。我们看看这个文件里的内容

```java
Manifest-Version: 1.0
Created-By: Maven Archiver 3.4.0
Build-Jdk-Spec: 12
Implementation-Title: demo
Implementation-Version: 0.0.1-SNAPSHOT
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.example.demo.DemoApplication
Spring-Boot-Version: 2.2.0.M6
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
```

上面有几个参数是需要我们关注的：

1. Main-Class
    这个是jar的入口函数main方法的类路径，这里可以看到，这个是spring的方法，不是我们工程中写的方法。
2. Start-Class
    这个参数是 Spring Boot定义的，可以看到，这个类是我们工程里的了。这个是我们工程的main方法的类路径。
3. Spring-Boot-Classes
    这个很明显，是工程代码在jar中的路径
4. Spring-Boot-Lib
    这个是工程中引入的依赖jar。Spring Boot把工程依赖的所有jar都打包在了项目的jar中，这就是为什么它是个fatjar。

先看看jar的入口函数org.springframework.boot.loader.JarLauncher。这个类在工程中是找不到的，因为是jar启动需要的，在打包阶段由maven打进去的，工程是不需要的。它在包spring-boot-loader中，GAV是:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-loader</artifactId>
    <version>2.0.2.RELEASE</version>
    <scope>provided</scope>
</dependency>
```

我们看下它的main方法:



```java
public static void main(String[] args) throws Exception{
        new JarLauncher().launch(args);
    }
```

非常的简单，它调用了`JarLauncher`的`launch`方法，并且把参数都传递进去了。但是在`JarLauncher`中并没有`launch`这个方法，这个方法是`JarLauncher`的祖父类`Launcher`的方法，继承关系是这样的`JarLauncher--->ExecutableArchiveLauncher--->Launcher`。
 我们看一下`launch`方法的实现:

```java
protected void launch(String[] args) throws Exception {
    JarFile.registerUrlProtocolHandler();
    ClassLoader classLoader = createClassLoader(getClassPathArchives());
    launch(args, getMainClass(), classLoader);
}
```

这是一个`protected`方法，只能在它的子类中使用。首先看第一句代码:`JarFile.registerUrlProtocolHandler();`，它的实现是:

```java
public static void registerUrlProtocolHandler() {
    //PROTOCOL_HANDLER = "java.protocol.handler.pkgs";
    String handlers = System.getProperty(PROTOCOL_HANDLER, "");
    //HANDLERS_PACKAGE = "org.springframework.boot.loader"
    System.setProperty(PROTOCOL_HANDLER, ("".equals(handlers) ? HANDLERS_PACKAGE : handlers + "|" + HANDLERS_PACKAGE));
    resetCachedUrlHandlers();
}
```

这几句代码非常简单，就是重新设置了一个属性值，叫做`java.protocol.handler.pkgs`。
 那么这个属性值是干什么用的呢，从属性名字上我们能大体猜测一下，跟protocol有关，也就是协议。什么是协议？http，https这是最常见的协议吧？对的，这个属性值是存放的一个路径，这个路径下面对应的是各种协议的处理逻辑。除了http和https，还有jar，file，ftp等等协议。这个属性值默认是`sun.net.www.protocol.jar.Handler`，这是Java的默认实现，大家可以去这个路径下看一下。

Spring Boot重新将这个路径设置了一下，添加了`org.springframework.boot.loader`路径，这个路径下有Spring Boot提供的Jar协议处理逻辑，也就是覆盖了原来的Jar处理逻辑。Spring Boot的Jar跟普通的Jar是有区别的，依赖是打包在Jar中的，引导类是Spring提供的实现，但是我们最后的目的肯定是启动我们自己的工程。所以在这个地方覆盖了默认的Jar启动逻辑，按照Spring的Jar启动逻辑来走，其目的就是自定义Jar包依赖的查找逻辑，从Jar内部找依赖，最终启动用户的工程。

我们再看第二句代码：

```undefined
ClassLoader classLoader = createClassLoader(getClassPathArchives());
```

简略一看，大家都能懂的，是创建了一个ClassLoader。我们仔细研究下`getClassPathArchives`方法，它的实现是：

```java
@Override
    protected List<Archive> getClassPathArchives() throws Exception {
        List<Archive> archives = new ArrayList<>(
                this.archive.getNestedArchives(this::isNestedArchive));
        postProcessClassPathArchives(archives);
        return archives;
    }
```

最主要就是方法的第一句代码，它是获取用户工程代码和依赖的。拆分一下，我们先看`isNestedArchive`方法，它在JarLauncher和WarLauncher中，一共两种实现，我们看下JarLauncher中的实现：

```java
@Override
    protected boolean isNestedArchive(Archive.Entry entry) {
        if (entry.isDirectory()) {
            return entry.getName().equals(BOOT_INF_CLASSES);
        }
        return entry.getName().startsWith(BOOT_INF_LIB);
    }
```

这个方法是一个过滤器，它把`BOOT-INF/classes/`和`BOOT-INF/lib/`中的类和jar过滤了出来。然后再回到`getClassPathArchives`方法中，在`getNestedArchives`方法中：

```java
@Override
    public List<Archive> getNestedArchives(EntryFilter filter) throws IOException {
        List<Archive> nestedArchives = new ArrayList<>();
        for (Entry entry : this) {
            if (filter.matches(entry)) {
                nestedArchives.add(getNestedArchive(entry));
            }
        }
        return Collections.unmodifiableList(nestedArchives);
    }
```

在这里，把所有符合过滤条件的文件，都放到了nestedArchives中，然后返回给调用者。
 现在，在代码`ClassLoader classLoader = createClassLoader(getClassPathArchives());`中，我们就好理解了，`getClassPathArchives`方法获取到了工程代码和工程的依赖jar，然后根据这些东西，创建了`ClassLoader`。

好的，还有最后一句代码:

```undefined
launch(args, getMainClass(), classLoader);
```

这个毫无疑问，就是最后的启动过程了。同样的，先分析参数`getMainClass`方法：

```java
@Override
    protected String getMainClass() throws Exception {
        Manifest manifest = this.archive.getManifest();
        String mainClass = null;
        if (manifest != null) {
            mainClass = manifest.getMainAttributes().getValue("Start-Class");
        }
        if (mainClass == null) {
            throw new IllegalStateException(
                    "No 'Start-Class' manifest entry specified in " + this);
        }
        return mainClass;
    }
```

首先代码上来是获取Manifest，这个文件对应的就是Jar包中的MANIFEST.MF文件，我们说过，这个文件中定义了Jar包的信息。然后，注意后面的代码，从文件中获取了`Start-Class`的值。我们在看这个文件的时候，`Start-Class`是定义了我们工程的启动类，对吧？终于，这个地方要运行我们自己的main方法了，现在我们拿到了用户的main方法的所在类。

然后看launch方法，是怎么启动的：

```java
protected void launch(String[] args, String mainClass, ClassLoader classLoader)
            throws Exception {
        Thread.currentThread().setContextClassLoader(classLoader);
        createMainMethodRunner(mainClass, args, classLoader).run();
    }
```

首先把我们刚刚创建的classLoader放到了当前线程中。这个classLoader中带着我们工程代码和工程依赖，没忘吧？OK。
 继续继续，敲黑板，重点来了，createMainMethodRunner方法的实现如下：

```java
public void run() throws Exception {
        Class<?> mainClass = Thread.currentThread().getContextClassLoader()
                .loadClass(this.mainClassName);
        Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
        mainMethod.invoke(null, new Object[] { this.args });
    }
```

通过我们创建的classLoader，去找`Start-Class`定义的class类文件。在这个class文件中，找main方法，然后调用这个main方法。
 咚咚咚咚，Jar启动流程到这里就结束了，后面就是Spring Boot应用的启动过程了。
 到这里，Spring Boot Jar启动流程就分析完了，是不是非常简洁和条理清晰，很巧妙的方法，佩服这些大佬。
 大家可以下载Spring Boot的源代码，然后再仔细的回味一下。

附录：

1. Spring Boot Jar解压之后的文件目录树

```ruby
├── BOOT-INF
│   ├── classes
│   │   ├── application.properties
│   │   └── com
│   │       └── example
│   │           └── demo
│   │               └── DemoApplication.class
│   └── lib
│       ├── classmate-1.5.0.jar
│       ├── hibernate-validator-6.0.17.Final.jar
│       ├── jackson-annotations-2.9.0.jar
│       ├── jackson-core-2.9.9.jar
│       ├── jackson-databind-2.9.9.3.jar
│       ├── jackson-datatype-jdk8-2.9.9.jar
│       ├── jackson-datatype-jsr310-2.9.9.jar
│       ├── jackson-module-parameter-names-2.9.9.jar
│       ├── jakarta.annotation-api-1.3.5.jar
│       ├── jakarta.validation-api-2.0.1.jar
│       ├── jboss-logging-3.4.1.Final.jar
│       ├── jul-to-slf4j-1.7.28.jar
│       ├── log4j-api-2.12.1.jar
│       ├── log4j-to-slf4j-2.12.1.jar
│       ├── logback-classic-1.2.3.jar
│       ├── logback-core-1.2.3.jar
│       ├── slf4j-api-1.7.28.jar
│       ├── snakeyaml-1.25.jar
│       ├── spring-aop-5.2.0.RC2.jar
│       ├── spring-beans-5.2.0.RC2.jar
│       ├── spring-boot-2.2.0.M6.jar
│       ├── spring-boot-autoconfigure-2.2.0.M6.jar
│       ├── spring-boot-starter-2.2.0.M6.jar
│       ├── spring-boot-starter-json-2.2.0.M6.jar
│       ├── spring-boot-starter-logging-2.2.0.M6.jar
│       ├── spring-boot-starter-tomcat-2.2.0.M6.jar
│       ├── spring-boot-starter-validation-2.2.0.M6.jar
│       ├── spring-boot-starter-web-2.2.0.M6.jar
│       ├── spring-context-5.2.0.RC2.jar
│       ├── spring-core-5.2.0.RC2.jar
│       ├── spring-expression-5.2.0.RC2.jar
│       ├── spring-jcl-5.2.0.RC2.jar
│       ├── spring-test-5.2.0.RC2.jar
│       ├── spring-web-5.2.0.RC2.jar
│       ├── spring-webmvc-5.2.0.RC2.jar
│       ├── tomcat-embed-core-9.0.24.jar
│       ├── tomcat-embed-el-9.0.24.jar
│       └── tomcat-embed-websocket-9.0.24.jar
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── com.example
│           └── demo
│               ├── pom.properties
│               └── pom.xml
└── org
    └── springframework
        └── boot
            └── loader
                ├── ExecutableArchiveLauncher.class
                ├── JarLauncher.class
                ├── LaunchedURLClassLoader$UseFastConnectionExceptionsEnumeration.class
                ├── LaunchedURLClassLoader.class
                ├── Launcher.class
                ├── MainMethodRunner.class
                ├── PropertiesLauncher$1.class
                ├── PropertiesLauncher$ArchiveEntryFilter.class
                ├── PropertiesLauncher$PrefixMatchingArchiveFilter.class
                ├── PropertiesLauncher.class
                ├── WarLauncher.class
                ├── archive
                │   ├── Archive$Entry.class
                │   ├── Archive$EntryFilter.class
                │   ├── Archive.class
                │   ├── ExplodedArchive$1.class
                │   ├── ExplodedArchive$FileEntry.class
                │   ├── ExplodedArchive$FileEntryIterator$EntryComparator.class
                │   ├── ExplodedArchive$FileEntryIterator.class
                │   ├── ExplodedArchive.class
                │   ├── JarFileArchive$EntryIterator.class
                │   ├── JarFileArchive$JarFileEntry.class
                │   └── JarFileArchive.class
                ├── data
                │   ├── RandomAccessData.class
                │   ├── RandomAccessDataFile$1.class
                │   ├── RandomAccessDataFile$DataInputStream.class
                │   ├── RandomAccessDataFile$FileAccess.class
                │   └── RandomAccessDataFile.class
                ├── jar
                │   ├── AsciiBytes.class
                │   ├── Bytes.class
                │   ├── CentralDirectoryEndRecord.class
                │   ├── CentralDirectoryFileHeader.class
                │   ├── CentralDirectoryParser.class
                │   ├── CentralDirectoryVisitor.class
                │   ├── FileHeader.class
                │   ├── Handler.class
                │   ├── JarEntry.class
                │   ├── JarEntryFilter.class
                │   ├── JarFile$1.class
                │   ├── JarFile$2.class
                │   ├── JarFile$JarFileType.class
                │   ├── JarFile.class
                │   ├── JarFileEntries$1.class
                │   ├── JarFileEntries$EntryIterator.class
                │   ├── JarFileEntries.class
                │   ├── JarURLConnection$1.class
                │   ├── JarURLConnection$2.class
                │   ├── JarURLConnection$CloseAction.class
                │   ├── JarURLConnection$JarEntryName.class
                │   ├── JarURLConnection.class
                │   ├── StringSequence.class
                │   └── ZipInflaterInputStream.class
                └── util
                    └── SystemPropertyUtils.class
```

1. 普通Jar解压之后的文件树：

```css
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── com.guofeng
│           └── first-spring-boot
│               ├── pom.properties
│               └── pom.xml
└── com
    └── guofeng
        └── think
            └── App.class
```































