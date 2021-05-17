## Redis常用命令

[TOC]



### key pattern 查询相应的key

- redis允许模糊查询key　　有3个通配符 *、?、[]

+ randomkey：返回随机key　　

+ type key：返回key存储的类型

+ exists key：判断某个key是否存在

+ del key：删除key

+ rename key newkey：改名

+ renamenx key newkey：如果newkey不存在则修改成功

+ move key 1：将key移动到1数据库

+ ttl key：查询key的生命周期（秒）。当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间。

+ expire key 整数值：设置key的生命周期以秒为单位

+ pexpire key 整数值：设置key的生命周期以毫秒为单位

+ pttl key：查询key 的生命周期（毫秒）

+ perisist key：把指定key设置为永久有效

### 字符串类型的操作

+ set key value [ex 秒数] [px 毫秒数] [nx/xx]　　

  　　如果ex和px同时写，则以后面的有效期为准

　　　　nx：如果key不存在则建立

　　　　xx：如果key存在则修改其值

+ get key：取值

+ mset key1 value1 key2 value2 一次设置多个值

+ mget key1 key2 ：一次获取多个值

+ setrange key offset value：把字符串的offset偏移字节改成value，如果偏移量 > 字符串长度，该字符自动补0x00

+ append key value ：把value追加到key 的原值上

+ getrange key start stop：获取字符串中[start, stop]范围的值，对于字符串的下标，左数从0开始，右数从-1开始

　　**注意：**当start>length，则返回空字符串，当stop>=length，则截取至字符串尾，如果start所处位置在stop右边，则返回空字符串

+ getset key nrevalue：获取并返回旧值，在设置新值

+ incr key：自增，返回新值，如果incr一个不是int的value则返回错误，incr一个不存在的key，则设置key为1

+ incrby key 2：跳2自增

+ incrbyfloat by 0.7： 自增浮点数　

+ setbit key offset value：设置offset对应二进制上的值，返回该位上的旧值

　　**注意：**如果offset过大，则会在中间填充0，offset最大到2^32-1，即可推出最大的字符串为512M

+ bitop operation destkey key1 [key2..]  对key1 key2做opecation并将结果保存在destkey上，opecation可以是AND OR NOT XOR

+ strlen key：取指定key的value值的长度

+ setex key time value：设置key对应的值value，并设置有效期为time秒

### 链表操作

Redis的list类型其实就是一个每个子元素都是string类型的双向链表，链表的最大长度是2^32。list既可以用做栈，也可以用做队列。list的pop操作还有阻塞版本，主要是为了避免轮询。

> 链表默认是从左侧增加，l代表左，r代表右。更好理解

+ lpush key value：把值插入到链表头部

+ rpush key value：把值插入到链表尾部

+ lpop key ：返回并删除链表头部元素

+ rpop key： 返回并删除链表尾部元素

+ lrange key start stop：返回链表中[start, stop]中的元素

+ lrem key count value：从链表中删除value值，删除count的绝对值个value后结束。

  count > 0 从表头删除　　

  count < 0 从表尾删除　　

  count=0 全部删除

+ ltrim key start stop：剪切key对应的链接，切[start, stop]一段并把改制重新赋给key

+ lindex key index：返回index索引上的值

+ llen key：计算链表的元素个数

+ linsert key after|before search value：在key 链表中寻找search，并在search值之前|之后插入value

+ rpoplpush source dest：把source 的末尾拿出，放到dest头部，并返回单元值

　　应用场景： task + bak 双链表完成安全队列

​	　业务逻辑： rpoplpush task bak	

　　接收返回值并做业务处理，如果成功则rpop bak清除任务，如果不成功，下次从bak表取任务

+ brpop，blpop key timeout：等待弹出key的尾/头元素

　　timeout为等待超时时间，如果timeout为0则一直等待下去

　　应用场景：长轮询ajax，在线聊天时能用到

### hashes类型及操作

​    

