# ThreadLocal

## 前言

在日常的开发中，我们经常会遇到在当前运行线程中保存一些信息，并且各线程之间是隔离的，不会相互影响，不存在并发问题，通过这样的方式来实现请求调用链中方法之间参数传递的解耦，提升代码结构的稳定性等。Java ThreadLocal就是用于实现这一目标的。在学习之前我们先带着以下几个问题：

1. ThreadLocal 是什么?
2. ThreadLocal 怎么用?
3. ThreadLocal 和线程同步机制相比较？
4. ThreadLocal 是如何实现线程隔离的呢？
5. ThreadLocal 如何避免内存泄漏呢？
6. ThreadLocal 与 Thread、ThreadLocalMap 之间的关系？

> 以下分析均基于JDK1.8。

**ThreadLocal**，很多地方叫做线程本地变量，也有些地方叫做线程本地存储。

ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量，这样同时多个线程访问该变量并不会彼此相互影响，因此他们使用的都是自己从内存中拷贝过来的变量的副本，这样就不存在线程安全问题，也不会影响程序的执行性能。

> **注意**：虽然ThreadLocal能够解决上面说的问题，但是由于在每个线程中都创建了副本，所以要考虑它对资源的消耗，比如内存的占用会比不使用ThreadLocal要大。



## ThreadLocal 怎么用

通常使用静态的变量来维护ThreadLocal，如:



```java
static ThreadLocal<String> sThreadLocal = new ThreadLocal<String>
```

会自动在每一个线程上创建一个 T 的副本，副本之间彼此独立，互不影响，可以用 ThreadLocal 存储一些参数，以便在线程中多个方法中使用，用以代替方法传参的做法。

通过一个例子来了解 ThreadLocal：

```java
public class ThreadLocalDemo {

    /**
     * ThreadLocal变量，每个线程都有一个副本，互不干扰
     */
    public static final ThreadLocal<String> THREAD_LOCAL = new ThreadLocal<>();

    public static void main(String[] args) throws Exception {
        new ThreadLocalDemo().threadLocalTest();
    }

    public void threadLocalTest() throws Exception {
        // 主线程设置值
        THREAD_LOCAL.set("一角钱技术");
        String v = THREAD_LOCAL.get();
        System.out.println("Thread-0线程执行之前，" + Thread.currentThread().getName() + "线程取到的值：" + v);

        new Thread(new Runnable() {
            @Override
            public void run() {
                String v = THREAD_LOCAL.get();
                System.out.println(Thread.currentThread().getName() + "线程取到的值：" + v);
                // 设置 threadLocal
                THREAD_LOCAL.set("一角钱技术2020");
                v = THREAD_LOCAL.get();
                System.out.println("重新设置之后，" + Thread.currentThread().getName() + "线程取到的值为：" + v);
                System.out.println(Thread.currentThread().getName() + "线程执行结束");
            }
        }).start();
        // 等待所有线程执行结束
        Thread.sleep(3000L);
        v = THREAD_LOCAL.get();
        System.out.println("Thread-0线程执行之后，" + Thread.currentThread().getName() + "线程取到的值：" + v);
    }
}
```

首先通过 `static final` 定义了一个 `THREAD_LOCAL` 变量，其中 `static` 是为了确保全局只有一个保存 String 对象的 ThreadLocal 实例；`final` 确保 ThreadLocal 的实例不可更改，防止被意外改变，导致放入的值和取出来的不一致，另外还能防止 ThreadLocal 的内存泄漏。上面的例子是演示在不同的线程中获取它会得到不同的结果，运行结果如下：

```java
Thread-0线程执行之前，main线程取到的值：一角钱技术
Thread-0线程取到的值：null
重新设置之后，Thread-0线程取到的值为：一角钱技术2020
Thread-0线程执行结束
Thread-0线程执行之后，main线程取到的值：一角钱技术
```

- 首先在 `Thread-0` 线程执行之前，先给 `THREAD_LOCAL` 设置为 `一角钱技术`，然后可以取到这个值；

- 然后通过创建一个新的线程以后去取这个值，发现新线程取到的为 null，意味着这个变量在不同线程中取到的值是不同的，不同线程之间对于 ThreadLocal 会有对应的副本；

