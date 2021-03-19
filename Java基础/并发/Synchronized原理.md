## Synchronized详解

> synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性。
>
> synchronized用来锁定共享资源，使得同一时刻，只有一个线程可以访问和修改它，修改完毕后，其他线程才可以使用。这种方式叫做互斥锁。

### synchronized的三种应用方式

synchronized关键字最主要有以下3种应用方式，下面分别介绍

- 修饰实例方法，作用于当前实例加锁，进入同步代码钱要获得当前实例的锁
- 修饰静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
- 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码块前要获得给定对象的锁

#### synchronized作用于实例方法

我们设置类变量`static`为共享资源， 然后多个线程去修改。修改的含义是： 先读取，计算，再写入。那么这个过程就不是原子的，多个线程操作就会出现共享资源争抢问题。

我们在实例方法上添加synchronized，那么，同一个实例执行本方法时，抢到锁到可以执行。

1. 同一个实例

```java
public class FunctionSync implements Runnable {

    private static int i = 0;

    /**
     * synchronized 修饰实例方法
     */
    public synchronized void increase(){
        i++;
    }

    @Override
    public void run() {
        for(int j=0;j<100000;j++){
            increase();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        testOneInstance();
    }

    private static void testOneInstance() throws InterruptedException {
        FunctionSync functionSync = new FunctionSync();
        Thread thread1 = new Thread(functionSync);
        Thread thread2 = new Thread(functionSync);

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println(i);
    }
}
```

​	测试结果：

![image-20210318141450643](C:\Users\wys1557\AppData\Roaming\Typora\typora-user-images\image-20210318141450643.png)



2. 不同实例

```java
public class FunctionSync implements Runnable {

    private static int i = 0;

    /**
     * synchronized 修饰实例方法
     */
    public synchronized void increase(){
        i++;
    }

    @Override
    public void run() {
        for(int j=0;j<100000;j++){
            increase();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        testTwoInstance();
    }

    private static void testTwoInstance() throws InterruptedException {
        FunctionSync functionSync1 = new FunctionSync();
        FunctionSync functionSync2 = new FunctionSync();
        Thread thread1 = new Thread(functionSync1);
        Thread thread2 = new Thread(functionSync2);

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println(i);
    }
}
```

运行结果：

![image-20210318141734393](C:\Users\wys1557\AppData\Roaming\Typora\typora-user-images\image-20210318141734393.png)·总结：

**`总结`**

`synchronized`修饰的是类方法，锁的是实例，当多个线程操作不同实例时，会使用不同实例的锁，就无法保证修改static变量的有序性了。



#### synchronized作用于静态方法

synchronized作用于静态方法时，锁就是当前类到class对象锁。由于静态成员变量不专属于任何一个实例对象，是类成员，因此通过class对象锁可以控制静态成员的并发操作。

```java
public class FunctionSync implements Runnable {

    private static int i = 0;

    /**
     * synchronized 修饰实例方法
     */
    public static synchronized void increase(){
        i++;
    }

    @Override
    public void run() {
        for(int j=0;j<100000;j++){
            increase();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        testTwoInstance();


    }

    private static void testOneInstance() throws InterruptedException {
        FunctionSync functionSync = new FunctionSync();
        Thread thread1 = new Thread(functionSync);
        Thread thread2 = new Thread(functionSync);

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println(i);
    }
}
```



#### synchronized同步代码块

```java
public class CodeBlockSync implements Runnable {

    private static int i = 0;

    /**
     * synchronized 修饰实例方法
     */
    public void increase(){
        i++;
    }

    @Override
    public void run() {
        synchronized (CodeBlockSync.class) {
            for(int j=0;j<100000;j++){
                increase();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        testTwoInstance();
    }

    private static void testTwoInstance() throws InterruptedException {
        CodeBlockSync codeBlockSync1 = new CodeBlockSync();
        CodeBlockSync codeBlockSync2 = new CodeBlockSync();
        Thread thread1 = new Thread(codeBlockSync1);
        Thread thread2 = new Thread(codeBlockSync2);

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println(i);
    }

}
```

测试结果：

![image-20210318142052192](C:\Users\wys1557\AppData\Roaming\Typora\typora-user-images\image-20210318142052192.png)

### synchronized字节码解析

> 利用javap工具查看生成的class文件信息来分析Synchronize的实现

#### synchronized作用于实例方法

```shell
D:\workspace\basic-skill\concurrent\target\classes\com\king\sync>javap -c FunctionSync.class
Compiled from "FunctionSync.java"
public class com.king.sync.FunctionSync implements java.lang.Runnable {
  public com.king.sync.FunctionSync();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public synchronized void increase();
    Code:
       0: getstatic     #2                  // Field i:I
       3: iconst_1
       4: iadd
       5: putstatic     #2                  // Field i:I
       8: return

  public void run();
    Code:
       0: iconst_0
       1: istore_1
       2: iload_1
       3: ldc           #3                  // int 100000
       5: if_icmpge     18
       8: aload_0
       9: invokevirtual #4                  // Method increase:()V
      12: iinc          1, 1
      15: goto          2
      18: return

  public static void main(java.lang.String[]) throws java.lang.InterruptedException;
    Code:
       0: invokestatic  #5                  // Method testTwoInstance:()V
       3: return

  static {};
    Code:
       0: iconst_0
       1: putstatic     #2                  // Field i:I
       4: return
}
```

