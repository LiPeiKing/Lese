## Java CAS 解析

### 概述

![img](https://user-gold-cdn.xitu.io/2018/2/2/1615562b45874ef1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在Java并发中，我们最初接触的应该就是`synchronized`关键字了，但是`synchronized`属于重量级锁，很多时候会引起性能问题，`volatile`也是个不错的选择，但是`volatile`不能保证原子性，只能在某些场合下使用。



像`synchronized`这种独占锁属于**悲观锁**，它是在假设一定会发生冲突的，那么加锁恰好有用，除此之外，还有**乐观锁**，乐观锁的含义就是假设没有发生冲突，那么我正好可以进行某项操作，如果要是发生冲突呢，那我就重试直到成功，乐观锁最常见的就是`CAS`。



### 原理解释

我们在读Concurrent包下的类的源码时，发现无论是**ReenterLock内部的AQS，还是各种Atomic开头的原子类**，内部都应用到了`CAS`，最常见的就是我们在并发编程时遇到的`i++`这种情况。传统的方法肯定是在方法上加上`synchronized`关键字:

```java
public class Test {

    public volatile int i;

    public synchronized void add() {
        i++;
    }
}
```

但是这种方法在性能上可能会差一点，我们还可以使用`AtomicInteger`，就可以保证`i`原子的`++`了。

```java
public class Test {

    public AtomicInteger i;

    public void add() {
        i.getAndIncrement();
    }
}
```

我们来看`getAndIncrement`的内部：

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

再深入到`getAndAddInt`():

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

这里我们见到`compareAndSwapInt`这个函数，它也是`CAS`缩写的由来。那么仔细分析下这个函数做了什么呢？

首先我们发现`compareAndSwapInt`前面的`this`，那么它属于哪个类呢，我们看上一步`getAndAddInt`，前面是`unsafe`。这里我们进入的`Unsafe`类。这里要对`Unsafe`类做个说明。结合`AtomicInteger`的定义来说：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;
    
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
    
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    
    private volatile int value;
    ...

```

在`AtomicInteger`数据定义的部分，我们可以看到，其实实际存储的值是放在`value`中的，除此之外我们还获取了`unsafe`实例，并且定义了`valueOffset`。再看到`static`块，懂类加载过程的都知道，`static`块的加载发生于类加载的时候，是最先初始化的，这时候我们调用`unsafe`的`objectFieldOffset`从`Atomic`类文件中获取`value`的偏移量，那么`valueOffset`其实就是记录`value`的偏移量的。

再回到上面一个函数`getAndAddInt`，我们看`var5`获取的是什么，通过调用`unsafe`的`getIntVolatile(var1, var2)`，这是个native方法，具体实现到JDK源码里去看了，其实就是获取`var1`中，`var2`偏移量处的值。`var1`就是`AtomicInteger`，`var2`就是我们前面提到的`valueOffset`,这样我们就从内存里获取到现在`valueOffset`处的值了。

现在重点来了，`compareAndSwapInt（var1, var2, var5, var5 + var4）`其实换成`compareAndSwapInt（obj, offset, expect, update）`比较清楚，意思就是如果`obj`内的`value`和`expect`相等，就证明没有其他线程改变过这个变量，那么就更新它为`update`，如果这一步的`CAS`没有成功，那就采用自旋的方式继续进行`CAS`操作，取出乍一看这也是两个步骤了啊，其实在`JNI`里是借助于一个`CPU`指令完成的。所以还是原子操作。

实例：



![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1598458244467-287a3e35-397a-4a0c-a9b4-ec180eb9817b.png)

- 其实 CAS 的底层是 lock cmpxchg 指令(X86 架构)，在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的原子性
- 在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子 的。

### CAS 的底层原理

最终实现：

cmpxchg = cas 修改变量的值

```assembly
lock cmpxchg 指令
```

硬件：

lock 指令在执行后面指令的时候锁定一个北桥信号（不采用锁总线的方式）



### 用户态与内核态

**内核态：**CPU可以访问内存的所有数据，包括外围设备，例如硬盘，网卡，CPU也可以将自己从一个程序切换到另一个程序。

**用户态：**只能受限的访问内存，且不允许访问外围设备，占用CPU的能力被剥夺，CPU资源可以被其他程序获取。



为什么要有用户态和内核态？

由于需要限制不同的程序之间的访问能力, 防止他们获取别的程序的内存数据, 或者获取外围设备的数据, 并发送到网络, CPU划分出两个权限等级 -- 用户态和内核态。

### CAS 的问题

1. **ABA问题**

CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。这就是CAS的ABA问题。 常见的解决思路是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么`A-B-A` 就会变成`1A-2B-3A`。 目前在JDK的atomic包里提供了一个类`AtomicStampedReference`来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

2. **循环时间长开销大**

上面我们说过如果CAS不成功，则会原地自旋，如果长时间自旋会给CPU带来非常大的执行开销。


























































