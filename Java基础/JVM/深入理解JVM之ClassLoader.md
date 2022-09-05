# 深入理解JVM之ClassLoader

Java在诞生之初便提出 "Write Once, Run Anywhere"，各提供商发布很多不同平台的虚拟机，这些虚拟机都可以载入并执行同平台无关的字节码。

设计者在第一版Java虚拟机规范中便承诺 "In the future, we will consider bounded extensions to the Java virtual machine to provide better support for other languages"，时至今日，商业机构和开源机构已在Java之外发展出一大批可以在JVM上运行的语言，如Clojure、Groovy、JRuby、Jpython、Scala、Kotlin等。这些语言都不是本地可执行程序，需要JVM将编译后的class文件加载后才能够运行，负责加载class文件的组件便是ClassLoader.

## JVM类加载流程

java语言系统内置了众多类加载器，从一定程度上讲，只存在两种不同的类加载器：一种是启动类加载器，此类加载由C++实现，是JVM的一部分；另一种就是所有其他的类加载器，这些类加载器均由java实现，且全部继承自`java.lang.ClassLoader`

- Bootstrap ClassLoader 启动类加载器，最顶层的加载类，由C++实现，负责加载%JAVA_HOME%/lib目录中或-Xbootclasspath中参数指定的路径中的，并且是虚拟机识别的（按名称）类库
- Extention ClassLoader 扩展类加载器，由启动类加载器加载，实现为`sun.misc.Launcher$ExtClassLoader`，负责加载目录%JRE_HOME%/lib/ext目录中或-Djava.ext.dirs中参数指定的路径中的jar包和class文件
- Application ClassLoader 应用类加载器，也称为系统类加载器(System ClassLoader，可由`java.lang.ClassLoader.getSystemClassLoader()`获取)，实现为`sun.misc.Launcher$AppClassLoader`，由启动类加载器加载，负责加载当前应用classpath下的所有类

## 双亲委派模型

java语言系统有众多类加载器，包括用户自定义类加载器，各加载器之间的加载顺序如何？首先从JVM入口应用`sun.misc.Launcher`聊起

### Launcher

```java
public Launcher() {
    ExtClassLoader localExtClassLoader;
    try {
        // 加载扩展类加载器
        localExtClassLoader = ExtClassLoader.getExtClassLoader();
    } catch (IOException localIOException1) {
        throw new InternalError("Could not create extension class loader", localIOException1);
    }
    try {
        // 加载应用类加载器
        this.loader = AppClassLoader.getAppClassLoader(localExtClassLoader);
    } catch (IOException localIOException2) {
        throw new InternalError("Could not create application class loader", localIOException2);
    }
    // 设置AppClassLoader为线程上下文类加载器
    Thread.currentThread().setContextClassLoader(this.loader);
    // ...
    
    static class ExtClassLoader extends java.net.URLClassLoader
    static class AppClassLoader extends java.net.URLClassLoader
}
```

`Launcher`初始化了`ExtClassLoader`和`AppClassLoader`，并将`AppClassLoader`设置为线程上下文类加载器，同时，初始化`AppClassLoader`时传入了`ExtClassLoader`实例，WHY? 这里要写一个大大的问号

`ExtClassLoader`和`AppClassLoader`都继承自`URLClassLoader`，而最终的父类则为`ClassLoader`。

