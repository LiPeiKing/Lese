## ConcurrentHashMap详解

一般在多线程的场景，我都会使用好几种不同的方式去代替：

- 使用Collections.synchronizedMap(Map)创建线程安全的map集合；
- Hashtable
- ConcurrentHashMap

### Collections.synchronizedMap

在SynchronizedMap内部维护了一个普通对象Map，还有排斥锁mutex，如图

![img](https://pic3.zhimg.com/50/v2-082c52252257cd8b1f65dd17639490f9_hd.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-082c52252257cd8b1f65dd17639490f9_720w.jpg?source=1940ef5c)

```text
Collections.synchronizedMap(new HashMap<>(16));
```

我们在调用这个方法的时候就需要传入一个Map，可以看到有两个构造器，如果你传入了mutex参数，则将对象排斥锁赋值为传入的对象。

如果没有，则将对象排斥锁赋值为this，即调用synchronizedMap的对象，就是上面的Map。

创建出synchronizedMap之后，再操作map的时候，就会对方法上锁，如图全是 

![img](https://pic1.zhimg.com/50/v2-fb6d545b340499bded625da1d80dbf45_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-fb6d545b340499bded625da1d80dbf45_720w.jpg?source=1940ef5c)



### Hashtable

跟HashMap相比Hashtable是线程安全的，适合在多线程的情况下使用，但是效率可不太乐观。

他在对数据操作的时候都会上锁，所以效率比较低下。

![img](https://pic2.zhimg.com/50/v2-05b448fb170aa9bcbe6250b6f842797d_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-05b448fb170aa9bcbe6250b6f842797d_720w.jpg?source=1940ef5c)



Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null。

因为Hashtable在我们put 空值的时候会直接抛空指针异常，但是HashMap却做了特殊处理。

```text
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这是因为Hashtable使用的是**安全失败机制（fail-safe）**，这种机制会使你此次读到的数据不一定是最新的数据。

如果你使用null值，就会使得其无法判断对应的key是不存在还是为空，因为你无法再调用一次contain(key）来对key是否存在进行判断，ConcurrentHashMap同理。

- **实现方式不同**：Hashtable 继承了 Dictionary类，而 HashMap 继承的是 AbstractMap 类。
     Dictionary 是 JDK 1.0 添加的，貌似没人用过这个，我也没用过。

- **初始化容量不同**：HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75。

- **扩容机制不同**：当现有容量大于总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1。

- **迭代器不同**：HashMap 中的 Iterator 迭代器是 fail-fast 的，而 Hashtable 的 Enumerator 不是 fail-fast 的。
     所以，当其他线程改变了HashMap 的结构，如：增加、删除元素，将会抛出ConcurrentModificationException 异常，而 Hashtable 则不会。



#### fail-fast是啥？

##### 概念

**快速失败（fail—fast）**是java集合中的一种机制， 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。

##### 原理

迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。

集合在被遍历期间如果内容发生变化，就会改变modCount的值。

每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

**Tip**：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。

因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。

##### 使用场景

java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）算是一种安全机制吧。



#### fail—safe

> 安全失败（fail—safe）： java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。



### ConcurrentHashMap 

jdk7以前跟以后(jdk8)实现原理不一样，所以我们分2个版本研究。



#### jdk7版本

ConcurrentHashMap和HashMap设计思路差不多，但是为支持并发操作，做了一定的改进，ConcurrentHashMap引入Segment 的概念，目的是将map拆分成多个Segment(默认16个)。操作ConcurrentHashMap细化到操作某一个Segment。在多线程环境下，不同线程操作不同的Segment，他们互不影响，这便可实现并发操作。

Segment 字面翻译是一段，部分的意思，但我们更多称之为”槽”。Segment 继承 ReentrantLock类 ，当我们对ConcurrentHashMap并发操作时，只要锁住一个 segment，其他剩余的Segment依然可以操作。这样只要保证每个 Segment 是线程安全的，我们就实现了全局的线程安全。
 具体结构：



![img](https:////upload-images.jianshu.io/upload_images/807144-4db95a9fa5fedc1c?imageMogr2/auto-orient/strip|imageView2/2/w/820/format/webp)

ConcurrentHashMap结构图

上图可看出ConcurrentHashMap的大体结构，ConcurrentHashMap由一个Segment[]数组组成，数组元素是一个数组+链表的结构。
 抽取部分源码：

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable {
  final Segment<K,V>[] segments;
  ...
}
```

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
  transient volatile HashEntry<K,V>[] table;
}
```

```java
static final class HashEntry<K,V> {
   final int hash;
   final K key;
   volatile V value;
   volatile HashEntry<K,V> next;
 }
```

从源码上看，我们可以看到Segment就类似一个小型的hashMap，ConcurrentHashMap就是HashMap集合。那么我们就来看下put添加操作：
 ConcurrentHashMap

```csharp
public V put(K key, V value) {
        Segment<K,V> s;
        //计算hash key值
        int hash = hash(key);
        //通过hash key值计算出存入Segment的位置
        int j = (hash >>> segmentShift) & segmentMask;
        if ((s = (Segment<K,V>)UNSAFE.getObject          
             (segments, (j << SSHIFT) + SBASE)) == null) 
            //初始化Segment
            s = ensureSegment(j);
         //添加
        return s.put(key, hash, value, false);
}
```

`Segment:`

```csharp
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    //segment操作加锁，使用尝试获取锁方式。如果获取失败，进入scanAndLockForPut方法
    HashEntry<K,V> node = tryLock() ? null :
    scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
    HashEntry<K,V>[] tab = table;
    int index = (tab.length - 1) & hash;
    HashEntry<K,V> first = entryAt(tab, index);
    for (HashEntry<K,V> e = first;;) {
        if (e != null) {
        K k;
        if ((k = e.key) == key ||
            (e.hash == hash && key.equals(k))) {
            oldValue = e.value;
            if (!onlyIfAbsent) {
            e.value = value;
            ++modCount;
            }
            break;
        }
        e = e.next;
        }
        else {
        if (node != null)
            node.setNext(first);
        else
            node = new HashEntry<K,V>(hash, key, value, first);
        int c = count + 1;
        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    //扩容， 这是后续做解释
            rehash(node);
        else
            setEntryAt(tab, index, node);
        ++modCount;
        count = c;
        oldValue = null;
        break;
        }
    }
    } finally {
    //释放锁
    unlock();
    }
    return oldValue;
}
```

上面几个方法，ConcurrentHashMap在进行put操作时，先通过key找到承载的Segment对象位置，然后竞争操作Segment的独占锁，以确保操作线程。获取锁方式很简单，就是tryLock()，如果获取锁失败，执行scanAndLockForPut方法

```csharp
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // 迭代次数
    while (!tryLock()) {
    HashEntry<K,V> f; 
    if (retries < 0) {
        if (e == null) {
        if (node == null) // speculatively create node
            node = new HashEntry<K,V>(hash, key, value, null);
        retries = 0;
        }
        else if (key.equals(e.key))
        retries = 0;
        else
        e = e.next;
    }
        //超过迭代次数，阻塞
    else if (++retries > MAX_SCAN_RETRIES) {
        lock();
        break;
    }
    else if ((retries & 1) == 0 &&
         (f = entryForHash(this, hash)) != first) {
        e = first = f; // re-traverse if entry changed
        retries = -1;
    }
    }
    return node;
}
```

scanAndLockForPut 实现也比较简单，循环调用tryLock，多次获取，如果循环次数retries 次数大于事先设置定好的MAX_SCAN_RETRIES，就执行lock() 方法，此方法会阻塞等待，一直到成功拿到Segment锁为止。

```dart
//循环次数，单核为1， 多核为64.
static final int MAX_SCAN_RETRIES =
            Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;