Redis hash 是一个string类型的field和value的映射表，它的添加、删除操作都是O(1)（平均）。hash特别适用于存储对象，将一个对象存储在hash类型中会占用更少的内存，并且可以方便的存取整个对象。

**配置：** hash_max_zipmap_entries 64 #配置字段最多64个

　　　　　 hash_max_zipmap_value 512 #配置value最大为512字节

+ hset myhash field value：设置myhash的field为value

+ hsetnx myhash field value：不存在的情况下设置myhash的field为value

+ hmset myhash field1 value1 field2 value2：同时设置多个field

+ hget myhash field：获取指定的hash field

+ hmget myhash field1 field2：一次获取多个field

+ hincrby myhash field 5：指定的hash field加上给定的值

+ hexists myhash field：测试指定的field是否存在

+ hlen myhash：返回hash的field数量

+ hdel myhash field：删除指定的field

+ hkeys myhash：返回hash所有的field

+ hvals myhash：返回hash所有的value

+ hgetall myhash：获取某个hash中全部的field及value　

### 集合结构操作

　特点：无序性、确定性、唯一性

+ sadd key value1 value2：往集合里面添加元素

+ smembers key：获取集合所有的元素

+ srem key value：删除集合某个元素

+ spop key：返回并删除集合中1个随机元素（可以坐抽奖，不会重复抽到某人）　　　

+ srandmember key：随机取一个元素

+ sismember key value：判断集合是否有某个值

+ scard key：返回集合元素的个数

+ smove source dest value：把source的value移动到dest集合中

+ sinter key1 key2 key3：求key1 key2 key3的交集

+ sunion key1 key2：求key1 key2 的并集

+ sdiff key1 key2：求key1 key2的差集

+ sinterstore res key1 key2：求key1 key2的交集并存在res里　

### 有序集合

概念：它是在set的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，zset会自动按新的值调整顺序。可以理解为有两列的mysql表，一列存储value，一列存储顺序，操作中key理解为zset的名字。

和set一样sorted，sets也是string类型元素的集合，不同的是每个元素都会关联一个double型的score。sorted set的实现是skip list和hash table的混合体。

当元素被添加到集合中时，一个元素到score的映射被添加到hash table中，所以给定一个元素获取score的开销是O(1)。另一个score到元素的映射被添加的skip list，并按照score排序，所以就可以有序地获取集合中的元素。添加、删除操作开销都是O(logN)和skip list的开销一致，redis的skip list 实现是双向链表，这样就可以逆序从尾部去元素。sorted set最经常使用方式应该就是作为索引来使用，我们可以把要排序的字段作为score存储，对象的ID当元素存储。

+ zadd key score1 value1：添加元素

+ zrange key start stop [withscore]：把集合排序后,返回名次[start,stop]的元素 默认是升续排列 withscores 是把score也打印出来

+ zrank key member：查询member的排名（升序0名开始）

+ zrangebyscore key min max [withscores] limit offset N：集合（升序）排序后取score在[min, max]内的元素，并跳过offset个，取出N个

+ zrevrank key member：查询member排名（降序 0名开始）

+ zremrangebyscore key min max：按照score来删除元素，删除score在[min, max]之间

+ zrem key value1 value2：删除集合中的元素

+ zremrangebyrank key start end：按排名删除元素，删除名次在[start, end]之间的

+ zcard key：返回集合元素的个数

+ zcount key min max：返回[min, max]区间内元素数量

+ zinterstore dest numkeys key1[key2..] [WEIGHTS weight1 [weight2...]] [AGGREGATE SUM|MIN|MAX]

　　求key1，key2的交集，key1，key2的权值分别是weight1，weight2

　　聚合方法用 sum|min|max

　　聚合结果 保存子dest集合内

　　注意：weights,aggregate如何理解？

　　答：如果有交集，交集元素又有score，score怎么处理？aggregate num->score相加，min最小score，max最大score，另外可以				通过weights设置不同的key的权重，交集时 score*weight