- 接着在线程 `Thread-0` 中执行对 `THREAD_LOCAL` 的修改，将值改为 `一角钱技术2020`，可以发现线程 `Thread-0` 获取的值变为了 `一角钱技术2020`，主线程依然会读取到属于它的副本数据 `一角钱技术`，这就是线程的封闭。

## ThreadLocal和线程同步机制相比较

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。

在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂的问题，程序设计和编写难度相对较大。

而ThreadLocal则从另一个角度来解决多线程的并发访问。ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

总的来说，对于多线程资源共享的问题，**同步机制**采用了“以时间换空间”的方式，而**ThreadLocal**采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。

## ThreadLocal源码解析

## 

![img](https:////upload-images.jianshu.io/upload_images/10170978-3bc3416c45254b93.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 成员变量



```java
// 当前 ThreadLocal 的 hashCode，由 nextHashCode() 计算而来
// 用于计算当前 ThreadLocal 在 ThreadLocalMap 中的索引位置
private final int threadLocalHashCode = nextHashCode();
// 哈希魔数，主要与斐波那契散列法以及黄金分割有关
private static final int HASH_INCREMENT = 0x61c88647;
// 返回计算出的下一个哈希值，其值为 i * HASH_INCREMENT，其中 i 代表调用次数
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
// 保证了在一台机器中每个 ThreadLocal 的 threadLocalHashCode 是唯一的
private static AtomicInteger nextHashCode = new AtomicInteger();
```

其中的 `HASH_INCREMENT` 也不是随便取的，它转化为十进制是 `1640531527`，`2654435769` 转换成 int 类型就是 `-1640531527`，`2654435769` 等于 `(√5-1)/2` 乘以 2 的 32 次方。`(√5-1)/2` 就是黄金分割数，近似为 `0.618`，也就是说 `0x61c88647` 理解为一个黄金分割数乘以 2 的 32 次方，它可以保证 nextHashCode 生成的哈希值，均匀的分布在 2 的幂次方上，且小于 2 的 32 次方。

下面用例子来证明下：



```java
private static final int HASH_INCREMENT = 0x61c88647;

public static void main(String[] args) throws Exception {
    int n = 5;
    int max = 2 << (n - 1);
    for (int i = 0; i < max; i++) {
        System.out.print(i * HASH_INCREMENT & (max - 1));
        System.out.print(" ");

    }
}
```

运行结果为：`0 7 14 21 28 3 10 17 24 31 6 13 20 27 2 9 16 23 30 5 12 19 26 1 8 15 22 29 4 11 18 25`

可以发现元素索引值完美的散列在数组当中，并没有出现冲突。

### 内部类ThreadLocalMap

ThreadLocalMap 是 ThreadLocal 的静态内部类，当一个线程有多个 ThreadLocal 时，需要一个容器来管理多个 ThreadLocal，ThreadLocalMap 的作用就是管理线程中多个 ThreadLocal。

ThreadLocalMap 其实就是一个简单的 Map 结构，底层是数组，有初始化大小，也有扩容阈值大小，数组的元素是 Entry。

ThreadLocalMap的数据结构是一个用数组表示的环，数组长度必须是2的次幂，同样通过hash方式确定节点在数组中的下标（hash值是ThreadLocal的递增变量，而不是hashcode值），对于hash冲突的情况，采用**线性探测法**，直接将元素防止对应下标后面的下一个空闲单元。

ThreadLocalMap的key采用的是弱引用WeakReference，因此在使用过程中还需要注意及时清理key已经被gc回收的节点，及时释放无效空间。

> 关于弱引用可以查看[《Java基础 ｜强引用、弱引用、软引用、虚引用》](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FuFQ07Mvh1-vLHf-K8QYNwg)

#### 成员属性



```java
// 初始容量，必须为 2 的幂
private static final int INITIAL_CAPACITY = 16;

// 存储 ThreadLocal 的键值对实体数组，长度必须为 2 的幂
private Entry[] table;

// ThreadLocalMap 元素数量
private int size = 0;

//扩容的阈值，默认是数组大小的三分之二
private int threshold; // Default to 0
```

#### Entry类

Entry是ThreadLocalMap的内部类，用来表示其中的节点，继承了弱引用WeadReference<ThreadLocalMap>类。



```java
// 键值对实体的存储结构
static class Entry extends WeakReference<ThreadLocal<?>> {
    // 当前线程关联的 value，这个 value 并没有用弱引用追踪
    Object value;
    /**
     * 构造键值对
     *
     * @param k k 作 key,作为 key 的 ThreadLocal 会被包装为一个弱引用
     * @param v v 作 value
     */     
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

**Entry 的 key 就是 ThreadLocal 的引用，value 是 ThreadLocal 的值**。同时，Entry也继承WeakReference，所以说Entry所对应key（ThreadLocal实例）的引用是一个弱引用。

> 弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

#### 构造方法

ThreadLocalMap 提供了两个构造方法：

1. **ThreadLocalMap#ThreadLocalMap(ThreadLocal<?>, Object)**



```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

- 根据第一个节点的key和value初始化map。
- 初始化数组，确定节点在数组的下标，初始化table[i]，设置size和threshold。
- 进行散列的hash值是ThreadLocal的threadLocalHashCode，递增生成。

1. **ThreadLocalMap#ThreadLocalMap(ThreadLocalMap)**



```java
private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];

    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}
```

初始化数组和threshold，遍历节点加入数组。

#### 擦除机制

ThreadLocalMap中内部类Entry，继承了WeakReference，其key值是弱引用类型，在没有强引用时会被gc回收，因此ThreadLocalMap要及时对这部分过期节点进行擦除。

1. **ThreadLocalMap#expungeStaleEntry(int)**



```java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

擦除staleSlot处的无效节点，同时扫描处于staleSlot + 1 – 下一个null节点之间的节点，对于过期节点进行擦除，有效节点rehash，判断是否需要修改位置。

1. **ThreadLocalMap#expungeStaleEntries()**



```java
private void expungeStaleEntries()   {
    Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        Entry e = tab[j];
        if (e != null && e.get() == null)
            expungeStaleEntry(j);
    }
}
```

全量扫描擦除，遍历数组中的所有节点，对于过期节点调用擦除方法expungeStaleEntry进行擦除。

1. **ThreadLocalMap#cleanSomeSlots(int i, int n)**



```java
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
```

启发式扫描擦除。从 i+1 开始扫描检查，如果连续log n个单元不需要擦除则结束方法，否则找到一个过期节点，重置计数，将n置为数组长度，重新开始新一轮的扫描。只有扫描过程中有一个过期节点，则认为擦除成功，返回true。

#### ThreadLocalMap#getEntry(ThreadLocal<?>)



```java
/**
 * 返回 key 关联的键值对实体
 *
 * @param key threadLocal
 * @return
 */
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    // 若 e 不为空，并且 e 的 ThreadLocal 的内存地址和 key 相同，直接返回
    if (e != null && e.get() == key) {
        return e;
    } else {
        // 碰撞查找，从 i 开始向后遍历找到键值对实体
        return getEntryAfterMiss(key, i, e);
    }
}
```

我们再来看一下getEntryAfterMiss方法：



```java
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