通过`invokevirtual`指令实现同步

#### synchronized作用于静态方法

```shell
D:\workspace\basic-skill\concurrent\target\classes\com\king\sync>javap -c FunctionSync.class
Compiled from "FunctionSync.java"
public class com.king.sync.FunctionSync implements java.lang.Runnable {
  public com.king.sync.FunctionSync();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static synchronized void increase();
    Code:
       0: getstatic     #2                  // Field i:I
       3: iconst_1
       4: iadd
       5: putstatic     #2                  // Field i:I
       8: return

  public void run();
    Code:
       0: iconst_0
       1: istore_1
       2: iload_1
       3: ldc           #3                  // int 100000
       5: if_icmpge     17
       8: invokestatic  #4                  // Method increase:()V
      11: iinc          1, 1
      14: goto          2
      17: return

  public static void main(java.lang.String[]) throws java.lang.InterruptedException;
    Code:
       0: invokestatic  #5                  // Method testTwoInstance:()V
       3: return

  static {};
    Code:
       0: iconst_0
       1: putstatic     #2                  // Field i:I
       4: return
}

```

通过`invokestatic`指令实现同步



#### synchronized作用于同步代码块

```sh
D:\workspace\basic-skill\concurrent\target\classes\com\king\sync>javap -c CodeBlockSync.class
Compiled from "CodeBlockSync.java"
public class com.king.sync.CodeBlockSync implements java.lang.Runnable {
  public com.king.sync.CodeBlockSync();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public void increase();
    Code:
       0: getstatic     #2                  // Field i:I
       3: iconst_1
       4: iadd
       5: putstatic     #2                  // Field i:I
       8: return

  public void run();
    Code:
       0: ldc           #3                  // class com/king/sync/CodeBlockSync
       2: dup
       3: astore_1
       4: monitorenter
       5: iconst_0
       6: istore_2
       7: iload_2
       8: ldc           #4                  // int 100000
      10: if_icmpge     23
      13: aload_0
      14: invokevirtual #5                  // Method increase:()V
      17: iinc          2, 1
      20: goto          7
      23: aload_1
      24: monitorexit
      25: goto          33
      28: astore_3
      29: aload_1
      30: monitorexit
      31: aload_3
      32: athrow
      33: return
    Exception table:
       from    to  target type
           5    25    28   any
          28    31    28   any

  public static void main(java.lang.String[]) throws java.lang.InterruptedException;
    Code:
       0: invokestatic  #6                  // Method testTwoInstance:()V
       3: return

  static {};
    Code:
       0: iconst_0
       1: putstatic     #2                  // Field i:I
       4: return
}
```

同步代码块是使用`monitorenter`和`monitorexit`指令实现的

### synchronized 原理



#### 对象头概念

在JVM中，对象在内存中到布局分为三块区域：对象头，实例数据和对齐填充。 如下：