### HyperLogLog

#### 简介

Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

#### 什么是基数?

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

**实例**

以下实例演示了 HyperLogLog 的工作过程：

```shell
redis 127.0.0.1:6379> PFADD w3ckey "redis"
 
1) (integer) 1
 
redis 127.0.0.1:6379> PFADD w3ckey "mongodb"
 
1) (integer) 1
 
redis 127.0.0.1:6379> PFADD w3ckey "mysql"
 
1) (integer) 1
 
redis 127.0.0.1:6379> PFCOUNT w3ckey
 
(integer) 3
```

#### Redis HyperLogLog 命令

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PFADD key element element ...\]](https://www.redis.net.cn/order/3629.html) 添加指定元素到 HyperLogLog 中。 |
| 2    | [PFCOUNT key key ...\]](https://www.redis.net.cn/order/3630.html) 返回给定 HyperLogLog 的基数估算值。 |
| 3    | [PFMERGE destkey sourcekey sourcekey ...\]](https://www.redis.net.cn/order/3631.html) 将多个 HyperLogLog 合并为一个 HyperLogLog |

### 服务器相关命令

+ ping：测定连接是否存活

+ echo：在命令行打印一些内容

+ select：选择数据库

+ quit：退出连接

+ dbsize：返回当前数据库中key的数目

+ info：获取服务器的信息和统计

+ monitor：实时转储收到的请求

+ config get 配置项：获取服务器配置的信息

　　config set 配置项 值：设置配置项信息

+ flushdb：删除当前选择数据库中所有的key

+ flushall：删除所有数据库中的所有的key

+ time：显示服务器时间，时间戳（秒），微秒数

+ bgrewriteaof：后台保存rdb快照

+ bgsave：后台保存rdb快照

+ save：保存rdb快照

+ lastsave：上次保存时间

+ shutdown [save/nosave]

  注意：如果不小心运行了flushall，立即shutdown nosave，关闭服务器，然后手工编辑aof文件，去掉文件中的flushall相关行，然后开启服务器，就可以倒回原来是数据。如果flushall之后，系统恰好bgwriteaof了，那么aof就清空了，数据丢失。

+ showlog：显示慢查询

　　问：多慢才叫慢？

　　答：由slowlog-log-slower-than 10000，来指定（单位为微秒）

　　问：服务器存储多少条慢查询记录

　　答：由slowlog-max-len 128，来做限制　

## Redis事务

[MULTI](http://www.redis.cn/commands/multi.html) 、 [EXEC](http://www.redis.cn/commands/exec.html) 、 [DISCARD](http://www.redis.cn/commands/discard.html) 和 [WATCH](http://www.redis.cn/commands/watch.html) 是 Redis 事务相关的命令。事务可以一次执行多个命令， 并且带有以下两个重要的保证：

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
- 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。

[EXEC](http://www.redis.cn/commands/exec.html) 命令负责触发并执行事务中的所有命令：

- 如果客户端在使用 [MULTI](http://www.redis.cn/commands/multi.html) 开启了一个事务之后，却因为断线而没有成功执行 [EXEC](http://www.redis.cn/commands/exec.html) ，那么事务中的所有命令都不会被执行。
- 另一方面，如果客户端成功在开启事务之后执行 [EXEC](http://www.redis.cn/commands/exec.html) ，那么事务中的所有命令都会被执行。

当使用 AOF 方式做持久化的时候， Redis 会使用单个 write(2) 命令将事务写入到磁盘中。

然而，如果 Redis 服务器因为某些原因被管理员杀死，或者遇上某种硬件故障，那么可能只有部分事务命令会被成功写入到磁盘中。

如果 Redis 在重新启动时发现 AOF 文件出了这样的问题，那么它会退出，并汇报一个错误。

使用`redis-check-aof`程序可以修复这一问题：它会移除 AOF 文件中不完整事务的信息，确保服务器可以顺利启动。

从 2.2 版本开始，Redis 还可以通过乐观锁（optimistic lock）实现 CAS （check-and-set）操作，具体信息请参考文档的后半部分。

### 用法

[MULTI](http://www.redis.cn/commands/multi.html) 命令用于开启一个事务，它总是返回 `OK` 。 [MULTI](http://www.redis.cn/commands/multi.html) 执行之后， 客户端可以继续向服务器发送任意多条命令， 这些命令不会立即被执行， 而是被放到一个队列中， 当 [EXEC](http://www.redis.cn/commands/exec.html)命令被调用时， 所有队列中的命令才会被执行。

另一方面， 通过调用 [DISCARD](http://www.redis.cn/commands/discard.html) ， 客户端可以清空事务队列， 并放弃执行事务。

以下是一个事务例子， 它原子地增加了 `foo` 和 `bar` 两个键的值：

```bash
> set foo 1
OK
> set bar 1
OK
> multi
OK
> incr foo
QUEUED
> incr bar
QUEUED
> get foo
QUEUED
> get bar
QUEUED
> exec
2
2
2
2
```

[EXEC](http://www.redis.cn/commands/exec.html) 命令的回复是一个数组， 数组中的每个元素都是执行事务中的命令所产生的回复。 其中， 回复元素的先后顺序和命令发送的先后顺序一致。

当客户端处于事务状态时， 所有传入的命令都会返回一个内容为 `QUEUED` 的状态回复（status reply）， 这些被入队的命令将在 EXEC 命令被调用时执行。

### 事务中的错误

使用事务时可能会遇上以下两种错误：

- 事务在执行 [EXEC](http://www.redis.cn/commands/exec.html) 之前，入队的命令可能会出错。比如说，命令可能会产生语法错误（参数数量错误，参数名错误，等等），或者其他更严重的错误，比如内存不足（如果服务器使用 `maxmemory` 设置了最大内存限制的话）。
- 命令可能在 [EXEC](http://www.redis.cn/commands/exec.html) 调用之后失败。举个例子，事务中的命令可能处理了错误类型的键，比如将列表命令用在了字符串键上面，诸如此类。

对于发生在 [EXEC](http://www.redis.cn/commands/exec.html) 执行之前的错误，客户端以前的做法是检查命令入队所得的返回值：如果命令入队时返回 `QUEUED` ，那么入队成功；否则，就是入队失败。如果有命令在入队时失败，那么大部分客户端都会停止并取消这个事务。

至于那些在 [EXEC](http://www.redis.cn/commands/exec.html) 命令执行之后所产生的错误， 并没有对它们进行特别处理： 即使事务中有某个/某些命令在执行时产生了错误， 事务中的其他命令仍然会继续执行。

### 为什么 Redis 不支持回滚（roll back）

以下是这种做法的优点：

- Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
- 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， 回滚并不能解决编程错误带来的问题。 举个例子， 如果你本来想通过 [INCR](http://www.redis.cn/commands/incr.html) 命令将键的值加上 1 ， 却不小心加上了 2 ， 又或者对错误类型的键执行了 [INCR](http://www.redis.cn/commands/incr.html) ， 回滚是没有办法处理这些情况的。

### 放弃事务

当执行 [DISCARD](http://www.redis.cn/commands/discard.html) 命令时， 事务会被放弃， 事务队列会被清空， 并且客户端会从事务状态中退出：

### 使用 check-and-set 操作实现乐观锁

[WATCH](http://www.redis.cn/commands/watch.html) 命令可以为 Redis 事务提供 check-and-set （CAS）行为。

被 [WATCH](http://www.redis.cn/commands/watch.html) 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 [EXEC](http://www.redis.cn/commands/exec.html) 执行之前被修改了， 那么整个事务都会被取消， [EXEC](http://www.redis.cn/commands/exec.html) 返回[nil-reply](http://www.redis.cn/topics/protocol.html#nil-reply)来表示事务已经失败。

当多个客户端同时对同一个键进行这样的操作时， 就会产生竞争条件。举个例子， 如果客户端 A 和 B 都读取了键原来的值， 比如 10 ， 那么两个客户端都会将键的值设为 11 ， 但正确的结果应该是 12 才对。

有了 [WATCH](http://www.redis.cn/commands/watch.html) ， 我们就可以轻松地解决这类问题了：

```bash
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```

使用上面的代码， 如果在 [WATCH](http://www.redis.cn/commands/watch.html) 执行之后， [EXEC](http://www.redis.cn/commands/exec.html) 执行之前， 有其他客户端修改了 `mykey` 的值， 那么当前客户端的事务就会失败。 程序需要做的， 就是不断重试这个操作， 直到没有发生碰撞为止。

这种形式的锁被称作乐观锁， 它是一种非常强大的锁机制。 并且因为大多数情况下， 不同的客户端会访问不同的键， 碰撞的情况一般都很少， 所以通常并不需要进行重试。

### 了解 `WATCH`

[WATCH](http://www.redis.cn/commands/watch.html) 使得 [EXEC](http://www.redis.cn/commands/exec.html) 命令需要有条件地执行： 事务只能在所有被监视键都没有被修改的前提下执行， 如果这个前提不能满足的话，事务就不会被执行。 [了解更多->](http://code.google.com/p/redis/issues/detail?id=270)

[WATCH](http://www.redis.cn/commands/watch.html) 命令可以被调用多次。 对键的监视从 [WATCH](http://www.redis.cn/commands/watch.html) 执行之后开始生效， 直到调用 [EXEC](http://www.redis.cn/commands/exec.html) 为止。

用户还可以在单个 [WATCH](http://www.redis.cn/commands/watch.html) 命令中监视任意多个键， 就像这样：

```bash
redis> WATCH key1 key2 key3
OK
```

当 [EXEC](http://www.redis.cn/commands/exec.html) 被调用时， 不管事务是否成功执行， 对所有键的监视都会被取消。

另外， 当客户端断开连接时， 该客户端对键的监视也会被取消。

使用无参数的 [UNWATCH](http://www.redis.cn/commands/unwatch.html) 命令可以手动取消对所有键的监视。 对于一些需要改动多个键的事务， 有时候程序需要同时对多个键进行加锁， 然后检查这些键的当前值是否符合程序的要求。 当值达不到要求时， 就可以使用 [UNWATCH](http://www.redis.cn/commands/unwatch.html) 命令来取消目前对键的监视， 中途放弃这个事务， 并等待事务的下次尝试。

### 使用 WATCH 实现 ZPOP

[WATCH](http://www.redis.cn/commands/watch.html) 可以用于创建 Redis 没有内置的原子操作。举个例子， 以下代码实现了原创的 [ZPOP](http://www.redis.cn/commands/zpop.html) 命令， 它可以原子地弹出有序集合中分值（score）最小的元素：

```bash
WATCH zset
element = ZRANGE zset 0 0
MULTI
ZREM zset element
EXEC
```

程序只要重复执行这段代码， 直到 [EXEC](http://www.redis.cn/commands/exec.html) 的返回值不是[nil-reply](http://www.redis.cn/topics/protocol.html#nil-reply)回复即可。

### Redis 脚本和事务

从定义上来说， Redis 中的脚本本身就是一种事务， 所以任何在事务里可以完成的事， 在脚本里面也能完成。 并且一般来说， 使用脚本要来得更简单，并且速度更快。

因为脚本功能是 Redis 2.6 才引入的， 而事务功能则更早之前就存在了， 所以 Redis 才会同时存在两种处理事务的方法。

不过我们并不打算在短时间内就移除事务功能， 因为事务提供了一种即使不使用脚本， 也可以避免竞争条件的方法， 而且事务本身的实现并不复杂。

不过在不远的将来， 可能所有用户都会只使用脚本来实现事务也说不定。 如果真的发生这种情况的话， 那么我们将废弃并最终移除事务功能。





