用于在查找节点时没有直接命中的情况下进行线性的碰撞查找，对照查找过程中的过期节点，进行擦除。

#### ThreadLocalMap#remove(ThreadLocal<?>)



```java
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

根据key值移除节点。找到节点后不是简单的将该节点置为null，还需要调用擦除方法，不然该节点后面的hash冲突节点会无法通过getEntry获取到。

#### ThreadLocalMap#set(ThreadLocal<?>, Object)

调用set() 时，会把当前 `threadLocal` 对象作为 key，想要保存的对象作为 value，存入 map。用于增加或覆盖节点，类似于Map接口的put方法。



```java
/**
 * 在 map 中存储键值对<key, value>
 *
 * @param key   threadLocal
 * @param value 要设置的 value 值
 */
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    // 计算 key 在数组中的下标
    int i = key.threadLocalHashCode & (len - 1);
    // 遍历一段连续的元素，以查找匹配的 ThreadLocal 对象
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        // 获取该哈希值处的ThreadLocal对象
        ThreadLocal<?> k = e.get();

        // 键值ThreadLocal匹配，直接更改map中的value
        if (k == key) {
            e.value = value;
            return;
        }

        // 若 key 是 null，说明 ThreadLocal 被清理了，直接替换掉
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    // 直到遇见了空槽也没找到匹配的ThreadLocal对象，那么在此空槽处安排ThreadLocal对象和缓存的value
    tab[i] = new Entry(key, value);
    int sz = ++size;
    // 进行启发式擦除，节点数量大于阈值。如果右节点擦除成功，节点数量不可能大于阈值
    if (!cleanSomeSlots(i, sz) && sz >= threshold) {
        // 扩容的过程也是对所有的 key 重新哈希的过程
        rehash();
    }
}
```

我们依次来看看调用的几个方法：

1. **ThreadLocalMap#replaceStaleEntry(ThreadLocal<?>, Object, int)**



```java
private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                int staleSlot) {
     Entry[] tab = table;
     int len = tab.length;
     Entry e;

     // Back up to check for prior stale entry in current run.
     // We clean out whole runs at a time to avoid continual
     // incremental rehashing due to garbage collector freeing
     // up refs in bunches (i.e., whenever the collector runs).
     int slotToExpunge = staleSlot;
     for (int i = prevIndex(staleSlot, len);
          (e = tab[i]) != null;
          i = prevIndex(i, len))
         if (e.get() == null)
             slotToExpunge = i;

     // Find either the key or trailing null slot of run, whichever
     // occurs first
     for (int i = nextIndex(staleSlot, len);
          (e = tab[i]) != null;
          i = nextIndex(i, len)) {
         ThreadLocal<?> k = e.get();

         // If we find key, then we need to swap it
         // with the stale entry to maintain hash table order.
         // The newly stale slot, or any other stale slot
         // encountered above it, can then be sent to expungeStaleEntry
         // to remove or rehash all of the other entries in run.
         if (k == key) {
             e.value = value;

             tab[i] = tab[staleSlot];
             tab[staleSlot] = e;

             // Start expunge at preceding stale entry if it exists
             if (slotToExpunge == staleSlot)
                 slotToExpunge = i;
             cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
             return;
         }

         // If we didn't find stale entry on backward scan, the
         // first stale entry seen while scanning for key is the
         // first still present in the run.
         if (k == null && slotToExpunge == staleSlot)
             slotToExpunge = i;
     }

     // If key not found, put new entry in stale slot
     tab[staleSlot].value = null;
     tab[staleSlot] = new Entry(key, value);

     // If there are any other stale entries in run, expunge them
     if (slotToExpunge != staleSlot)
         cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
 }