![](https://img-blog.csdnimg.cn/20210205174726941.png)



##### 对象头

对象头分为两部分：Mark Word（标记字段） 与 Class Pointer(类型指针)。

| 虚拟机位数 | 头对象结构             | 说明                                                         |
| :--------- | :--------------------- | :----------------------------------------------------------- |
| 32/64bit   | Mark Word              | 存储对象的hashcode, 锁信息或分代年龄或GC标志等信息           |
| 32/64bit   | Class Metadata Address | 类型指针指向对象的类元数据， JVM通过这个指针确定该对象是哪个类的实例 |

1. Mark Word（标记字段）

    Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。Java对象头一般占有两个机器码（在32位虚拟机中，1个机器码等于4字节，也就是32bit），但是如果对象是数组类型，则需要三个机器码，因为JVM虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。

    其中Mark Word在默认情况下存储着对象的HashCode, 分代年龄，锁标记等， 以下是32位JVM的Mark Word默认存储结构。

    | 锁状态   | 25bit        | 4bit         | 1bit是否是偏向锁 | 2bit锁标志位 |
    | :------- | :----------- | :----------- | :--------------- | :----------: |
    | 无锁状态 | 对象HashCode | 对象分代年龄 | 0                |      01      |

    由于对象头的信息是与对象自身定义的数据没有关系到额外存储成本，因此考虑到JVM的空间效率，Mark Word被设计成为一个非固定的数据结构，以便存储更多有效的数据，它会根据对象本身的状态复用自己的存储空间，如32位JVM下，除了上述列出的Mark Word默认存储结构外，还有如下可能变化的结构：

2. Class Metadata Address

    类型指针指向对象的类元数据， JVM通过这个指针确定该对象是哪个类的实例

##### 实例数据

 存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐。

##### 对其填充

由于虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐。

#### Monitor

> Monitor 被翻译为监视器或管程

每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置指向 Monitor 对象的指针



Monitor 结构如下:

![](https://cdn.nlark.com/yuque/0/2020/png/117408/1597076885591-78093474-65f6-4073-905a-93bbaea69180.png?x-oss-process=image%2Fresize%2Cw_746)

- 刚开始 Monitor 中 Owner 为 null

-  当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一 个 Owner 

- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入 EntryList BLOCKED 

- Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争的时是非公平的 

- 图中 WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程，后面讲 wait-notify 时会分析

```tex
注意：
- synchronized 必须是进入同一个对象的 monitor 才有上述的效果 
- 不加 synchronized 的对象不会关联监视器，不遵从以上规则
```

### synchronized 中的锁

![](https://cdn.nlark.com/yuque/0/2020/png/117408/1597150756383-6c77d359-ce91-424b-bf1c-8a6b3fdd3dc8.png?x-oss-process=image%2Fresize%2Cw_743)

![](https://cdn.nlark.com/yuque/0/2020/png/117408/1598768382860-c813fa31-bf64-4129-a123-8fca8bd7b00e.png?x-oss-process=image%2Fresize%2Cw_746)

##### 1. 轻量级锁

轻量级锁的使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以 使用轻量级锁来优化。 

轻量级锁对使用者是透明的，即语法仍然是 **`synchronized`**

假设有两个方法同步块，利用同一个对象加锁

```java
static final Object obj = new Object(); 
public static void method1() { 
    synchronized( obj ) { 
        // 同步块 A 
        method2(); 
    } 
} 
public static void method2() { 
    synchronized( obj ) { 
        // 同步块 B 
    }
}
```

- 创建锁记录（Lock Record）对象，**每个线程都的栈帧**都会包含一个锁记录的结构，内部可以存储锁定对象的 Mark Word

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597078243527-f3f4bdc9-b1f3-44e9-8d22-21f421ffdfa7.png)

- 让锁记录中 Object reference 指向锁对象，并尝试用 cas 替换 Object 的 Mark Word，将 Mark Word 的值存 入锁记录

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597078382332-ea38ab2c-2b87-4261-8578-e7acd150c9ce.png)

- 如果 cas 替换成功，对象头中存储了 锁记录地址和状态 00 ，表示由该线程给对象加锁，这时图示如下

​														![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597078422914-8d125529-e8f9-4af2-a728-cbc14821ab88.png)



- 如果 cas 失败，有两种情况

- - 如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入**锁膨胀过程** 
    - 如果是自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597078512223-d24f338a-ca96-42d7-bb3a-c4e040eefb23.png)

- 当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重 入计数减一

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597078562468-05d3e4a6-bdb2-430d-9575-3fc7488b8bdc.png)

- 当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时使用 cas 将 Mark Word 的值恢复给对象 头

- - 成功，则解锁成功
    -  失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程



##### 2.锁膨胀

> 锁膨胀祸

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1598769373835-9084df02-be3b-4654-b252-ff719416becd.png?x-oss-process=image%2Fresize%2Cw_1500)





如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁。

```java
static Object obj = new Object(); 
public static void method1() {
synchronized( obj ) {

// 同步块

} }
```

- 当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597127189855-bc840219-d3e0-46d3-8ab9-a5af627d7203.png?x-oss-process=image%2Fresize%2Cw_1500)

- 这时 Thread-1 加轻量级锁失败，进入锁膨胀流程

- - 即为 Object 对象申请 Monitor 锁，让 Object 指向重量级锁地址 
    - 然后自己进入 Monitor 的 EntryList BLOCKE

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597147516793-88b100be-7e2e-48e0-9f94-79a83b680ee2.png?x-oss-process=image%2Fresize%2Cw_1500)

- 当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁 流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程

##### 3.自旋优化

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步 块，释放了锁），这时当前线程就可以**避免阻塞**。 

自旋重试成功的情况

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597148281436-dda61f61-840d-4715-9fdf-d6452497c1b5.png?x-oss-process=image%2Fresize%2Cw_1500)



自旋重试失败的情况

![image.png](https://cdn.nlark.com/yuque/0/2020/png/117408/1597148340921-e05c2069-eae2-4fdd-a93f-9eaa19d5b145.png?x-oss-process=image%2Fresize%2Cw_1500)



- 自旋会占用 CPU 时间，单核 CPU 自旋就是浪费，多核 CPU 自旋才能发挥优势。 
- 在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会 高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能。
-  Java 7 之后不能控制是否开启自旋功能

##### 4. 偏向锁

轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作。

 Java 6 中引入了偏向锁来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现 这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有 例如：

```java
static final Object obj = new Object(); 
public static void m1() {
    synchronized( obj ) { // 同步块 A
        m2(); 
    }
}
public static void m2() {
    synchronized( obj ) { // 同步块 B
        m3();
    }
}
public static void m3() {
    synchronized( obj ) {
        // 同步代码块c
    }
}
```

一个对象创建时:

- 如果开启了偏向锁(默认开启)，那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的thread、epoch、age 都为 0
- 偏向锁是默认是延迟的，不会在程序启动时立即生效，如果想避免延迟，可以加VM参数

​    **-****XX:BiasedLockingStartupDelay=0** 来禁用延迟

- 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、age 都为 0，第一次用到 hashcode 时才会赋值





