```

ConcurrentHashMap的扩容跟HashMap有点不同， ConcurrentHashMap的Segment槽是固定的16个，不变的

```css
final Segment<K,V>[] segments;
```

而ConcurrentHashMap的扩容讲的是Segment中的HashEntry数组扩容。当HashEntry达到某个临界点后，会扩容2为之前的2倍， 原理跟HashMap扩容类似。

现在我们，在看会put方法中一个if分支

```swift
if (c > threshold && tab.length < MAXIMUM_CAPACITY)
    //扩容， 这是后续做解释
    rehash(node);
else
```

rehash方法就是HashEntry扩展逻辑。当线程执行到rehash方法时，表示当前线程已经获取到到当前Segment的锁对象，这就表示rehash方法的执行是线程安全，不会存在并发问题。
 ConcurrentHashMap的remove方法跟put方法操作一样，先获取segement对象后再操作，这里就不重复了。那么我们来看下get操作：

```csharp
   public V get(Object key) {
        Segment<K,V> s; 
        HashEntry<K,V>[] tab;
        //1:计算key的hash值
        int h = hash(key);
        //2:确定在segment的位置，得到HashEntry数组
        long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
        if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
            (tab = s.table) != null) {
            //3:得到数据链表，迭代，查找key对应的value值
            for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                     (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
                 e != null; e = e.next) {
                K k;
                if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                    return e.value;
            }
        }
        return null;
    }