```

slotToExpunge 表示第一个过期节点

- 从staleSlot向前扫描，扫描到第一个为null的节点截止，如果中间有过期节点，记录扫描过程中遇到的最后一个过期节点的下标为 slotToExpunge；
- 从staleSlot向后扫描，扫描找到key值对应的节点或null节点截止：
  - 如果在 [从staleSlot向前扫描] 中没有找到过期节点，需要本次扫描中遇到的第一个过期节点的下标记录为 slotToExpunge ；
  - 如果找到来 key值对应的节点，覆盖后将该节点移到 staleSlot 处，并将该节点的原来的位置作为过期节点处理；
  - 如果没有找到节点，新建节点放置到 staleSlot 处。
- 如果在两次扫描中找到了过期节点，先对该节点进行擦除，并调用启发式扫描擦除。

总体来说，假如 i 下标处的节点是 staleSlot 节点左边离得最近的null节点，j 下标处的节点是 staleSlot 节点右边离得最近的null节点，并且key值对应的节点作为过期节点处理。

那么该方法的功能就两段：

- 将 key、value 组成节点放到 staleSlot 处；
- 如果在（i — j）的序列中扫描到了过期节点，那么擦除该节点，并从该节点后的第一个null节点开始启发式擦除。

> 之所以需要向前扫描，是为了避免在扫描过程中对有效节点的rehash后出现由过期节点导致的hash冲突。

1. **ThreadLocalMap#rehash()**



```java
private void rehash() {
    expungeStaleEntries();

    // Use lower threshold for doubling to avoid hysteresis
    if (size >= threshold - threshold / 4)
        resize();
}
```

启动全局扫描擦除，擦除后再次判断是否需要扩容。之所以叫做rehash，可以理解成在全局扫描中所有的有效节点都需要重新hash确定位置。可以看到，并不是节点数量大于阈值后就会触发扩容，只有全局扫描擦除后数量仍大于阈值的3/4（容量的1/2）才会进行扩容。

1. **ThreadLocalMap#resize()**



```java
/**
* 扩容，重新计算索引，标记垃圾值，方便 GC 回收
*/
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    // 新建一个数组，按照2倍长度扩容
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;
    // 将旧数组的值拷贝到新数组上
    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            // 若有垃圾值，则标记清理该元素的引用，以便GC回收
            if (k == null) {
                e.value = null; // Help the GC
            } else {
                // 计算 ThreadLocal 在新数组中的位置
                int h = k.threadLocalHashCode & (newLen - 1);
                如果发生冲突，使用线性探测往后寻找合适的位置
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                    newTab[h] = e;
                    count++;
                }
            }
    }
    // 设置新的扩容阀值，为数组成都的三分之二
    setThreshold(newLen);
    size = count;
    table = newTab;
}
```

建立新数组，容量为原来的2倍，遍历数组中的元素，将有效节点hash后放入新数组，设置threshold，size等属性。

### ThreadLocal的 remove 方法

remove 方法源码如下所示：



```java
/**
 * 清理当前 ThreadLocal 对象关联的键值对
 */
