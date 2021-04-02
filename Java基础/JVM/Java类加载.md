## 类加载机制

![img](https://upload-images.jianshu.io/upload_images/23383522-231d614951394d23.png?imageMogr2/auto-orient/strip|imageView2/2/w/682/format/webp)



### 1.类的生命周期

类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存。

Java类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using) 和 卸载(Unloading)七个阶段。其中准备、验证、解析3个部分统称为连接（Linking），如下图所示。

![img](https:////upload-images.jianshu.io/upload_images/23383522-50f0b8a56cf0b0d4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

注意：加载、验证、准备、初始化和卸载这5个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班地开始，而解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定（也称为**动态绑定**或晚期绑定）。



#### 1.1、什么是类加载

我们所开发的程序是如何被加载至jvm中运行的，将我们开发的java项目打包成jar包或者war包，实际是将.java文件编译成为.class字节码文件，然后通过类加载器加载至jvm中运行，供后续代码运行使用。

 如下图：

![img](https:////upload-images.jianshu.io/upload_images/26071699-c5799f6fc557483f.png?imageMogr2/auto-orient/strip|imageView2/2/w/483/format/webp)



#### 1.2 、类加载的过程

我们的代码经历了 以下五个阶段

加载 —>验证 —>准备 —>解析 —>初始化

##### （1）加载阶段

当我们在代码中使用到的类就会进行加载，也就是hello.class调用user.class的时候，就会将字节码文件加载至jvm内存中。

![img](https:////upload-images.jianshu.io/upload_images/26071699-ffe6ae657dab26b5.png?imageMogr2/auto-orient/strip|imageView2/2/w/417/format/webp)

类的加载过程主要完成三件事：

- **通过全类名获取定义此类的二进制字节流（获取.class文件的字节流）**
- **将字节流上面所代表的静态存储结构转换为方法区的运行时数据结构**
- **在内存中生成一个代表该类的Class对象，作为方法区这些数据的访问入口**

值得注意的是，加载阶段和连接阶段的部分内容是交叉进行的，加载阶段尚未结束，连接阶段可能就已经开始了。加载是类加载的一个阶段，注意不要混淆。



##### （2）验证阶段

进行验证我们开发的代码编译后的.class文件是否合法是否符合JVM规范，验证通过后才能交给JVM来运行

![img](https:////upload-images.jianshu.io/upload_images/26071699-53e486e850beba2b.png?imageMogr2/auto-orient/strip|imageView2/2/w/417/format/webp)

##### （3）准备阶段

**准备阶段是正式为类变量分配内存并设置类变量初始值的阶段**，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

- 这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在 Java 堆中。
- 这里所设置的初始值"通常情况"下是数据类型默认的零值（如0、0L、null、false等），比如我们定义了`public static int value=111` ，那么 value 变量在准备阶段的初始值就是 0 而不是111（初始化阶段才会赋值）。特殊情况：比如给 value 变量加上了 fianl 关键字`public static final int value=111` ，那么准备阶段 value 的值就被赋值为 111。

![img](https:////upload-images.jianshu.io/upload_images/26071699-a3c53b4c653daaf4.png?imageMogr2/auto-orient/strip|imageView2/2/w/417/format/webp)

##### （4）解析阶段

**虚拟机将常量池中的符号引用替换成直接引用的过程。符号引用就理解为一个标示，而在直接引用直接指向内存中的地址；**其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了**支持 Java 的动态绑定**。

![img](https:////upload-images.jianshu.io/upload_images/26071699-85d9f9337693ab8d.png?imageMogr2/auto-orient/strip|imageView2/2/w/417/format/webp)

##### （5）初始化阶段

也是核心阶段， 执行准备阶段的任务,会给类变量开辟内存空间和设置初始值，这段赋值代码就是在初始化阶段来执行的。**初始化阶段是执行类构造器 方法的过程**（实例化对象时候 会触发初始化 先初始化父类）。对于构造方法的调用，虚拟机会自己确保其在多线程环境中的安全性。因为构造方法是带锁线程安全，所以在多线程环境下进行类初始化的话可能会引起死锁，并且这种死锁很难被发现。

![img](https:////upload-images.jianshu.io/upload_images/26071699-f0f95e64739eb504.png?imageMogr2/auto-orient/strip|imageView2/2/w/417/format/webp)

**主动引用：**对于初始化阶段，虚拟机严格规范了有且只有5种情况下，必须对类进行初始化(只有主动去使用类才会初始化类)：

1）当遇到 new 、 getstatic、putstatic或invokestatic 这4条直接码指令时，比如 new 一个类，读取一个静态字段(未被 final 修饰)、或调用一个类的静态方法时。

- 当jvm执行new指令时会初始化类。即当程序创建一个类的实例对象。
- 当jvm执行getstatic指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。
- 当jvm执行putstatic指令时会初始化类。即程序给类的静态变量赋值。
- 当jvm执行invokestatic指令时会初始化类。即程序调用类的静态方法。

2）使用 `java.lang.reflect` 包的方法对类进行反射调用时如Class.forname("..."),newInstance()等等。 ，如果类没初始化，需要触发其初始化。

3）初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。

4）当虚拟机启动时，用户需要定义一个要执行的主类 (包含 main 方法的那个类)，虚拟机会先初始化这个类。

5）MethodHandle和VarHandle可以看作是轻量级的反射调用机制，而要想使用这2个调用， 就必须先使用findStaticVarHandle来初始化要调用的类。

**被动引用：**除此之外，所有引用类的方式都不会触发初始化，称为被动引用。被动引用的常见例子包括：

1）通过子类引用父类的静态字段，不会导致子类初始化。

```csharp
System.out.println(SubClass.value);  // value 字段在 SuperClass 中定义
```

2）通过数组定义来引用类，不会触发此类的初始化。该过程会对数组类进行初始化，数组类是一个由虚拟机自动生成的、直接继承自 Object 的子类，其中包含了数组的属性和方法。

```cpp
SuperClass[] sca = new SuperClass[10];
```

3）常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

```css
System.out.println(ConstClass.HELLOWORLD);
```



#### 1.3、 卸载

**卸载类即该类的Class对象被GC。**卸载类需要满足3个要求:

- 该类的所有的实例对象都已被GC，也就是说堆不存在该类的实例对象。
- 该类没有在其他任何地方被引用
- 该类的类加载器的实例已被GC

所以，**在JVM生命周期类，由jvm自带的类加载器加载的类是不会被卸载的。但是由我们自定义的类加载器加载的类是可能被卸载的。**

只要想通一点就好了，jdk自带的`BootstrapClassLoader,PlatformClassLoader,AppClassLoader`负责加载jdk提供的类，所以它们(类加载器的实例)肯定不会被回收。而我们自定义的类加载器的实例是可以被回收的，所以使用我们自定义加载器加载的类是可以被卸载掉的。

### 2.类与类加载器

#### 2.1类加载器分类

**启动类加载器（Bootstrap ClassLoader）：**最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者被 `-Xbootclasspath`参数指定的路径中的所有类。

**ExtensionClassLoader(扩展类加载器)** ：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。

**AppClassLoader(应用程序类加载器)** ：面向用户的加载器，负责加载当前应用classpath下的所有jar包和类。

**用户自定义类加载器：**通过继承 java.lang.ClassLoader类的方式实现。

#### 2.2双亲委派模型

##### （1）概述

应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。

下图展示了**类加载器之间的层次关系，称为双亲委派模型（Parents Delegation Model）**。该模型要求除了顶层的启动类加载器外，其它的类加载器都要有自己的【**父加载器**】。这里的父子关系一般通过组合关系（Composition）来实现，而不是继承关系（Inheritance）。

![img](https:////upload-images.jianshu.io/upload_images/23383522-532d5ab51da714b3.png?imageMogr2/auto-orient/strip|imageView2/2/w/828/format/webp)

双亲委派模型

##### （2）**工作流程**

一个类加载器收到了类加载的请求，它首先不会自己去加载这个类，**而是把这个请求委派给父类加载器去完成，每一层的类加载器都是如此**，这样所有的加载请求都会被传送到顶层的启动类加载器中，**只有当父加载无法完成加载请求（它的搜索范围中没找到所需的类）时，子加载器才会尝试去加载类。**

##### （3）**双亲委派模型好处**

首先，使得Java类随着它的类加载器一起具有一种带有**优先级的层次关系**，**保证基础类的统一，即核心API不被篡改**。其次，保证java程序更加稳定，**可以避免类的重复加载**（jvm区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类）

##### （4）**具体实现**

以下是抽象类 `java.lang.ClassLoader`的代码片段，其中的 `loadClass() 方法`运行过程如下：**先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出 `ClassNotFoundException`，此时尝试自己去加载。**

```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    private final ClassLoader parent;

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
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
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

#### 2.3 自定义类加载器

以下代码中的 `FileSystemClassLoader`是自定义类加载器，继承自 `java.lang.ClassLoader`，用于加载文件系统上的类。它**首先根据类的全名在文件系统上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 `defineClass() 方法`来把这些字节代码转换成`java.lang.Class 类`的实例。**

`java.lang.ClassLoader 的 loadClass()`实现了双亲委派模型的逻辑，自定义类加载器一般不去重写它，但是需要重写 `findClass()`方法。

```java
public class FileSystemClassLoader extends ClassLoader {

    private String rootDir;

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
    }
}
```



