```

细心的朋友会发现get方法并没有获取锁的操作，这时就得讨论下执行get操作线程安全问题啦。
 1：一线程执行put，另一个线程执行get
 ConcurrentHashMap约定新添的节点是在链表的表头， 所以如果先执行get，后执行put， get操作已经遍历到链表中间了， 不会影响put的安全执行。如果先执行put，这时候，就必须保证刚刚插入的表头节点能被读取，ConcurrentHashMap使用的UNSAFE.putOrderedObject赋值方式保证。
 2：一个线程执行put，并在扩容操作期间， 另一个线程执行get
 ConcurrentHashMap扩容是新创建了HashEntry数组，然后进行迁移数据，最后面将 newTable赋值给oldTable。如果 get 先执行，那么就是在oldTable 上做查询操作，不发送线程安全问题；而如果put 先执行，那么 put 操作的可见性保证就是 oldTable使用了 volatile 关键字即可。

```java
transient volatile HashEntry<K,V>[] table;
```

3:一线程执行remove，另一个线程执行get
 ConcurrentHashMap的删除分2种情况， 1>删除节点在链表表头。那操作节点就是HashEntry数组元素了，虽然HashEntry[] table 使用了volatile修饰， 但是， volatile并保证数据内部元素的操作可见性，所以只能使用UNSAFE 来操作元素。2>删除节点中标中间， 那么好办， 只需要保证节点中的next属性是volatile修饰即可

```java
    static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
   }
```

总结：get方法之所以不需要加锁，原因比较简单，get为只读操作，不会改动map数据结构，所以在操作过程中，只需要保证涉及读取数据的属性为线程可见即可，也即使用volatile修饰。

#### jdk8版本

jdk8版本的ConcurrentHashMap相对于jdk7版本，发送了很大改动，jdk8直接抛弃了Segment的设计，采用了较为轻捷的Node + CAS + Synchronized设计，保证线程安全。



![img](https:////upload-images.jianshu.io/upload_images/807144-6264960638978dff?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp)

​											 <u>ConcurrentHashMap结构图</u>

看上图ConcurrentHashMap的大体结构，一个node数组，默认为16，可以自动扩展，扩展速度为0.75

```java
private static finalint DEFAULT_CONCURRENCY_LEVEL = 16;
private static final float LOAD_FACTOR = 0.75f;
```

每一个节点，挂载一个链表，当链表挂载数据大于8时，链表自动转换成红黑树

```dart
static final int TREEIFY_THRESHOLD = 8;
```

部分代码:

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    transient volatile Node<K,V>[] table;
}
```

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
}
```

```java
static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
}
```

```java
static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
 }