public void remove() {
    // 返回当前线程持有的 map
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null) {
        // 从 map 中清理当前 ThreadLocal 对象关联的键值对
        m.remove(this);
    }
}
```

remove 方法的时序图如下所示：



![img](https:////upload-images.jianshu.io/upload_images/10170978-b3f394809d5150ae.png?imageMogr2/auto-orient/strip|imageView2/2/w/994/format/webp)

remove 方法是先获取到当前线程的 ThreadLocalMap，并且调用了它的 remove 方法，从 map 中清理当前 ThreadLocal 对象关联的键值对，这样 value 就可以被 GC 回收了。

### ThreadLocal的 set 方法

set 方法源码如下：



```java
/**
 * 为当前 ThreadLocal 对象关联 value 值
 *
 * @param value 要存储在此线程的线程副本的值
 */
public void set(T value) {
    // 返回当前ThreadLocal所在的线程
    Thread t = Thread.currentThread();
    // 返回当前线程持有的map
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // 如果 ThreadLocalMap 不为空，则直接存储<ThreadLocal, T>键值对
        map.set(this, value);
    } else {
        // 否则，需要为当前线程初始化 ThreadLocalMap，并存储键值对 <this, firstValue>
        createMap(t, value);
    }
}
```

set 方法的作用是把我们想要存储的 value 给保存进去。其主要流程为：

1. 先获取当当前线程的引用；
2. 利用这个引用来获取到 ThreadLocalMap；
3. 如果 map 为空，则去创建一个 ThreadLocalMap；
4. 如果 map 不为空，就利用 ThreadLocalMap 的 set 方法将 value 添加到 map 中。

> 其中 map 就是 ThreadLocalMap。

调用 ThreadLocalMap.set() 时，会把当前 `threadLocal` 对象作为 key，想要保存的对象作为 value，存入 map。

set 方法的时序图如下所示：



![img](https:////upload-images.jianshu.io/upload_images/10170978-e657cb49ec2fbaf8.png?imageMogr2/auto-orient/strip|imageView2/2/w/994/format/webp)

### ThreadLocal的 getMap 方法



```java
/**
 * 返回当前线程 thread 持有的 ThreadLocalMap
 *
 * @param t 当前线程
 * @return ThreadLocalMap
 */
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

getMap 方法的作用主要是获取当前线程内的 ThreadLocalMap 对象，原来这个 ThreadLocalMap 是线程Thread类的一个属性，我们来看看 Thread 中相关的代码：



```java
/**
 * ThreadLocal 的 ThreadLocalMap 是线程的一个属性，所以在多线程环境下 threadLocals 是线程安全的
 */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

### ThreadLocal的 get 方法

get 方法源码如下：



```java
/**
 * 返回当前 ThreadLocal 对象关联的值
 *
 * @return
 */
