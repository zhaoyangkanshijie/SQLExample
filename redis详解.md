# redis详解

* 参考链接

    [「查缺补漏」巩固你的Redis知识体系](https://juejin.im/post/6857667542652190728#heading-28)

* [Redis使用场景](#Redis使用场景)
    * [String-缓存](#String-缓存)
    * [String-限流|计数器](#String-限流|计数器)
    * [String-分布式锁](#String-分布式锁)
    * [String-分布式Session](#String-分布式Session)
    * [List简单队列-栈](#List简单队列-栈)
    * [List社交类APP-好友列表](#List社交类APP-好友列表)
    * [Set抽奖|好友关系（合，并，交集）](#Set抽奖|好友关系（合，并，交集）)
    * [Zset排行榜](#Zset排行榜)
* [为什么Redis能这么快](#为什么Redis能这么快)
* [从海量Key里查询出某一个固定前缀的Key](#从海量Key里查询出某一个固定前缀的Key)
* [如何实现异步队列](#如何实现异步队列)
* [Redis持久化](#Redis持久化)
* [redis通讯协议(RESP)](#redis通讯协议(RESP))
* [redis架构有哪些](#redis架构有哪些)
* [Redis集群-如何从海量数据里快速找到所需？](#Redis集群-如何从海量数据里快速找到所需？)
* [缓存](#缓存)
* [缓存与数据库双写一致](#缓存与数据库双写一致)
* [何保证Redis中的数据都是热点数据](#何保证Redis中的数据都是热点数据)
* [Redis的并发竞争问题如何解决?](#Redis的并发竞争问题如何解决?)
* [Redis回收进程如何工作的?](#Redis回收进程如何工作的?)
* [使用Lua脚本的好处](#使用Lua脚本的好处)
* [Redis慢查询分析](#Redis慢查询分析)
* [提高Redis处理效率](#提高Redis处理效率)

---

## Redis使用场景

### String-缓存

```java
// 1.Cacheable 注解
// controller 调用 service 时自动判断有没有缓存，如果有就走redis缓存直接返回，如果没有则数据库然后自动放入redis中
// 可以设置过期时间，KEY生成规则 （KEY生成规则基于 参数的toString方法）
@Cacheable(value = "yearScore", key = "#yearScore")
@Override
public List<YearScore> findBy (YearScore yearScore) {}

// 2.手动用缓存
if (redis.hasKey(???) {
    return ....
} 

redis.set(find from DB)...
```

### String-限流|计数器

```java
// 注：这只是一个最简单的Demo 效率低，耗时旧，但核心就是这个意思
// 计数器也是利用单线程incr...等等
@RequestMapping("/redisLimit")
public String testRedisLimit(String uuid) {
    if (jedis.get(uuid) != null) {
        Long incr = jedis.incr(uuid);
        if (incr > MAX_LIMITTIME) {
            return "Failure Request";
        } else {
            return "Success Request";
        }
    }

    // 设置Key 起始请求为1，10秒过期  ->  实际写法肯定封装过,这里就是随便一写
    jedis.set(uuid, "1");
    jedis.expire(uuid, 10);
    return "Success Request";
}

```

### String-分布式锁

```java
/***
 * 核心思路：
 *     分布式服务调用时setnx,返回1证明拿到，用完了删除，返回0就证明被锁，等...
 *     SET KEY value [EX seconds] [PX milliseconds] [NX|XX]
 *     EX second:设置键的过期时间为second秒
 *     PX millisecond:设置键的过期时间为millisecond毫秒
 *     NX：只在键不存在时，才对键进行设置操作
 *     XX:只在键已经存在时，才对键进行设置操作
 *
 * 1.设置锁
 *     A. 分布式业务统一Key
 *     B. 设置Key过期时间
 *     C. 设置随机value,利用ThreadLocal 线程私有存储随机value
 *
 * 2.业务处理
 *     ...
 *
 * 3.解锁
 *     A. 无论如何必须解锁 - finally (超时时间和finally 双保证)
 *     B. 要对比是否是本线程上的锁，所以要对比线程私有value和存储的value是否一致(避免把别人加锁的东西删除了)
 */
@RequestMapping("/redisLock")
public String testRedisLock () {
    try {
        for(;;){
            RedisContextHolder.clear();
            String uuid = UUID.randomUUID().toString();

            String set = jedis.set(KEY, uuid, "NX", "EX", 1000);
            RedisContextHolder.setValue(uuid);

            if (!"OK".equals(set)) {
                // 进入循环-可以短时间休眠
            } else {
                // 获取锁成功 Do Somethings....
                break;
            }
        }
    } finally {
        // 解锁 -> 保证获取数据，判断一致以及删除数据三个操作是原子的， 因此如下写法是不符合的
        /*if (RedisContextHolder.getValue() != null && jedis.get(KEY) != null && RedisContextHolder.getValue().equals(jedis.get(KEY))) {
                jedis.del(KEY);
            }*/

        // 正确姿势 -> 使用Lua脚本,保证原子性
        String luaScript = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
        Object eval = jedis.eval(luaScript, Collections.singletonList(KEY), Collections.singletonList(RedisContextHolder.getValue()));
    }
    return "锁创建成功-业务处理成功";
}
```

### String-分布式Session

需要分布式session -> nginx负载均衡 分发到不同的Tomcat，即使利用IP分发，可以利用request获取session，但是其中一个挂了，旧需要分布式session

A服务-用户校验服务，B服务-业务层

* 情况A：

    A,B 服务单机部署：

    cookie：登录成功后，存储信息到cookie，A服务自身通过request设置session，获取session，B服务通过唯一key或者userid 查询数据库获取用户信息

    cookie+redis：登录成功后，存储信息到cookie，A服务自身通过request设置session，获取session，B服务通过唯一key或者userid 查询redis获取用户信息

* 情况B：
    A服务多节点部署，B服务多节点部署

    B服务获取用户信息的方式其实是不重要的，必然要查，要么从数据库，要么从cookie

    A服务：登录成功后，存储唯一key到cookie， 与此同时，A服务需要把session（KEY-UserInfo）同步到redis中，不能存在单纯的request（否则nginx分发到另一个服务器就完犊子了）

* 官方实现：

    spring-session-data-redis

    有一个内置拦截器，拦截request，session通过redis交互，普通使用代码依然是request.getSession....  但是实际上这个session的值已经被该组件拦截，通过redis进行同步了

### List简单队列-栈

```java
// 说白了利用redis - list数据结构 支持从左从右push，从左从右pop
@Component
public class RedisStack {

    @Resource
    Jedis jedis;

    private final static String KEY = "Stack";

    /** push **/
    public void push (String value) {
        jedis.lpush(KEY, value);
    }

    /** pop **/
    public String pop () {
        return jedis.lpop(KEY);
    }
}
```
```java
@Component
public class RedisQueue {

    @Resource
    JedisPool jedisPool;

    private final static String KEY = "Queue";

    /** push **/
    public void push (String value) {
        Jedis jedis = jedisPool.getResource();
        jedis.lpush(KEY, value);
    }

    /** pop **/
    public String pop () {
        Jedis jedis = jedisPool.getResource();
        return jedis.rpop(KEY);
    }
}

```

### List社交类APP-好友列表

根据时间显示好友，多个好友列表，求交集，并集，显示共同好友等

### Set抽奖|好友关系（合，并，交集）

```redis
// 插入key 及用户id
sadd cat:1 001 002 003 004 005 006

// 返回抽奖参与人数
scard cat:1

// 随机抽取一个
srandmember cat:1

// 随机抽取一人，并移除
spop cat:1
```

### Zset排行榜

根据分数实现有序列表

微博热搜：每点击一次 分数+1 即可

--- 不用数据库目的是因为避免order by 进行全表扫描

## 为什么Redis能这么快

1. Redis完全基于内存，绝大部分请求是纯粹的内存操作，执行效率高。
2. Redis使用单进程单线程模型的(K,V)数据库，将数据存储在内存中，存取均不会受到硬盘IO的限制，因此其执行速度极快，另外单线程也能处理高并发请求，还可以避免频繁上下文切换和锁的竞争，同时由于单线程操作，也可以避免各种锁的使用，进一步提高效率
3. 数据结构简单，对数据操作也简单，Redis不使用表，不会强制用户对各个关系进行关联，不会有复杂的关系限制，其存储结构就是键值对，类似于HashMap，HashMap最大的优点就是存取的时间复杂度为O(1)
4. C语言编写，效率更高
5. Redis使用多路I/O复用模型，为非阻塞IO
6. 有专门设计的RESP协议

补充：

常见的IO模型有四种：

1. 同步阻塞IO（Blocking IO）：即传统的IO模型。

2. 同步非阻塞IO（Non-blocking IO）：默认创建的socket都是阻塞的，非阻塞IO要求socket被设置为NONBLOCK。注意这里所说的NIO并非Java的NIO（New IO）库。

3. IO多路复用（IO Multiplexing）：即经典的Reactor设计模式，有时也称为异步阻塞IO，Java中的Selector和Linux中的epoll都是这种模型。

4. 异步IO（Asynchronous IO）：即经典的Proactor设计模式，也称为异步非阻塞IO

服务端是单线程的，多客户端连接时候，如果客户端没有发起任何动作，则服务端会把其视为不活跃的IO流，将其挂起，当有真正的动作时，会通过回调的方式执行相应的事件

## 从海量Key里查询出某一个固定前缀的Key

1. KEYS [pattern] 注意key很多的话，这样做肯定会出问题，造成redis崩溃

2. SCAN cursor [MATCH pattern][COUNT count] 游标方式查找

## 如何实现异步队列

假设场景:A服务生产数据 - B服务消费数据，即可利用此种模型构造-生产消费者模型

1. 使用Redis中的List作为队列

2. 使用BLPOP key [key...] timeout  -> LPOP key [key ...] timeout:阻塞直到队列有消息或者超时

3. pub/sub：主题订阅者模式

    缺点:消息的发布是无状态的，无法保证可达。对于发布者来说，消息是“即发即失”的，此时如果某个消费者在生产者发布消息时下线，重新上线之后，是无法接收该消息的，要解决该问题需要使用专业的消息队列

## Redis持久化

持久化就是把内存的数据写到磁盘中去，防止服务宕机了内存数据丢失。

Redis 提供了两种持久化方式:RDB（默认）和AOF：

1. RDB(Redis DataBase)

    把当前进程数据生成快照文件保存到硬盘的过程。分为手动触发和自动触发

    * 手动触发 -> save (不推荐，阻塞严重) bgsave -> （save的优化版，微秒级阻塞）

        shutdowm 关闭服务时，如果没有配置AOF，则会使用bgsave持久化数据

        bgsave会从当前父进程fork一个子进程，然后生成rdb文件,缺点：频率低，无法做到实时持久化

2. AOF(Append-only file)

    AOF文件存储的也是RESP协议

    每当执行服务器(定时)任务或者函数时flushAppendOnlyFile 函数都会被调用， 这个函数执行以下两个工作:

    1. WRITE：根据条件，将 aof_buf 中的缓存写入到 AOF 文件

    2. SAVE：根据条件，调用 fsync 或 fdatasync 函数，将 AOF 文件保存到磁盘中。

    存储结构:内容是redis通讯协议(RESP )格式的命令文本存储

    原理：相当于存储了redis的执行命令(类似mysql的sql语句日志)，数据的完整性和一致性更高

3. 比较

    1. aof文件比rdb更新频率高

    2. aof比rdb更安全

    3. rdb性能更好

## redis通讯协议(RESP)

Redis 即 REmote Dictionary Server (远程字典服务)；

而Redis的协议规范是 Redis Serialization Protocol (Redis序列化协议)

RESP 是redis客户端和服务端之前使用的一种通讯协议；

RESP 的特点：实现简单、快速解析、可读性好

```txt
set key value 协议翻译如下：

* 3    ->  表示以下有几组命令

$ 3    ->  表示命令长度是3
SET

$6     ->  表示长度是6
keykey

$5     ->  表示长度是5
value

完整即：
* 3
$ 3
SET
$6
keykey
$5 
value

服务器在执行最后一条命令后，返回结果，返回格式如下：
For Simple Strings the first byte of the reply is "+" 回复
For Errors the first byte of the reply is "-" 错误
For Integers the first byte of the reply is ":" 整数
For Bulk Strings the first byte of the reply is "$" 字符串
For Arrays the first byte of the reply is "*" 数组
```

## redis架构有哪些

* 单节点

    * 主从复制

        Master-slave  主从赋值，此种结构可以考虑关闭master的持久化，只让从数据库进行持久化，另外可以通过读写分离，缓解主服务器压力

    * 哨兵

        Redis sentinel 是一个分布式系统中监控 redis 主从服务器，并在主服务器下线时自动进行故障转移。其中三个特性：

        1. 监控（Monitoring）：Sentinel会不断地检查你的主服务器和从服务器是否运作正常。

        2. 提醒（Notification）：当被监控的某个 Redis 服务器出现问题时，Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。

        3. 自动故障迁移（Automatic failover）：当一个主服务器不能正常工作时，Sentinel 会开始一次自动故障迁移操作。

        特点：

        1. 保证高可用

        2. 监控各个节点

        3. 自动故障迁移


        缺点：
        
        1. 主从模式，切换需要时间丢数据

        2. 没有解决 master 写的压力

* 集群

从redis 3.0之后版本支持redis-cluster集群，Redis-Cluster采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

特点：

* 无中心架构（不存在哪个节点影响性能瓶颈），少了 proxy 层。
* 数据按照 slot 存储分布在多个节点，节点间数据共享，可动态调整数据分布。
* 可扩展性，可线性扩展到 1000 个节点，节点可动态添加或删除。
* 高可用性，部分节点不可用时，集群仍可用。通过增加 Slave 做备份数据副本
* 实现故障自动 failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave到 Master 的角色提升。

缺点：

* 资源隔离性较差，容易出现相互影响的情况。
* 数据通过异步复制,不保证数据的强一致性

## Redis集群-如何从海量数据里快速找到所需？

* 分片

    按照某种规则去划分数据，分散存储在多个节点上。通过将数据分到多个Redis服务器上，来减轻单个Redis服务器的压力。

* 一致性Hash算法

    分片，通常的做法就是获取节点的Hash值,该算法对2^32 取模，将Hash值空间组成虚拟的圆环，整个圆环按顺时针方向组织，每个节点依次为0、1、2...2^32-1，之后将每个服务器进行Hash运算，确定服务器在这个Hash环上的地址，确定了服务器地址后，对数据使用同样的Hash算法，将数据定位到特定的Redis服务器上。

    * Hash环的数据倾斜问题

        Hash环在服务器节点很少的时候，容易遇到服务器节点不均匀的问题，这会造成数据倾斜，数据倾斜指的是被缓存的对象大部分集中在Redis集群的其中一台或几台服务器上。

## 缓存

* 缓存穿透

    一般的缓存系统，都是按照key去缓存查询，如果不存在对应的value，就应该去后端系统查找（比如DB）。一些恶意的请求会故意查询不存在的key,请求量很大，就会对后端系统造成很大的压力。这就叫做缓存穿透。

    * 如何避免？

    1. 对查询结果为空的情况也进行缓存，缓存时间设置短一点，或者该key对应的数据insert了之后清理缓存。

    2. 对一定不存在的key进行过滤。可以把所有的可能存在的key放到一个大的Bitmap中，查询时通过该bitmap过滤。

    3. 由于请求参数是不合法的（每次都请求不存在的参数），于是我们可以使用布隆过滤器（Bloomfilter）或压缩filter提前进行拦截，不合法就不让这个请求进入到数据库层

* 缓存雪崩

    当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，会给后端系统带来很大压力。导致系统崩溃。

    * 如何避免？

    1. 在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

    2. 做二级缓存，A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期

    3. 不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。

    4. 启用限流策略，尽量避免数据库被干掉

* 缓存击穿

    一个存在的key，在缓存过期的一刻，同时有大量的请求，这些请求都会击穿到DB，造成瞬时DB请求量大、压力骤增。

    * 解决方案

    1. 在访问key之前，采用SETNX（set if not exists）来设置另一个短期key来锁住当前key的访问，访问结束再删除该短期key

    2. 服务层处理 - 方法加锁 + 双重校验

## 缓存与数据库双写一致

如果仅仅是读数据和新增数据，没有此类问题

当数据需要更新时，三种更新策略保证缓存与数据库的双写一致性

1. 先更新数据库，再更新缓存

    执行顺序无法保证，加锁的话，确实可以避免，但这样吞吐量会下降，可以根据业务场景考虑

2. 先删除缓存，再更新数据库

    有一个请求A进行更新操作，另一个请求B进行查询操作。那么会出现如下情形:
    1. 请求A进行写操作，删除缓存
    2. 请求B查询发现缓存不存在
    3. 请求B去数据库查询得到旧值
    4. 请求B将旧值写入缓存
    5. 请求A将新值写入数据库

    因此采用：采用延时双删策略，即进入逻辑就删除Key，执行完操作，延时再删除key

3. 先更新数据库，再删除缓存

    可能出现问题的场景：
    1. 缓存刚好失效
    2. 请求A查询数据库，得一个旧值
    3. 请求B将新值写入数据库
    4. 请求B删除缓存
    5. 请求A将查到的旧值写入缓存

    处理方法：
    * 给键设置合理的过期时间
    * 异步延时删除key

## 何保证Redis中的数据都是热点数据

* 可以通过手工或者主动方式，去加载热点数据
* Redis有其自己的数据淘汰策略：

    redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略（回收策略）。redis 提供 6种数据淘汰策略：

    1. volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
    2. volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
    3. volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
    4. allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
    5. allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
    6. no-enviction（驱逐）：禁止驱逐数据

## Redis的并发竞争问题如何解决?

多线程同时操作统一Key的解决办法：

Redis为单进程单线程模式，采用队列模式将并发访问变为串行访问。Redis本身没有锁的概念，Redis对于多个客户端连接并不存在竞争，但是在Jedis客户端对Redis进行并发访问时会发生连接超时、数据转换错误、阻塞、客户端关闭连接等问题，这些问题均是由于客户端连接混乱造成

对此有多种解决方法：

1. 条件允许的情况下，请使用redis自带的incr命令,decr命令

2. 乐观锁方式

```sql
watch price
get price $price
$price = $price + 10
multi
set price $price
exec
```

3. 针对客户端，操作同一个key的时候，进行加锁处理

4. 场景允许的话，使用setnx 实现

## Redis回收进程如何工作的?

当所需内存超过配置的最大内存时，redis会启用数据淘汰规则

默认规则是：# maxmemory-policy noeviction

即只允许读，无法继续添加key

因此常需要配置淘汰策略，比如LRU算法

## Redis大批量增加数据

使用管道模式:cat data.txt | redis-cli --pipe

data.txt文本
```txt
SET Key0 Value0
SET Key1 Value1
...
SET KeyN ValueN
```

## 使用Lua脚本的好处

减少网络开销。可以将多个请求通过脚本的形式一次发送，减少网络时延

原子操作，redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。因此在编写脚本的过程中无需担心会出现竞态条件，无需使用事务

复用，客户端发送的脚本会永久存在redis中，这样，其他客户端可以复用这一脚本而不需要使用代码完成相同的逻辑

## Redis慢查询分析

redis 命令会放在redis内置队列中，然后主线程一个个执行，因此 其中一个 命令执行时间过长，会造成成批量的阻塞

获取慢查询记录:slowlog get

获取慢查询记录量:slowlog len

(慢查询队列是先进先出的，因此新的值在满载的时候，旧的会出去)

Redis 慢查询 -> 执行阶段耗时过长 

conf文件设置： 
```txt
slowlog-low-slower-than 10000 -> 10000微秒,10毫秒 (默认)

0 -> 记录所有命令

-1 -> 不记录命令

slow-max-len 存放的最大条数
```

慢查询导致原因: value 值过大，解决办法:数据分段（更细颗粒度存放数据）

## 提高Redis处理效率

基于Jedis 的批量操作 Pipelined

```java
Jedis jedis = new Jedis("127.0.0.1", 6379);
Pipeline pipelined = jedis.pipelined();
for (String key : keys) {
    pipelined.del(key);
}

pipelined.sync();
jedis.close();

// pipelined 实际是封装过一层的指令集 ->  实际应用的还是单条指令，但是节省了网络传输开销（服务端到Redis环境的网络开销）
```