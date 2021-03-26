## Java设计模式

### 单例模式

#### 基本概念

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

**注意：**

- 1、单例类只能有一个实例。
- 2、单例类必须自己创建自己的唯一实例。
- 3、单例类必须给所有其他对象提供这一实例。

#### 实现

我们将创建一个 `SingleObject` 类。`SingleObject` 类有它的私有构造函数和本身的一个静态实例。

`SingleObject*`类提供了一个静态方法，供外界获取它的静态实例。`SingletonPatternDemo*`类使用 `SingleObject` 类来获取 `SingleObject*`对象。

![](https://www.runoob.com/wp-content/uploads/2014/08/62576915-36E0-4B67-B078-704699CA980A.jpg)

##### 1、懒汉式，线程不安全

```java
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
  
    public static Singleton getInstance() {  
    if (instance == null) {  
        instance = new Singleton();  
    }  
    return instance;  
    }  
}
```

##### 2、懒汉式，线程安全

```java
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
    public static synchronized Singleton getInstance() {  
    if (instance == null) {  
        instance = new Singleton();  
    }  
    return instance;  
    }  
}
```

##### 3、饿汉式

```java
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
}
```

##### 4、双检锁/双重校验锁（DCL，即 double-checked locking）

```java
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```

##### 5、登记式/静态内部类

```java
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}
```

### 工厂模式

在工厂模式中，我们没有创建逻辑暴露给客户端创建对象，并使用一个通用的接口引用新创建的对象。

#### 实现

我们将创建一个Shape接口和实现Shape接口的具体类。 一个工厂类ShapeFactory会在下一步中定义。

使用ShapeFactory来获取一个Shape对象。 它会将信息（CIRCLE/RECTANGLE/SQUARE）传递给ShapeFactory以获取所需的对象类型。

​	实现工厂模式的结构如下图所示 -

​	![img](http://www.yiibai.com/uploads/images/201612/07/444221217_55898.jpg)

1. 代码

    ```java
    package com.king.simple.shape;
    
    public interface IShape {
        void draw();
    }
    
    package com.king.simple.shape.util;
    
    public interface Constant {
    
        String CIRCLE = "CIRCLE";
    
        String RECTANGLE = "RECTANGLE";
    
        String SQUARE = "SQUARE";
    }
    
    package com.king.simple.shape.impl;
    
    import com.king.simple.shape.IShape;
    
    public class CricleShape implements IShape {
        @Override
        public void draw() {
            System.out.println("CricleShape draw...");
        }
    }
    
    package com.king.simple.shape.impl;
    
    import com.king.simple.shape.IShape;
    
    public class RectangleShape implements IShape {
        @Override
        public void draw() {
            System.out.println("RectangleShape draw ...");
        }
    }
    
    package com.king.simple.shape.impl;
    
    import com.king.simple.shape.IShape;
    
    public class SquareShape implements IShape {
        @Override
        public void draw() {
            System.out.println("SquareShape draw ...");
        }
    }
    
    package com.king.simple;
    
    import com.king.simple.shape.IShape;
    import com.king.simple.shape.impl.CricleShape;
    import com.king.simple.shape.impl.RectangleShape;
    import com.king.simple.shape.impl.SquareShape;
    import com.king.simple.shape.util.Constant;
    
    public class ShapeFactory {
    
        public IShape getShape(String shapeType) {
            IShape shape = null;
            if (Constant.CIRCLE.equals(shapeType)) {
                shape = new CricleShape();
            } else if (Constant.RECTANGLE.equals(shapeType)) {
                shape = new RectangleShape();
            } else if (Constant.SQUARE.equals(shapeType)) {
                shape = new SquareShape();
            } else {
                // do nothing
            }
            return shape;
        }
    
    }
    
    
    ```

2. 测试

![image-20210325171208467](C:\Users\wys1557\AppData\Roaming\Typora\typora-user-images\image-20210325171208467.png)



### 抽象工厂

抽象工厂模式是一个超级工厂，用来创建其他工厂。 这个工厂也被称为工厂的工厂。 这种类型的设计模式属于创建模式，因为此模式提供了创建对象的最佳方法之一。

在抽象工厂模式中，接口负责创建相关对象的工厂，而不明确指定它们的类。 每个生成的工厂可以按照工厂模式提供对象

#### 简单实现

https://www.yiibai.com/design_pattern/abstract_factory_pattern.html

#### 代理实现

- ##### JDK代理

代码

````java
package com.king.abstractfactory.color;

public interface IColor {
    void fill();
}

package com.king.abstractfactory.color.impl;

import com.king.abstractfactory.color.IColor;

public class Blue implements IColor {
    @Override
    public void fill() {
        System.out.println("Blue fill...");
    }
}

package com.king.abstractfactory.color.impl;

import com.king.abstractfactory.color.IColor;

public class Green implements IColor {
    @Override
    public void fill() {
        System.out.println("Green fill...");
    }
}

package com.king.abstractfactory.proxy;

import com.king.abstractfactory.color.IColor;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class DynamicProxyHandler implements InvocationHandler {

    private IColor iColor;

    public DynamicProxyHandler(IColor iColor) {
        this.iColor = iColor;
    }


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("代理类：" + method.getClass().getName() + " 代理方法：" + method.getName());
        method.invoke(iColor, args);
        return null;
    }
}

package com.king.abstractfactory.proxy;

import com.king.abstractfactory.color.IColor;

import java.lang.reflect.Proxy;

public class JDKProxy {
    public static <T> Object getProxy(IColor color) {
        DynamicProxyHandler dynamicProxyHandler = new DynamicProxyHandler(color);
        return Proxy.newProxyInstance(color.getClass().getClassLoader(), color.getClass().getInterfaces(), dynamicProxyHandler);
    }
}
````

测试：

```java
@Test
public void testDynamic() {
    IColor buleProxy = (IColor) JDKProxy.getProxy(new Blue());
    buleProxy.fill();

    IColor greenProxy = (IColor) JDKProxy.getProxy(new Green());
    greenProxy.fill();

}
```

![image-20210325171722950](C:\Users\wys1557\AppData\Roaming\Typora\typora-user-images\image-20210325171722950.png)

- CGLIB代理

代码：

```java
package com.king.abstractfactory.proxy;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibProxyInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("代理类：" + method.getClass().getName() + " 代理方法：" + method.getName());
        return methodProxy.invokeSuper(o, objects);
    }
}
```



测试：

```java
@Test
public void testCglib() {

    // 创建Enhancer对象，用于生成代理类
    Enhancer enhancer = new Enhancer();
    // 设置哪个类需要代理
    enhancer.setSuperclass(Blue.class);
    // 设置怎么代理，这里传入的是Callback对象-MethodInterceptor父类
    CglibProxyInterceptor cglibProxyInterceptor = new CglibProxyInterceptor();
    enhancer.setCallback(cglibProxyInterceptor);
    // 获取代理类实例
    Blue tank = (Blue) enhancer.create();
    tank.fill();

}
```

![image-20210325171819560](C:\Users\wys1557\AppData\Roaming\Typora\typora-user-images\image-20210325171819560.png)

### 代理模式

代理模式给某一个对象提供一个代理对象，并由代理对象控制对原对象的引用(接口的引用)。静态代理是指，代理类在程序运行前就已经定义好，其与**`目标类(被代理类)`**的关系在程序运行前就已经确立。静态代理类似于企业与企业的法律顾问间的关系。法律顾问与企业的代理关系，并不是在“官司“发生后才建立的，而是之前就确立好的一种关系。而动态代理就是外面打官司一样，是官司发生了之后临时请的律师。

#### 基本概念

代理模式是对象的结构模式。

代理模式给某一个对象提供一个代理对象，并由代理对象控制对原对象的引用(接口的引用)

#### 静态代理

**案例:** 

比如我们有一个可以移动的坦克，它的主要方法是 `move() `，但是我们需要记录它移动的时间，以及在它移动前后做日志，其静态代理的实现模式就类似下面的图 :



![img](http://blog.itpub.net/ueditor/php/upload/image/20190611/1560259472971678.jpg)



两个代理类以及结构关系:



![img](http://blog.itpub.net/ueditor/php/upload/image/20190611/1560259472124202.jpg)



代码:

```java
package com.king.entity;

public interface Move {

    void move();

}

package com.king.entity.impl;

import com.king.entity.Move;
import java.util.Random;
import java.util.concurrent.TimeUnit;

public class Tank implements Move {
    @Override
    public void move() {
        System.out.println("Tank moving...");
        try {
            TimeUnit.SECONDS.sleep(new Random().nextInt(3));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

代理类: `TimeProxy `

```java
package com.king.normal.proxy;

import com.king.entity.Move;

public class TimeProxy implements Move {

    private Move target;

    public TimeProxy(Move tank) {
        this.target = tank;
    }

    @Override
    public void move() {
        long start = System.currentTimeMillis();
        this.target.move();
        // 在后面做一些事情: 记录结束时间,并计算move()运行时间
        long end = System.currentTimeMillis();
        System.out.println("spend all time : " + (end - start)/1000 + "s.");
    }
}
```

测试:

```java
@Test
public void testTimeProxy() {
    Move target = new TimeProxy(new Tank());
    target.move();
}
```

输出：

![image-20210324172444763](C:\Users\wys1557\AppData\Roaming\Typora\typora-user-images\image-20210324172444763.png)



这其中有两个很重要的点:

- 代理对象内部都有着被代理对象(target)实现的接口的引用；
- 代理对象都实现了被代理对象(target)实现的接口 -- 不需要也可以如实例代码

现在单看做时间这个代理，如果我们现在多了一个飞机，飞机里面的方法是 `fly() `，现在要给飞机做代理，那么我们不能用之前写的 `TimeProxy `，我们需要额外的写一个 `PlaneTimeProxy `，这明显是冗余代码，所以这就是静态代理最大的缺点，这可以用动态代理解决 。

#### 动态代理

动态代理是指， 程序在整个运行过程中根本就不存在目标类的代理类(在JDK内部叫 `$Proxy0 `，我们看不到) ，目标对象的代理对象只是由代理生成工具(如代理工厂类) 在程序运行时由 JVM 根据反射等机制动态生成的。代理对象与目标对象的代理关系在程序运行时才确立。

对比静态代理，静态代理是指在程序运行前就已经定义好了目标类的代理类。代理类与目标类的代理关系在程序运行之前就确立了。

首先看动态代理的一些特点:

- 动态代理不需要写出代理类的名字，你要的代理对象我直接给你产生，是使用的时候生成的；
- 只需要调用 `Proxy.newProxyInstance() `就可以给你产生代理类；



**实例：**

```java
package com.king.entity;

public interface Move {

    void move();
}

package com.king.entity;

public interface Fly {
    void fly();
}

package com.king.entity.impl;


import com.king.entity.Move;
import java.util.Random;
import java.util.concurrent.TimeUnit;

public class Tank implements Move {
    @Override
    public void move() {
        System.out.println("Tank moving...");
        try {
            TimeUnit.SECONDS.sleep(new Random().nextInt(3));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

package com.king.entity.impl;

import com.king.entity.Fly;

import java.util.Random;
import java.util.concurrent.TimeUnit;

public class Plane implements Fly {
    @Override
    public void fly() {
        System.out.println("Plane flying...");
        try {
            TimeUnit.SECONDS.sleep(new Random().nextInt(3));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```



动态代理类：

```java
package com.king.dynamic;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class TimeProxyInvocationHandler implements InvocationHandler {

    private Object target;

    public TimeProxyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 在前面做一些事情: 记录开始时间
        long start = System.currentTimeMillis();
        System.out.println("start time : " + start);
        method.invoke(target, args); // 调用目标方法  invoke是调用的意思, 可以有返回值的方法(我们这里move和fly都没有返回值)
        // 在后面做一些事情: 记录结束时间,并计算move()运行时间
        long end = System.currentTimeMillis();
        System.out.println("end time : " + end);
        System.out.println("spend all time : " + (end - start)/1000 + "s.");
        return null;
    }
}
```

测试：

```java
@Test
public void testDynamicProxy() {

    Move tank = new Tank();

    TimeProxyInvocationHandler timeProxyInvocationHandler = new TimeProxyInvocationHandler(tank);
    Move tankProxy = (Move) Proxy.newProxyInstance(tank.getClass().getClassLoader(),
            tank.getClass().getInterfaces(), timeProxyInvocationHandler);

    tankProxy.move();

    Fly fly = new Plane();
    TimeProxyInvocationHandler timeProxyInvocationHandler1 = new TimeProxyInvocationHandler(fly);
    Fly flyProxy = (Fly) Proxy.newProxyInstance(fly.getClass().getClassLoader(),
            fly.getClass().getInterfaces(), timeProxyInvocationHandler1);

    flyProxy.fly();

}
```



JDK静态代理是通过直接编码创建的，而JDK动态代理是利用反射机制在运行时创建代理类的。动态代理是jdk的技术，在java的java.lang.reflect包下提供了一个`Proxy`类和一个`InvocationHandler接`口，通过这个类和这个接口可以生成JDK动态代理类和动态代理对象。

**`InvocationHandler`** 用于表示在执行某个方法之前，之后你想加入什么（监控）代码。这个是真正在干活的类。
**`Proxy`** 用于自动生成代理类，InvocationHandler 将会作为Proxy的一个参数来生成代理类。但这个生成的细节在这里不深究。

**这两者是怎么挂上钩的呢？**

通过`Proxy.newInstance(`)方法，`invocationHandler`会作为参数传入Proxy， 由Proxy在编译的时候加工自动生成代理类，这个生成的代理类里就会有`invocationHandler`里指定的要执行的代码。

其实在动态代理中，核心是`InvocationHandler`。每一个代理的实例都会有一个关联的调用处理程序(InvocationHandler)。对待代理实例进行调用时，将对方法的调用进行编码并指派到它的调用处理器(`InvocationHandler`)的`invoke`方法。所以对代理对象实例方法的调用都是通过`InvocationHandler`中的`invoke`方法来完成的，而`invoke`方法会根据传入的代理对象、方法名称以及参数决定调用代理的哪个方法。



**源码分析**

```java
 public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        //如果h为空将抛出异常
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();//拷贝被代理类实现的一些接口，用于后面权限方面的一些检查
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            //在这里对某些安全权限进行检查，确保我们有权限对预期的被代理类进行代理
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * 下面这个方法将产生代理类
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * 使用指定的调用处理程序获取代理类的构造函数对象
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            //假如代理类的构造函数是private的，就使用反射来set accessible
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            //根据代理类的构造函数来生成代理类的对象并返回
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

所以代理类其实是通过**`getProxyClass`**方法来生成的：

```java
 /**
     * 生成一个代理类，但是在调用本方法之前必须进行权限检查
     */
    private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        //如果接口数量大于65535，抛出非法参数错误
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

       
        // 如果在缓存中有对应的代理类，那么直接返回
        // 否则代理类将有 ProxyClassFactory 来创建
        return proxyClassCache.get(loader, interfaces);
    }
```



那么**ProxyClassFactory**是什么呢？

```java
   /**
     *  里面有一个根据给定ClassLoader和Interface来创建代理类的工厂函数  
     *
     */
    private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        // 代理类的名字的前缀统一为“$Proxy”
        private static final String proxyClassNamePrefix = "$Proxy";

        // 每个代理类前缀后面都会跟着一个唯一的编号，如$Proxy0、$Proxy1、$Proxy2
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            for (Class<?> intf : interfaces) {
                /*
                 * 验证类加载器加载接口得到对象是否与由apply函数参数传入的对象相同
                 */
                Class<?> interfaceClass = null;
                try {
                    interfaceClass = Class.forName(intf.getName(), false, loader);
                } catch (ClassNotFoundException e) {
                }
                if (interfaceClass != intf) {
                    throw new IllegalArgumentException(
                        intf + " is not visible from class loader");
                }
                /*
                 * 验证这个Class对象是不是接口
                 */
                if (!interfaceClass.isInterface()) {
                    throw new IllegalArgumentException(
                        interfaceClass.getName() + " is not an interface");
                }
                /*
                 * 验证这个接口是否重复
                 */
                if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                    throw new IllegalArgumentException(
                        "repeated interface: " + interfaceClass.getName());
                }
            }

            String proxyPkg = null;     // 声明代理类所在的package
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

            /*
             * 记录一个非公共代理接口的包，以便在同一个包中定义代理类。同时验证所有非公共
             * 代理接口都在同一个包中
             */
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }

            if (proxyPkg == null) {
                // 如果全是公共代理接口，那么生成的代理类就在com.sun.proxy package下
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }

            /*
             * 为代理类生成一个name  package name + 前缀+唯一编号
             * 如 com.sun.proxy.$Proxy0.class
             */
            long num = nextUniqueNumber.getAndIncrement();
            String proxyName = proxyPkg + proxyClassNamePrefix + num;

            /*
             * 生成指定代理类的字节码文件
             */
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                /*
                 * A ClassFormatError here means that (barring bugs in the
                 * proxy class generation code) there was some other
                 * invalid aspect of the arguments supplied to the proxy
                 * class creation (such as virtual machine limitations
                 * exceeded).
                 */
                throw new IllegalArgumentException(e.toString());
            }
        }
    }
```



由上方代码`byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);`可以看到，其实生成代理类字节码文件的工作是通过 `ProxyGenerate`类中的`generateProxyClass`方法来完成的。

```java
 public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
        ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
       // 真正用来生成代理类字节码文件的方法在这里
        final byte[] var4 = var3.generateClassFile();
       // 保存代理类的字节码文件
        if(saveGeneratedFiles) {
            AccessController.doPrivileged(new PrivilegedAction() {
                public Void run() {
                    try {
                        int var1 = var0.lastIndexOf(46);
                        Path var2;
                        if(var1 > 0) {
                            Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar), 
                                                                                   new String[0]);
                            Files.createDirectories(var3, new FileAttribute[0]);
                            var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                        } else {
                            var2 = Paths.get(var0 + ".class", new String[0]);
                        }

                        Files.write(var2, var4, new OpenOption[0]);
                        return null;
                    } catch (IOException var4x) {
                        throw new InternalError("I/O exception saving generated file: " + var4x);
                    }
                }
            });
        }

        return var4;
    }
```

下面来看看真正用于生成代理类字节码文件的**`generateClassFile`**方法：

```java
private byte[] generateClassFile() {
        //下面一系列的addProxyMethod方法是将接口中的方法和Object中的方法添加到代理方法中(proxyMethod)
        this.addProxyMethod(hashCodeMethod, Object.class);
        this.addProxyMethod(equalsMethod, Object.class);
        this.addProxyMethod(toStringMethod, Object.class);
        Class[] var1 = this.interfaces;
        int var2 = var1.length;

        int var3;
        Class var4;
       //获得接口中所有方法并添加到代理方法中
        for(var3 = 0; var3 < var2; ++var3) {
            var4 = var1[var3];
            Method[] var5 = var4.getMethods();
            int var6 = var5.length;

            for(int var7 = 0; var7 < var6; ++var7) {
                Method var8 = var5[var7];
                this.addProxyMethod(var8, var4);
            }
        }

        Iterator var11 = this.proxyMethods.values().iterator();
        //验证具有相同方法签名的方法的返回类型是否一致
        List var12;
        while(var11.hasNext()) {
            var12 = (List)var11.next();
            checkReturnTypes(var12);
        }

        //后面一系列的步骤用于写代理类Class文件
        Iterator var15;
        try {
             //生成代理类的构造函数
            this.methods.add(this.generateConstructor());
            var11 = this.proxyMethods.values().iterator();

            while(var11.hasNext()) {
                var12 = (List)var11.next();
                var15 = var12.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.ProxyMethod var16 = (ProxyGenerator.ProxyMethod)var15.next();
                    //将代理类字段声明为Method，并且字段修饰符为 private static.
                   //因为 10 是 ACC_PRIVATE和ACC_STATIC的与运算 故代理类的字段都是 private static Method ***
                    this.fields.add(new ProxyGenerator.FieldInfo(var16.methodFieldName, 
                                   "Ljava/lang/reflect/Method;", 10));
                   //生成代理类的方法
                    this.methods.add(var16.generateMethod());
                }
            }
           //为代理类生成静态代码块对某些字段进行初始化
            this.methods.add(this.generateStaticInitializer());
        } catch (IOException var10) {
            throw new InternalError("unexpected I/O Exception", var10);
        }

        if(this.methods.size() > '\uffff') { //代理类中的方法数量超过65535就抛异常
            throw new IllegalArgumentException("method limit exceeded");
        } else if(this.fields.size() > '\uffff') {// 代理类中字段数量超过65535也抛异常
            throw new IllegalArgumentException("field limit exceeded");
        } else {
            // 后面是对文件进行处理的过程
            this.cp.getClass(dotToSlash(this.className));
            this.cp.getClass("java/lang/reflect/Proxy");
            var1 = this.interfaces;
            var2 = var1.length;

            for(var3 = 0; var3 < var2; ++var3) {
                var4 = var1[var3];
                this.cp.getClass(dotToSlash(var4.getName()));
            }

            this.cp.setReadOnly();
            ByteArrayOutputStream var13 = new ByteArrayOutputStream();
            DataOutputStream var14 = new DataOutputStream(var13);

            try {
                var14.writeInt(-889275714);
                var14.writeShort(0);
                var14.writeShort(49);
                this.cp.write(var14);
                var14.writeShort(this.accessFlags);
                var14.writeShort(this.cp.getClass(dotToSlash(this.className)));
                var14.writeShort(this.cp.getClass("java/lang/reflect/Proxy"));
                var14.writeShort(this.interfaces.length);
                Class[] var17 = this.interfaces;
                int var18 = var17.length;

                for(int var19 = 0; var19 < var18; ++var19) {
                    Class var22 = var17[var19];
                    var14.writeShort(this.cp.getClass(dotToSlash(var22.getName())));
                }

                var14.writeShort(this.fields.size());
                var15 = this.fields.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.FieldInfo var20 = (ProxyGenerator.FieldInfo)var15.next();
                    var20.write(var14);
                }

                var14.writeShort(this.methods.size());
                var15 = this.methods.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.MethodInfo var21 = (ProxyGenerator.MethodInfo)var15.next();
                    var21.write(var14);
                }

                var14.writeShort(0);
                return var13.toByteArray();
            } catch (IOException var9) {
                throw new InternalError("unexpected I/O Exception", var9);
            }
        }
    }
```

下面是将接口与Object中一些方法添加到代理类中的`addProxyMethod`方法：

```java
private void addProxyMethod(Method var1, Class<?> var2) {
        String var3 = var1.getName();//获得方法名称
        Class[] var4 = var1.getParameterTypes();//获得方法参数类型
        Class var5 = var1.getReturnType();//获得方法返回类型
        Class[] var6 = var1.getExceptionTypes();//异常类型
        String var7 = var3 + getParameterDescriptors(var4);//获得方法签名
        Object var8 = (List)this.proxyMethods.get(var7);//根据方法前面获得proxyMethod的value
        if(var8 != null) {//处理多个代理接口中方法重复的情况
            Iterator var9 = ((List)var8).iterator();

            while(var9.hasNext()) {
                ProxyGenerator.ProxyMethod var10 = (ProxyGenerator.ProxyMethod)var9.next();
                if(var5 == var10.returnType) {
                    ArrayList var11 = new ArrayList();
                    collectCompatibleTypes(var6, var10.exceptionTypes, var11);
                    collectCompatibleTypes(var10.exceptionTypes, var6, var11);
                    var10.exceptionTypes = new Class[var11.size()];
                    var10.exceptionTypes = (Class[])var11.toArray(var10.exceptionTypes);
                    return;
                }
            }
        } else {
            var8 = new ArrayList(3);
            this.proxyMethods.put(var7, var8);
        }

        ((List)var8).add(new ProxyGenerator.ProxyMethod(var3, var4, var5, var6, var2, null));
    }
```



#### CGLIB

cglib是一个开源的项目，使用cglib也可以实现动态代理。具体怎么实现，cglib的实现思路是，1：导入cglib依赖包，2：代理类实现MethodInterceptor，3：使用Enhance类得到动态代理的实例，并执行指定方法。



##### 引入依赖

```scheme
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>3.2.5</version>
</dependency>
```

##### 代理类

```java
package com.king.cglib;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class TimeInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {

        long start = System.currentTimeMillis();
        System.out.println("start time : " + start);

        Object object = methodProxy.invokeSuper(o, objects);

        long end = System.currentTimeMillis();
        System.out.println("end time : " + end);
        System.out.println("spend all time : " + (end - start)/1000 + "s.");

        return object;
    }
}
```

##### 测试类

```java
@Test
public void testCglib() {

    // 创建Enhancer对象，用于生成代理类
    Enhancer enhancer = new Enhancer();
    // 设置哪个类需要代理
    enhancer.setSuperclass(Tank.class);
    // 设置怎么代理，这里传入的是Callback对象-MethodInterceptor父类
    TimeInterceptor timeInterceptor = new TimeInterceptor();
    enhancer.setCallback(timeInterceptor);
    // 获取代理类实例
    Tank tank = (Tank)enhancer.create();
    tank.move();

}
```

#### 代理模式总结

| 代理方式      | 实现                                                         | 优点                                                         | 缺点                                                         | 特点                                                       |
| :------------ | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------------------------------------------------- |
| JDK静态代理   | 代理类与委托类实现同一接口，并且在代理类中需要硬编码接口     | 实现简单，容易理解                                           | 代理类需要硬编码接口，在实际应用中可能会导致重复编码，浪费存储空间并且效率很低 | 好像没啥特点                                               |
| JDK动态代理   | 代理类与委托类实现同一接口，主要是通过代理类实现InvocationHandler并重写invoke方法来进行动态代理的，在invoke方法中将对方法进行增强处理 | 不需要硬编码接口，代码复用率高                               | 只能够代理实现了接口的委托类                                 | 底层使用反射机制进行方法的调用                             |
| CGLIB动态代理 | 代理类将委托类作为自己的父类并为其中的非final委托方法创建两个方法，一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法；另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理 | 可以在运行时对类或者是接口进行增强操作，且委托类无需实现接口 | 不能对final类以及final方法进行代理                           | 底层将方法全部存入一个数组中，通过数组索引直接进行方法调用 |