public T get() {
    // 返回当前 ThreadLocal 所在的线程
    Thread t = Thread.currentThread();
    // 从线程中拿到 ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // 从 map 中拿到 entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        // 如果不为空，读取当前 ThreadLocal 中保存的值
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T) e.value;
            return result;
        }
    }
    // 若 map 为空，则对当前线程的 ThreadLocal 进行初始化，最后返回当前的 ThreadLocal 对象关联的初值，即 value
    return setInitialValue();
}
```

get 方法的主要流程为：

1. 先获取到当前线程的引用；
2. 获取当前线程内部的 ThreadLocalMap；
3. 如果 map 存在，则获取当前 ThreadLocal 对应的 value 值；
4. 如果 map 不存在或者找不到 value 值，则调用 setInitialValue() 进行初始化。

get 方法的时序图如下所示：



![img](https:////upload-images.jianshu.io/upload_images/10170978-e7b60fa7ed9da737.png?imageMogr2/auto-orient/strip|imageView2/2/w/1046/format/webp)

其中每个 Thread 的 ThreadLocalMap 以 `threadLocal` 作为 key，保存自己的线程的 `value` 副本，也就是保存在每个线程中，并没有保存在 ThreadLocal 对象中。

### 小结

通过对源码的分析，现在我们来总结一下：

1. 每个Thread维护着一个ThreadLocalMap的引用；
2. ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储；
3. ThreadLocal创建的副本是存储在自己的threadLocals中的，也就是自己的ThreadLocalMap；
4. ThreadLocalMap的键值为ThreadLocal对象，而且可以有多个threadLocal变量，因此保存在map中；
5. 在进行get之前，必须先set，否则会报空指针异常，当然也可以初始化一个，但是必须重写initialValue()方法；
6. ThreadLocal本身并不存储值，它只是作为一个key来让线程从ThreadLocalMap获取value。

## ThreadLocal 应用场景

ThreadLocal 的特性也导致了应用场景比较广泛，主要的应用场景如下：

- 线程间数据隔离，各线程的 ThreadLocal 互不影响
- 方便同一个线程使用某一对象，避免不必要的参数传递
- 全链路追踪中的 traceId 或者流程引擎中上下文的传递一般采用 ThreadLocal
- Spring 事务管理器采用了 ThreadLocal
- Spring MVC 的 RequestContextHolder 的实现使用了 ThreadLocal

## 总结：面试常见问题

### Thread、ThreadLocal 以及 ThreadLocalMap关系

通过对以上源码的分析，Thread、ThreadLocal 以及 ThreadLocalMap 的关系有了进一步的理解，我们再通过一张图来总结下:



![img](https:////upload-images.jianshu.io/upload_images/10170978-5cb5653f46072b6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### ThreadLocal 是如何实现线程隔离的呢？

ThreadLocal 是如何做到线程数据隔离，前面源码分析 ThreadLocal 的 set 方法已经分析过，这里我们再总结一下：

ThreadLocal之所以能达到变量的线程隔离，其实就是每个线程都有一个自己的ThreadLocalMap对象来存储同一个threadLocal实例set的值，而取值的时候也是根据同一个threadLocal实例去自己的ThreadLocalMap里面找，自然就互不影响了，从而达到线程隔离的目的。如下图所示：



![img](https:////upload-images.jianshu.io/upload_images/10170978-6c0c0012266658c3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1119/format/webp)

### ThreadLocal内存泄漏问题

ThreadLocal 在没有外部**强引用**时，发生 GC时会被回收，那么 ThreadLocalMap 中保存的 key 值就变成了 null，而 Entry 又被 threadLocalMap 对象引用，threadLocalMap 对象又被 Thread 对象所引用，那么当 Thread 一直不终结的话，value 对象就会一直存在于内存中，也就导致了内存泄漏，直至 Thread 被销毁后，才会被回收。我们通过一张图来理解下：

![img](https:////upload-images.jianshu.io/upload_images/10170978-82151cf62958a93f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)


 ThreadLocal内存泄漏的根源是：**由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。**



#### 那么如何避免内存泄漏呢？

在使用完 ThreadLocal 变量后，需要我们手动 remove 掉，防止 ThreadLocalMap 中的 Entry 一直保持对 value 的强引用，导致 value 不能被回收。























































