![classloader](https://segmentfault.com/img/bV4ERs?w=651&h=521)

查看源码可以得知，初始化`AppClassLoader`时传入的`ExtClassLoader`实例最终设置为了`AppClassLoader`(ClassLoader)的parent属性，parent属性的作用是什么？

### 父类加载器

每个类都对应一个加载它的类加载器，我们可以通过程序来验证

```java
public class ClassLoaderDemo {
    public static void main(String[] args) {
        System.out.println("ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader());
        System.out.println("DNSNameService's ClassLoader is " + DNSNameService.class.getClassLoader());
        System.out.println("String's ClassLoader is " + String.class.getClassLoader());
    }
}
```

输出为

```plaintext
ClassLodarDemo's ClassLoader is sun.misc.Launcher$AppClassLoader@135fbaa4
DNSNameService's ClassLoader is sun.misc.Launcher$ExtClassLoader@6e0be858
String's ClassLoader is null
```

ClassLodarDemo为我们自己创建的类，其类加载器为AppClassLoader
DNSNameService为%JRE_HOME%/lib/ext目录下的类，其类加载器为ExtClassLoader
String存在于rt.jar中，但其类加载器为null，这里是应为rt.jar由Bootstrap ClassLoader加载，而Bootstrap ClassLoader是由C++编写，属于JVM的一部分

每个类加载器，都有一个父类加载器(parent)，同样通过程序验证

```java
public class ClassLoaderDemo {
    public static void main(String[] args) {
        System.out.println("ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader());
        System.out.println("The Parent of ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader().getParent());
        System.out.println("The GrandParent of ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader().getParent().getParent());
    }
}
```

输出为

```plaintext
ClassLodarDemo's ClassLoader is sun.misc.Launcher$AppClassLoader@135fbaa4
The Parent of ClassLodarDemo's ClassLoader is sun.misc.Launcher$ExtClassLoader@2503dbd3
The GrandParent of ClassLodarDemo's ClassLoader is null
```

`AppClassLoader`的父类加载器为`ExtClassLoader`
`ExtClassLoader`的父类加载器为null，null并不代表`ExtClassLoader`没有父类加载器，而是Bootstrap ClassLoader

### 双亲委派

![委派](https://segmentfault.com/img/bV4E1u?w=557&h=560)

类加载器在加载类或者其他资源时，使用的是如上图所示的双亲委派模型，这种模型要求除了顶层的BootStrap ClassLoader外，其余的类加载器都应当有自己的父类加载器（父类加载器不是父类继承），如果一个类加载器收到了类加载请求，首先会把这个请求委派给父类加载器加载，只有父类加载器无法完成类加载请求时，子类加载器才会尝试自己去加载。

要理解双亲委派，可以查看ClassLoader.loadClass方法

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // 检查是否已经加载过
        Class<?> c = findLoadedClass(name);
        if (c == null) { // 没有被加载过
            long t0 = System.nanoTime();
            // 首先委派给父类加载器加载
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // 如果父类加载器无法加载，才尝试加载
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

### 破坏双亲委派模型

双亲委派模型并不是一个强制性的约束模型，在java的世界中大部分的类加载器都遵循这个模型，但也有例外，一个典型的例子便是JNDI服务。

JNDI存放于rt.jar，由启动类加载器加载，JNDI的目的是对资源进行集中管理和查找，它需要调用由各厂商实现并部署在应用程序ClassPath下的JNDI接口实现的代码，如此一来，在双亲委派模型下，启动类加载器根本无法加载这些代码

针对此问题，java设计团队引入了线程上下文类加载器（Thread Context ClassLoader），这个类加载器可以通过Thread类的setContextClassLoader进行设置，默认继承父线程类加载器
通过线程上下文类加载器，JNDI服务通过此类加载器，由父类加载器请求子类加载器完成类加载动作

破坏双亲委派模型的例子还有很多，如tomcat服务、osgi、jigsaw等等，是否破坏双亲委派模型并没有对与错，只是不同场景下的具体应用而已

## 自定义ClassLoader

不论是AppClassLoader还是ExtClassLoader还是启动类加载器，其加载类的路径都是固定的，如果我们需要加载外部类或者资源，如某路径下或网络上，这样便需要自定义类加载器
自定义类加载器，只需要继承ClassLoader类，复写findClass方法，在findClass方法中调用defineClass方法即可

> 一个ClassLoader创建时如果没有指定parent，那么它的parent默认就是AppClassLoader
> 如果需要制定一个ClassLoader的父类加载器为启动类加载器，只需要将其parent指定为null即可

首先，编写一个测试用的类文件

```java
public class BeLoadedClass {
    public void say() {
        System.out.println("I'm Loaded by " + this.getClass().getClassLoader());
    }
}
```

将其编译，放入/data/classloader目录下

接下来，编写DiskClassLoader

```java
public class DiskClassLoader extends URLClassLoader {

    public DiskClassLoader(URL path) throws MalformedURLException {
        super(new URL[]{path});
    }

    public DiskClassLoader(URL path, ClassLoader parent) throws MalformedURLException {
        super(new URL[]{path}, parent);
    }
}
```

这里直接继承URLClassLoader类，该类findClass的实现如下

```java
protected Class<?> findClass(final String name) throws ClassNotFoundException {
    final Class<?> result;
    try {
        result = AccessController.doPrivileged(
            new PrivilegedExceptionAction<Class<?>>() {
                public Class<?> run() throws ClassNotFoundException {
                    // 类文件全路径
                    String path = name.replace('.', '/').concat(".class");
                    // 指定资源目录下查找
                    Resource res = ucp.getResource(path, false);
                    if (res != null) {
                        try {
                            // 调用defineClass生成类
                            return defineClass(name, res);
                        } catch (IOException e) {
                            throw new ClassNotFoundException(name, e);
                        }
                    } else {
                        return null;
                    }
                }
            }, acc);
    } catch (java.security.PrivilegedActionException pae) {
        throw (ClassNotFoundException) pae.getException();
    }
    if (result == null) {
        throw new ClassNotFoundException(name);
    }
    return result;
}
```

编写测试程序

```java
public class ClassLoaderDemo {
    public static void main(String[] args) throws Exception {
        URL path = new File("/data/classloader").toURI().toURL();

        DiskClassLoader diskClassLoaderA = new DiskClassLoader(path);
        Class<?> clazzA = diskClassLoaderA.loadClass("BeLoadedClass");
        Method sayA = clazzA.getMethod("say");
        Object instanceA = clazzA.newInstance();
        sayA.invoke(instanceA);
        System.out.println(diskClassLoaderA);
        System.out.println("clazzA@" + clazzA.hashCode());

        System.out.println("====");

        DiskClassLoader diskClassLoaderB = new DiskClassLoader(path, diskClassLoaderA);
        Class<?> clazzB = diskClassLoaderB.loadClass("BeLoadedClass");
        Method sayB = clazzB.getMethod("say");
        Object instanceB = clazzA.newInstance();
        sayB.invoke(instanceB);
        System.out.println(diskClassLoaderB);
        System.out.println("clazzB@" + clazzB.hashCode());

        System.out.println("====");

        DiskClassLoader diskClassLoaderC = new DiskClassLoader(path);
        Class<?> clazzC = diskClassLoaderC.loadClass("BeLoadedClass");
        Method sayC = clazzC.getMethod("say");
        Object instanceC = clazzC.newInstance();
        sayC.invoke(instanceC);
        System.out.println(diskClassLoaderC);
        System.out.println("clazzC@" + clazzC.hashCode());

        System.out.println("====");

        System.out.println("clazzA == clazzB " + (clazzA == clazzB));
        System.out.println("clazzC == clazzB " + (clazzC == clazzB));
    }
}
```

输出为

```plaintext
I'm Loaded by com.manerfan.jvm.oom.DiskClassLoader@4b67cf4d
com.manerfan.jvm.DiskClassLoader@4b67cf4d
clazzA@312714112
====
I'm Loaded by com.manerfan.jvm.oom.DiskClassLoader@4b67cf4d
com.manerfan.jvm.DiskClassLoader@29453f44
clazzB@312714112
====
I'm Loaded by com.manerfan.jvm.oom.DiskClassLoader@5cad8086
com.manerfan.jvm.DiskClassLoader@5cad8086
clazzC@1639705018
====
clazzA == clazzB true
clazzC == clazzB false
```

这里我们定义了3个ClassLoader，类搜索路径均为/data/classloader，其中diskClassLoaderB的父类加载器为diskClassLoaderA
clazzA clazzB clazzC 分别由 diskClassLoaderA diskClassLoaderB diskClassLoaderC 加载
instanceA instanceB instanceA 分别由 clazzA clazzB clazzC 创建

从输出可以看出，instanceA及instanceB所对应的class**`均由diskClassLoaderA加载`**
虽然diskClassLoaderA与diskClassLoaderB为两个不同的类加载器，但由于diskClassLoaderB的父类加载器为diskClassLoaderA，从输出结果可以看出，**clazzA与clazzB完全相同（包括内存地址）**，这也说明**`clazzA与clazzB均由同一类加载器加载`**
而instanceC所对应的class，classC却是由diskClassLoaderC加载

这个例子，能更好的帮助理解双亲委派模型

### 类的唯一性

对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间

如上例，虽然classA(classB)与classC的类路径相同，但由于被不同的类加载器加载，其却属于两个不同的类名称空间



摘自：[【修炼内功】[JVM\] 深入理解JVM之ClassLoader](https://segmentfault.com/a/1190000013469223)