```

Node
 ConcurrentHashMap核心内部类，它包装了key-value键值对，所有插入ConcurrentHashMap的数据都包装在这里面。
 TreeNode
 树节点类，当数据链表长度大于8时，会转换为TreeNode。注意，此时的TreeNode并不是红黑树对象，它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中，由TreeBin完成对红黑树的包装。
 TreeBin
 TreeNode节点的包装对象，可以认为是红黑树对象。它代替了TreeNode的根节点，ConcurrentHashMap的node“数组”中，存放就是TreeBin对象，而不是TreeNode对象。
 来看下jdk8版本ConcurrentHashMap 的put操作

```csharp
    public V put(K key, V value) {
        return putVal(key, value, false);
    }
```

```csharp
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        //一个死循环，目的，并发情况下，也可以保障安全添加成功
        //原理：cas算法的循环比较，直至成功
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                //第一次添加，先初始化node数组
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //计算出table[i]无节点，创建节点
                //casTabAt : 底层使用Unsafe.compareAndSwapObject 原子操作table[i]位置，如果为null，则添加新建的node节点，跳出循环，反之，再循环进入执行添加操作
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   
            }
            else if ((fh = f.hash) == MOVED)
                 //如果当前处于拓展状态，返回拓展后的tab，然后再进入循环执行添加操作
                tab = helpTransfer(tab, f);
            else {
                //链表中或红黑树中追加节点
                V oldVal = null;
                //使用synchronized 对 f 对象加锁， 这个f = tabAt(tab, i = (n - 1) & hash) ：table[i] 的node对象，并发环境保证线程操作安全
               //此处注意： 这里没有ReentrantLock，因为jdk1.8对synchronized 做了优化，其执行性能已经跟ReentrantLock不相上下。
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        //链表上追加节点
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //红黑树上追加节点
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    //节点数大于临界值，转换成红黑树
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

从put源码可看，JDK8版本更多使用的cas编程方式控制线程安全， 必要时也会使用synchronized 代码块保证线程安全。
 最后，再看会ConcurrentHashMap的get方法：



```kotlin
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            //获取table[i] 的node元素
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```



```java
 //确保多线程可见，并且保证获取到是内存中最新的table[i] 元素值
 static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
```

get源码也没有加锁操作，操作原理跟jdk1.7版本一样，这里就不累赘了。

回到上篇中
 场景1：多线程复合操作时是否能保证线程安全
 答案是不能，原因： ConcurrentHashMap 使用锁分离(jdk7)/cas(jdk8)方式保证并发环境下，添加/删除操作安全，但这进针对的是单个put 或者 remove方法，如果多个方法配合复合使用，依然需要额外加锁。
 场景2：多线程同时添加相同hash 码值时转换成红黑树时，是否存在并发问题
 答案是可以保证线程安全，原因：ConcurrentHashMap 链表转换成红黑树时，对转换方法做加锁防护了



```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                tryPresize(n << 1);
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                synchronized (b) {
                    if (tabAt(tab, index) == b) {
                        TreeNode<K,V> hd = null, tl = null;
                        for (Node<K,V> e = b; e != null; e = e.next) {
                            TreeNode<K,V> p =
                                new TreeNode<K,V>(e.hash, e.key, e.val,
                                                  null, null);
                            if ((p.prev = tl) == null)
                                hd = p;
                            else
                                tl.next = p;
                            tl = p;
                        }
                        setTabAt(tab, index, new TreeBin<K,V>(hd));
                    }
                }
            }
        }
    }
```

最后对ConcurrentHashMap 来个大总结：
 1、get方法不加锁；
 2、put、remove方法要使用锁
 jdk7使用锁分离机制(Segment分段加锁)
 jdk8使用cas + synchronized 实现锁操作
 3、Iterator对象的使用，运行一边更新，一遍遍历(可以根据原理自己拓展)
 4、复合操作，无法保证线程安全，需要额外加锁保证
 5、并发环境下，ConcurrentHashMap 效率较Collections.synchronizedMap()更高



https://www.jianshu.com/p/1e1a96075256