# Redis应用版

八股请看小林coding

[图解Redis介绍 | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/redis/)

##  教程连接

中文官网：

[Redis中文网](https://www.redis.net.cn/)

[超强、超详细Redis入门教程_这篇文章主要介绍了超强、超详细redis入门教程,本文详细介绍了redis数据库各个方-CSDN博客](https://blog.csdn.net/liqingtx/article/details/60330555)

[Redis 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/redis/redis-tutorial.html)





## 1.是什么

redis是一种基于内存的高性能，高并发key-value数据库，对数据的读写都在内存完成，常用于缓存，分布式锁等场景

PS：redis，mysql，linxu都是用c写的



相比其他 key - value 缓存产品有以下三个特点：

- Redis支持数据的持久化，可以将**内存**中的数据保存在**磁盘**中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis**支持数据的备份**，即master-slave模式的数据备份。



## 2.redis优势

1.**高性能，高可用，高拓展**

2.数据类型丰富

3.所有操作都是**原子性**的 （因为redis 网络IO和键值读写是**单线程**处理的）

4.丰富的特性，支持事务，数据持久化，Lua脚本，数据备份，发布订阅模式，过期删除，内存淘汰机制，多种集群方案



## 3.redis如何实现三高

三高的原因

1. 高性能：线程模型、网络 IO 模型、数据结构、持久化机制；
2. 高可用：[主从复制](https://so.csdn.net/so/search?q=主从复制&spm=1001.2101.3001.7020)、哨兵集群；
3. 高拓展：Cluster 分片集群



### **1.redis为什么高性能**

1***.基于内存操作***

Redis 是基于内存的数据库，不论读写操作都是在内存上完成的

2.***高效数据结构***

![image-20231005114008572](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231005114008572.png?lastModify=1697501626)

3.***单线程模型***

避免了线程创建，多线程上下文切换的性能消耗

避免了线程间竞争问题，不需要锁机制

4.***IO多路复用模型和性能优良的事件驱动***

Redis 采用 I/O 多路复用技术，并发处理连接。采用了 epoll + 自己实现的简单的事件框架。epoll 中的读、写、关闭、连接都转化成了事件，然后利用 epoll 的多路复用特性，绝不在 IO 上浪费一点时间。

Redis服务端，从整体上来看，其实是一个事件驱动的程序，所有的操作都以事件的方式来进行。



**单线程是否没有充分利用 CPU 资源呢？**

：因为 Redis 是基于内存的操作，使用Redis时，几乎不存在CPU成为瓶颈的情况， Redis主要受限于服务器内存和网络。（cpu读写内存很快，而且redis几乎所有操作都是简单的get，set，不存在复杂的计算操作，所以redis主要受限于内存和网络，而不是cpu） 然而，使用了单线程的处理方式，就意味着到达服务端的请求不可能被立即处理。就需要IO多路复用模型来保证单线程的资源利用率和处理效率





[Redis分片集群(详解+图)_Guo_10_Jun的博客-CSDN博客](https://blog.csdn.net/qq_45748269/article/details/121721611)

## 10.项目部分应用补充

### 10.1 补充：分页缓存

1.Zset维护分页缓存，按score排列，每次从那分页拿（但是删除和更新消耗很大，不推荐）





2.分页查询id列表，然后根据id列表到redis查询实体（更推荐

思考下用什么数据结构最合适？

这种情况维护缓存和数据库一致性可以用删除（更合理）

分页查询商品列表是典型的高并发吧？ 写入倒是挺少的，

### 10.2 缓存一致性？

更新缓存代价小就更新缓存，大就删除缓存。

并发一致性的保证：1.旁路缓存+先更新再删除 （不能完全防止，是通过数据库和缓存的操作速度差异实现的）

2.分布式锁？



### 10.3缓存穿透问题

缓存穿透问题，在商品下单时也有用？ ![image-20231002093533062](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231002093533062.png?lastModify=1697501626) 应该来说，在购物车批量下单无用（不需要直接查id，是根据用户拥有的已选中购物车下单），但直接下单接口的处理逻辑跟购物车批量下单一样，此外还需要加入缓存击穿的防范（缓存个null)

### 10.4 Redis+Lua 

first

#### 1.Lua脚本是什么？

Lua是一个高效（高并发、高性能）的轻量级脚本语言，广泛作为其他语言的嵌入脚本

#### 2.为什么redis要引入lua

主要是为了高效的实现redis的批量原子操作；

1.减少网络开销，在Lua脚本中可以把多个命令放在同一个脚本中批量运行

2.原子操作，Redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。换句话说，编写脚本的过程中无需担心会出现竞态条件。（redis原生命令不支持多条命令批量原子执行）

3.复用性，客户端发送的脚本会永远存储在Redis中，这意味着其他客户端可以复用这一脚本来完成同样的逻辑。

4.可自定义复杂逻辑，如if-else，高级运算符等。



PS：lua操作不能回滚，所以不带有原子性，但是我们可以对lua脚本全面测试，保证脚本操作不出错。

#### 3.使用lua的问题

- 当Lua脚本遇到异常时，已经执行过的逻辑是不会回滚的，所以必须对lua脚本全面测试保证脚本逻辑的健壮性。

- 在脚本编写中声明的变量全部使用`local`关键字。

- 在集群中使用Lua脚本要确保逻辑中所有的`key`分到相同机器，也就是同一个插槽(slot)中，可采用**Redis Hash Tag**技术。

- 再次重申Lua脚本一定不要包含过于耗时、过于复杂的逻辑。

  （在执行EVAL命令时，直到命令执行完毕前，其他客户端发送的命令将会阻塞。因此LUA脚本不宜编写一些过于复杂了逻辑，必须尽量保证Lua脚本的效率，否则会使其它客户端请求超时。）



#### 4.lua的语法

##### 1.数据类型

1. `nil` 空
2. `boolean` 布尔值
3. `number` 数字
4. `string` 字符串
5. `table` 表

##### 2.声明类型

声明类型非常简单，不用携带类型。

> --- 全局变量  name = 'felord.cn' --- 局部变量 local age = 18
>
> Redis脚本在实践中不要使用全局变量，局部变量效率更高



前面四种非常好理解，第五种`table`需要简单说一下，它既是数组又类似Java中的`HashMap`（字典），它是Lua中仅有的数据结构。

![image-20231003165027889](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231003165027889.png?lastModify=1697501626)

相当于一个集合，里面有一个数组，一个hashmap，你可以往里面存值，也可以放键值，这俩者不互相干扰，比如作为字典的图二，print arr【1】时返回的是nil 空值，也就是说，插入的键值不影响数组的索引，以数组索引形式取值也拿不到他



##### 3.判断

> ![image-20231003165247613](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231003165247613.png?lastModify=1697501626)

if then elseif then else end

##### 4.循环

![image-20231003165414312](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231003165414312.png?lastModify=1697501626)

ipair只遍历其中的数组，pairs还会遍历键值

ipairs中，i是数组索引，v是数组元素值



#### 5.如何应用？

##### 1.redis eval语法

**Redis Eval** 命令使用 Lua 解释器执行脚本。

**语法**

redis Eval 命令基本语法如下：

```
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...] 
```

参数说明：

- **script**： 参数是一段 Lua 5.1 脚本程序。脚本不必(也不应该)定义为一个 Lua 函数。
- **numkeys**： 用于指定键名参数的个数。
- **key [key ...]**： 从 EVAL 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用 1 为基址的形式访问( KEYS[1] ， KEYS[2] ，以此类推)。
- **arg [arg ...]**： 附加参数，在 Lua 中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)。



*实例分析*

![image-20231003163720892](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231003163720892.png?lastModify=1697501626)

如图，2是numkeys，key1，key2都是key数组里的，first，second是arg数组里的



*也可以直接通过 redis-cli --eval执行写好的lua脚本：*

redis-cli --eval /test.lua 0



##### 2.redis eval应用

通过return 返回结果，通过redis.call执行redis命令：

```shell
eval "return redis.call('keys','*')" 0
```



以下命令删除dict*格式的所有key值

> local redisKeys = redis.call('keys',KEYS[1]..'*'); for i,k in pairs(redisKeys) do  redis.call('del',k); end; return redisKeys;



一把可重入的[分布式锁](https://so.csdn.net/so/search?q=分布式锁&spm=1001.2101.3001.7020)，且支持设定锁时间。

```lua
-- 加锁脚本
-- key1：要加锁的名称 argv1：锁存活的时间ms argv2:当前线程或主机的地址
local expire_time = tonumber(ARGV[1])
if redis.call('exists', KEYS[1]) == 0 then
    -- 锁不存在，创建一把锁，存入hash类型的值
    redis.call('set', KEYS[1], 1)
    -- 设置锁的存活时间，防止死锁
    redis.call('pexpire', KEYS[1], expire_time)
    return 1
end

if redis.call('hexists', KEYS[1], ARGV[2]) == 1 then
    -- 表示是同一线程重入
    redis.call('hincrby', KEYS[1], ARGV[2], 1)
    -- 重新设置锁的过期时间
    redis.call('pexpire', KEYS[1], expire_time)
    return 1
end

-- 没抢到锁，返回锁的剩余有效时间ms
return redis.call('pttl', KEYS[1])
```

ps:HSET KEY_NAME FIELD VALUE 

```lua
-- 解锁脚本
-- 判断是当前线程持有锁，避免解了其他线程加的锁
if redis.call('hexists',KEYS[1],ARGV[2]) == 1 then
    -- 重入次数大于1，扣减次数
    if tonumber(redis.call('hget',KEYS[1],ARGV[2])) > 1 then
        return redis.call('hincrby', KEYS[1], ARGV[2], -1);
        -- 重入次数等于1，删除该锁
    else
        return redis.call('del', KEYS[1]);
    end
    -- 判断不是当前线程持有锁，返回解锁失败
else
    return 0;
end
```

[Redis分布式锁-这一篇全了解(Redission实现分布式锁完美方案)-CSDN博客](https://blog.csdn.net/asd051377305/article/details/108384490)

##### 3.java整合redis+lua

[Redis 使用lua脚本最全教程*redis lua语法*衡与墨的博客-CSDN博客](https://blog.csdn.net/le_17_4_6/article/details/117588021)

redisTemplate 执行脚本方法

```java
@Component
public class RedisUtil {
    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    /**
     * 执行 lua 脚本
     * @author hengyumo
     * @since 2021-06-05
     *
     * @param luaScript  lua 脚本
     * @param returnType 返回的结构类型
     * @param keys       KEYS
     * @param argv       ARGV
     * @param <T>        泛型
     *           
     * @return 执行的结果
     */
    public <T> T executeLuaScript(String luaScript, Class<T> returnType, String[] keys, String... argv) {
        return redisTemplate.execute(RedisScript.of(luaScript, returnType),
                new StringRedisSerializer(),
                new GenericToStringSerializer<>(returnType),
                Arrays.asList(keys),
                (Object[])argv);
    }
}
```

使用很简单，以下用上边使用过的两个脚本作为示例：

```java
    @Resource
    private RedisUtil redisUtil;

    @Test
    @SuppressWarnings("unchecked")
    public void testExecuteLuaScript() {
        String script = "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}";
        List<Object> list = (List<Object>)redisUtil.executeLuaScript(script,
                List.class, new String[] {"a", "b"}, "a", "b");
        list.forEach(x -> System.out.println(x.toString()));

        script = "for i=1,KEYS[1],1 do local k=KEYS[2]..i; redis.call('set',k,ARGV[1]);" +
                "if ARGV[2] then redis.call('expire',k,ARGV[2]) end;end;" +
                "return redis.call('keys',KEYS[2]..'*');";
        list = (List<Object>)redisUtil.executeLuaScript(script,
                List.class, new String[] {"10", "test"}, "0", "60");
        list.forEach(x -> System.out.println(x.toString()));

    }
```

![image-20231003174227720](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231003174227720.png?lastModify=1697501626)



[Redis 如何实现库存扣减操作和防止被超卖？ - 掘金 (juejin.cn)](https://juejin.cn/post/7182357365587935289)

![image-20231003174429579](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231003174429579.png?lastModify=1697501626)

分析：先查看对应key是否已存在，

if返回1说明已存在，取到库存值，并开始判断

​	if 为-1，则返回-1，结束

​	if 库存大于要扣除的库存数num，则incrby自增-num，返回incr后的库存数量，结束

​	if 前两个if都没有捕捉到，说明stock<num，返回-2，结束

如果上面三个if都没接到，那外层if结束，end，继续执行下面逻辑

else上面那个没接到，说明key目前不存在，说明对应商品的库存在缓存预热时没有加进去，返回-3，结束



好处：能够保证查询库存和扣减库存操作的原子性，保证不会多扣除redis库存

思考：多同时要扣减多个商品的库存， 如何保证原子性





## 10.5 redis最佳实践

### **key设计**

![image-20231013171156514](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231013171156514.png?lastModify=1697501626)

![image-20231013171211671](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231013171211671.png?lastModify=1697501626)

### **redis BigKey**

[Redis 大 key 要如何处理？_牛客网 (nowcoder.com)](https://www.nowcoder.com/discuss/526088159394824192?sourceSSR=search)

以下是对各个数据类型大key的描述：

- value是STRING类型，它的值超过5MB
- value是ZSET、Hash、List、Set等集合类型时，它的成员数量超过1w个

怎么处理：

1. 当vaule是string时，可以使用序列化、压缩算法将key的大小控制在合理范围内，但是序列化和反序列化都会带来更多时间上的消耗。或者将key进行拆分，一个大key分为不同的部分，记录每个部分的key，使用multiget等操作实现事务读取。
2. 当value是list/set等集合类型时，根据预估的数据规模来进行分片，不同的元素计算后分到不同的片。
3. 异步删除大key，unlink

怎么找到大key呢？

1. 通过 redis-cli --bigkeys 命令查找大key：最好在从节点或者业务低峰阶段进行扫描查询，以免影响实例正常运行该方法也存在不足，只能返回每种类型中最大的那个Key，且对于集合类型来说只统计集合元素个数多少而非实际占用的内存。
2. 使用RdbTools工具查找大keyRdbTools第三方开源工具可以用来解析Redis的RDB文件，找到其中的大key。比如下面这条命令，将大于10Kb的Key输出到一个表格文件

```bsh
rdb dump.rdb -c memory --bytes 10240 -f redis.csv
```





### 服务端优化

[Redis 性能优化的 13 条军规！史上最全 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/118532234)

等到时候再云吧，现在就说目前我们项目的甲方提供服务器和相关中间件，我们只需要把进程部署进去就行，所以redis只在自己服务器做过配置，没调过优

[项目优化之Redis调优*redis参数调优*一只Black的博客-CSDN博客](https://blog.csdn.net/weixin_43822632/article/details/123551463)



![image-20231013190047877](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231013190047877.png?lastModify=1697501626)



[线上Redis高并发性能调优实践 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/383886868)

### 缓存降级

当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。

缓存降级的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。

在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：

1. 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；
2. 警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；
3. 错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；
4. 严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。

**服务降级的目的，是为了防止Redis服务故障，导致数据库跟着一起发生雪崩问题**。因此，对于不重要的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis出现问题，不去数据库查询，而是直接返回默认值给用户。

### 补充

SWAP

[Redis常见阻塞原因总结 | JavaGuide(Java面试 + 学习指南)](https://javaguide.cn/database/redis/redis-common-blocking-problems-summary.html#swap-内存交换)

内存碎片

[Redis内存碎片详解 | JavaGuide(Java面试 + 学习指南)](https://javaguide.cn/database/redis/redis-memory-fragmentation.html#如何清理-redis-内存碎片)



## 11.Redission

[redisson使用全解——redisson官方文档+注释（上篇）_redisson官网中文-CSDN博客](https://blog.csdn.net/A_art_xiang/article/details/125525864)

[目录 · redisson/redisson Wiki · GitHub](https://github.com/redisson/redisson/wiki/目录)

[redisson简单介绍-CSDN博客](https://blog.csdn.net/qq_25582465/article/details/109309624)

[Redisson 实现分布式锁原理分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/135864820)

这里暂时只了解分布式锁，对于它更深层次的理解得先学完redis

### 1.what？

*Redisson是更适用于分布式开发的redis客户端，基于Redis、Lua和Netty建立起了成熟的分布式解决方案，提供了如分布式分布式数据结构，分布式锁等更高级的应用方案*



Redisson采用了基于NIO的Netty框架，不仅能作为Redis底层驱动客户端，具备提供对Redis各种组态形式的连接功能，对Redis命令能以同步发送、异步形式发送、异步流形式发送或管道形式发送的功能，LUA脚本执行处理，以及处理返回结果的功能，还在此基础上融入了更高级的应用方案，不但**将原生的**Redis Hash，List，Set，String，Geo，HyperLogLog等数据结构**封装为Java里**的映射（Map），列表（List），集（Set），通用对象桶（Object Bucket），地理空间对象桶（Geospatial Bucket），基数估计算法（HyperLogLog）等结构，在这基础上还提供了*分布式的多值映射（Multimap），本地缓存映射（LocalCachedMap），有序集（SortedSet），计分排序集（ScoredSortedSet），字典排序集（LexSortedSet），列队（Queue），阻塞队列（Blocking Queue），有界阻塞列队（Bounded Blocking Queue），双端队列（Deque），阻塞双端列队（Blocking Deque），阻塞公平列队（Blocking Fair Queue），延迟列队（Delayed Queue），布隆过滤器（Bloom Filter），原子整长形（AtomicLong），原子双精度浮点数（AtomicDouble），BitSet*等Redis原本没有的分布式数据结构。不仅如此，Redisson还实现了Redis文档中提到像**分布式锁Lock**这样的更高阶应用场景。事实上Redisson并没有不止步于此，在分布式锁的基础上还提供了*联锁（MultiLock），读写锁（ReadWriteLock），公平锁（Fair Lock），红锁（RedLock），信号量（Semaphore），可过期性信号量（PermitExpirableSemaphore）和闭锁（CountDownLatch）*这些实际当中对多线程高并发应用至关重要的基本部件。正是通过实现基于Redis的高阶应用方案，使**Redisson成为构建分布式系统的重要工具**。

在提供这些工具的过程当中，Redisson广泛的使用了承载于Redis订阅发布功能之上的分布式话题（Topic）功能。使得即便是在复杂的分布式环境下，Redisson的各个实例仍然具有能够保持相互沟通的能力。在以这为前提下，结合了自身独有的功能完善的分布式工具，Redisson进而提供了像分布式远程服务（Remote Service），分布式执行服务（Executor Service）和分布式调度任务服务（Scheduler Service）这样适用于不同场景的分布式服务。使得Redisson成为了一个基于Redis的Java中间件（Middleware）。

**Redisson提供了使用Redis的最简单和最便捷的方法**。Redisson的宗旨是促进使用者对Redis的关注分离（Separation of Concern），从而让使用者能够将精力更集中地放在处理业务逻辑上

在此不难看出，Redisson同其他Redis Java客户端有着很大的区别，相比之下其他客户端提供的功能还仅仅停留在作为数据库驱动层面上，比如仅针对Redis提供连接方式，发送命令和处理返回结果等。像上面这些高层次的应用则只能依靠使用者自行实现。

**Redission，Jedis，Lettuce区别**

Redisson是更高层的抽象，Jedis和Lettuce是Redis命令的封装。

Lettuce当多线程使用同一连接实例时，是线程安全的。而jedis不安全，所以要使用连接池，为每个jedis实例分配一个连接。

Redisson基于Redis、Lua和Netty建立起了成熟的分布式解决方案



### 2.redission分布式锁

#### 1.为什么用redission

为什么要用分布式锁？

在有些场景中，为了保证数据不重复，要求保证某一方法同一时刻只能被一个线程执行。在单机环境中，应用是在同一进程下的，只需要保证单进程多线程环境中的线程安全性，通过 JAVA 提供的 volatile、ReentrantLock、synchronized 以及 concurrent 并发包下一些线程安全的类等就可以做到。而在多机部署环境中，不同机器不同进程，就需要在多进程下保证线程的安全性了。因此，分布式锁应运而生。





首先要明确分布式锁需要满足的特性：

1. **互斥性**。在任意时刻，只有一个客户端能持有锁。（redis通过单线程实现）

2. **不死锁**。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。（要有超时时间）

3. **解铃还须系铃人**。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了，即**不能误解锁**。

4. **锁续期**，防止代码还没执行完锁就过期了

   



根据以上原则，我们来选择分布式锁的实现方案：

![image-20231004124317604](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231004124317604.png?lastModify=1697501626)

PS：redis+lua是可以实现锁重入和阻塞等待的。不过锁续期实现起来很复杂，而redission给我们封装好了功能完备的分布式锁实现，借助 **Redisson 的 WatchDog 机制** 能够很好的解决锁续期的问题



#### 2.redission分布式锁的实现

##### 2.1 应用代码

![image-20231004124609685](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231004124609685.png?lastModify=1697501626)



如图，操作很简便，首先构造config，然后构造redissionclient，然后用redissionclient获取所需要的对象实例进行操作就行。



##### 2.2 原理

概述：redission定义了加锁，解锁两个接口，通过lua脚本保证原子性，同时加锁解锁中用到了redis的发布订阅功能



**加锁&解锁Lua脚本**

**1、加锁Lua脚本**

```
-- 若锁不存在：则新增锁，并设置锁重入计数为1、设置锁过期时间
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('hset', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
-- 若锁存在，且唯一标识也匹配：则表明当前加锁请求为锁重入请求，故锁重入计数+1，并再次设置锁过期时间
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then
    redis.call('hincrby', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
-- 若锁存在，但唯一标识不匹配：表明锁是被其他线程占用，当前线程无权获取他人占用的锁，直接返回锁剩余过期时间
return redis.call('pttl', KEYS[1]);
```

![image-20231004154225664](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231004154225664.png?lastModify=1697501626)

![img](https://img-blog.csdnimg.cn/20191230100301140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3czNzI0MjYwOTY=,size_16,color_FFFFFF,t_70)

Q：返回nil、返回剩余过期时间有什么目的？  A：**当且仅当返回nil，才表示加锁成功；客户端需要感知加锁是否成功的结果**





**2.解锁lua脚本**

```
-- 若锁不存在：则直接广播解锁消息，并返回1
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('publish', KEYS[2], ARGV[1]);
    return 1; 
end;
 
-- 若锁存在，但唯一标识不匹配：则表明锁被其他线程占用，当前线程不允许解锁其他线程持有的锁
if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then
    return nil;
end; 
 
-- 若锁存在，且唯一标识匹配：则先将锁重入计数减1
local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); 
if (counter > 0) then 
    -- 锁重入计数减1后还大于0：表明当前线程持有的锁还有重入，不能进行锁删除操作，但可以友好地帮忙设置下过期时期
    redis.call('pexpire', KEYS[1], ARGV[2]); 
    return 0; 
else 
    -- 锁重入计数已为0：间接表明锁已释放了。直接删除掉锁，并广播解锁消息，去唤醒那些争抢过锁但还处于阻塞中的线程
    redis.call('del', KEYS[1]); 
    redis.call('publish', KEYS[2], ARGV[1]); 
    return 1;
end;
 
return nil;
```

![image-20231004165140393](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231004165140393.png?lastModify=1697501626)

![img](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy1oeHkwMjEuZGlkaXN0YXRpYy5jb20vc3RhdGljL2ttL2RvMV82eU1UdkJmcnFDdUlDR3Q2Nk1adg?x-oss-process=image/format,png)

Q1：广播解锁消息有什么用？  A：**是为了通知其他争抢锁阻塞住的线程，从阻塞中解除，并再次去争抢锁。**

Q2：返回值0、1、nil有什么不一样？  A：**当且仅当返回1，才表示当前请求真正触发了解锁Lua脚本；**



##### 2.3 加锁源码分析‘

1.**加锁源码**

```
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        // 1.尝试获取锁
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
        if (ttl == null) {
            return true;
        }

        // 申请锁的耗时如果大于等于最大等待时间，则申请锁失败.
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }

        current = System.currentTimeMillis();

        /**
         * 2.订阅锁释放事件，并通过 await 方法阻塞等待锁释放，有效的解决了无效的锁申请浪费资源的问题：
         * 基于信息量，当锁被其它资源占用时，当前线程通过 Redis 的 channel 订阅锁的释放事件，一旦锁释放会发消息通知待等待的线程进行竞争.
         *
         * 当 this.await 返回 false，说明等待时间已经超出获取锁最大等待时间，取消订阅并返回获取锁失败.
         * 当 this.await 返回 true，进入循环尝试获取锁.
         */
        RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
        // await 方法内部是用 CountDownLatch 来实现阻塞，获取 subscribe 异步执行的结果（应用了 Netty 的 Future）
        if (!subscribeFuture.await(time, TimeUnit.MILLISECONDS)) {
            if (!subscribeFuture.cancel(false)) {
                subscribeFuture.onComplete((res, e) -> {
                    if (e == null) {
                        unsubscribe(subscribeFuture, threadId);
                    }
                });
            }
            acquireFailed(threadId);
            return false;
        }

        try {
            // 计算获取锁的总耗时，如果大于等于最大等待时间，则获取锁失败.
            time -= System.currentTimeMillis() - current;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;

              }

            /**
             * 3.收到锁释放的信号后，在最大等待时间之内，循环一次接着一次的尝试获取锁
             * 获取锁成功，则立马返回 true，
             * 若在最大等待时间之内还没获取到锁，则认为获取锁失败，返回 false 结束循环
             */
            while (true) {
                long currentTime = System.currentTimeMillis();

                // 再次尝试获取锁
                ttl = tryAcquire(leaseTime, unit, threadId);
                // lock acquired
                if (ttl == null) {
                    return true;
                }
                // 超过最大等待时间则返回 false 结束循环，获取锁失败
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }

                /**
                 * 6.阻塞等待锁（通过信号量(共享锁)阻塞,等待解锁消息）：
                 */
                currentTime = System.currentTimeMillis();
                if (ttl >= 0 && ttl < time) {
                    //如果剩余时间(ttl)小于wait time ,就在 ttl 时间内，从Entry的信号量获取一个许可(除非被中断或者一直没有可用的许可)。
                    getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                } else {
                    //则就在wait time 时间范围内等待可以通过信号量
                    getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                }

                // 更新剩余的等待时间(最大等待时间-已经消耗的阻塞时间)
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }
            }
        } finally {
            // 7.无论是否获得锁,都要取消订阅解锁消息
            unsubscribe(subscribeFuture, threadId);
        }
//        return get(tryLockAsync(waitTime, leaseTime, unit));
    }
```

流程分析：

1. 尝试获取锁，返回 null 则说明加锁成功，返回一个数值，则说明已经存在该锁，ttl 为锁的剩余存活时间。
2. 如果此时客户端 2 进程获取锁失败，那么使用客户端 2 的线程 id（其实本质上就是进程 id）通过 Redis 的 channel 订阅锁释放的事件，。如果等待的过程中一直未等到锁的释放事件通知，当超过最大等待时间则获取锁失败，返回 false，也就是第 **39** 行代码。如果等到了锁的释放事件的通知，则开始进入一个不断重试获取锁的循环。
3. 循环中每次都先试着获取锁，并得到已存在的锁的剩余存活时间。如果在重试中拿到了锁，则直接返回。如果锁当前还是被占用的，那么等待释放锁的消息，具体实现使用了 JDK 的信号量 Semaphore 来阻塞线程，当锁释放并发布释放锁的消息后，信号量的 `release()` 方法会被调用，此时被信号量阻塞的等待队列中的一个线程就可以继续尝试获取锁了。

省流：先尝试获锁，如果获锁失败就订阅锁释放时间，当前线程通过countdownlatch阻塞等待，如果等到超时了就返回失败并且取消订阅，如果等到事件了，就进入循环开始竞争获锁，循环里每次先尝试获锁，如果没获取到就用信号量阻塞等待下一次锁释放，循环重试时如果超时了就返回失败。



> 特别注意：以上过程存在一个细节，这里有必要说明一下，也是分布式锁的一个关键点：当锁正在被占用时，等待获取锁的进程并不是通过一个 `while(true)` 死循环去获取锁，而是利用了 Redis 的发布订阅机制,通过 await 方法阻塞等待锁的进程，有效的解决了**无效的锁申请浪费资源的问题**。



##### 2.4 锁续期

Redisson 提供了一个续期机制， 只要客户端 1 一旦加锁成功，就会启动一个 Watch Dog。

如下是加锁方法最终调用的方法

```java
    private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, final long threadId) {
        if (leaseTime != -1L) {
            // 设定了固定有效期的
            return this.tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
        } else {
            // 没有设定有效期的，这里启动了一个守护线程对锁续期
            RFuture<Long> ttlRemainingFuture = this.tryLockInnerAsync(this.commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
            ttlRemainingFuture.addListener(new FutureListener<Long>() {
                public void operationComplete(Future<Long> future) throws Exception {
                    if (future.isSuccess()) {
                        Long ttlRemaining = (Long)future.getNow();
                        if (ttlRemaining == null) {
                            RedissonLock.this.scheduleExpirationRenewal(threadId);
                        }
                    }
                }
            });
            return ttlRemainingFuture;
        }
    }
```

注意：从以上源码我们看到 `leaseTime` 必须是 -1 才会开启 Watch Dog 机制，也就是如果你想**开启 Watch Dog** 机制**必须使用默认的加锁时间**为 30s。如果你自己自定义时间，超过这个时间，锁就会自定释放，并不会延长。



```java
private void scheduleExpirationRenewal(final long threadId) {
    if (!expirationRenewalMap.containsKey(this.getEntryName())) {
        Timeout task = this.commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
            public void run(Timeout timeout) throws Exception {
                // 执行lua 进行续期
                RFuture<Boolean> future = RedissonLock.this.renewExpirationAsync(threadId);
                future.addListener(new FutureListener<Boolean>() {
                    public void operationComplete(Future<Boolean> future) throws Exception {
                        RedissonLock.expirationRenewalMap.remove(RedissonLock.this.getEntryName());
                        if (!future.isSuccess()) {
                            RedissonLock.log.error("Can't update lock " + RedissonLock.this.getName() + " expiration", future.cause());
                        } else {
                            if ((Boolean)future.getNow()) {
                                RedissonLock.this.scheduleExpirationRenewal(threadId);
                            }
                        }
                    }
                });
            }
            // 每隔internalLockLeaseTime/3 = 10秒检查一次
        }, this.internalLockLeaseTime / 3L, TimeUnit.MILLISECONDS);
        if (expirationRenewalMap.putIfAbsent(this.getEntryName(), new RedissonLock.ExpirationEntry(threadId, task)) != null) {
            task.cancel();
        }
    }
}
```

Watch Dog 机制其实就是一个后台定时任务线程，获取锁成功之后，会将持有锁的线程放入到一个 `RedissonLock.EXPIRATION_RENEWAL_MAP`里面，然后每隔 10 秒 `（internalLockLeaseTime / 3）` 检查一下，如果客户端 1 还持有锁 key（判断客户端是否还持有 key，其实就是遍历 `EXPIRATION_RENEWAL_MAP` 里面线程 id 然后根据线程 id 去 Redis 中查，如果存在就会延长 key 的时间），那么就会不断的延长锁 key 的生存时间。



![image-20231004174818683](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231004174818683.png?lastModify=1697501626)

释放锁时会删除EXPIRATION_RENEWAL_MAP里面的对应线程数据（该jvm正在执行的其他锁也会往这个map里放线程，这个map应该是公用的？守护线程倒不是公用的，每一个需要续期的锁都会开一个守护线程）



PS：到这里看门狗的具体实现也就清楚了，无非是后台起一个定时任务的线程，每隔一定时间对该锁进行续命，延长锁的时间，很多人肯定好奇，那延长锁的次数是有限制的吗？难道无限进行续命吗，假设业务一直没执行完，难道锁一直不释放吗？起初我也有这样的疑问，但是想了想，实际业务中也不能发生这样的情况，除非是代码bug，或者陷入了死循环，这种情况直接抛异常然后finally unlock就行

##### 2.5 释放锁源码分析

```java
@Override
public RFuture<Void> unlockAsync(long threadId) {
    RPromise<Void> result = new RedissonPromise<Void>();
    // 1. 异步释放锁
    RFuture<Boolean> future = unlockInnerAsync(threadId);
    // 取消 Watch Dog 机制
    future.onComplete((opStatus, e) -> {
        cancelExpirationRenewal(threadId);

        if (e != null) {
            result.tryFailure(e);
            return;
        }

        if (opStatus == null) {
            IllegalMonitorStateException cause = new IllegalMonitorStateException("attempt to unlock lock, not locked by current thread by node id: "
                    + id + " thread-id: " + threadId);
            result.tryFailure(cause);
            return;
        }

        result.trySuccess(null);
    });

    return result;
}

protected RFuture<Boolean> unlockInnerAsync(long threadId) {
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            // 判断锁 key 是否存在
            "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                "return nil;" +
            "end; " +
            // 将该客户端对应的锁的 hash 结构的 value 值递减为 0 后再进行删除
            // 然后再向通道名为 redisson_lock__channel publish 一条 UNLOCK_MESSAGE 信息
            "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
            "if (counter > 0) then " +
                "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                "return 0; " +
            "else " +
                "redis.call('del', KEYS[1]); " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; "+
            "end; " +
            "return nil;",
            Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.UNLOCK_MESSAGE, internalLockLeaseTime, getLockName(threadId));
}
```

从以上代码来看，释放锁的步骤主要分三步：

1. 删除锁（这里注意可重入锁，在上面的脚本中有详细分析）。
2. 广播释放锁的消息，通知阻塞等待的进程（向通道名为 `redisson_lock__channel` publish 一条 `UNLOCK_MESSAGE` 信息）。
3. 取消 Watch Dog 机制，即将 `RedissonLock.EXPIRATION_RENEWAL_MAP` 里面的线程 id 删除，并且 cancel 掉 Netty 的那个定时任务线程

12步在lua脚本完成，第三步在jvm中完成



#### 3.优缺点

1.**优点**

​	1.通过watchDog解决了锁续期问题

​	2.封装了的分布式可重入锁的实现，满足了分布式锁需要满足的四个特性。

​	3.在阻塞等待申请锁资源的实现做了优化，利用redis的发布订阅机制，减少了无效的锁申请，提升了资源利用率

2.**缺点**

RedissonLock 同样没有解决 节点挂掉的时候，存在丢失锁的风险的问题。而现实情况是有一些场景无法容忍的，所以 Redisson 提供了实现了redlock算法的 RedissonRedLock，RedissonRedLock 真正解决了单点失败的问题，代价是需要额外的为 RedissonRedLock 搭建Redis环境。

所以，如果业务场景可以容忍这种小概率的错误，则推荐使用 RedissonLock， 如果无法容忍，则推荐使用 RedissonRedLock。



## 12.Pipeline



[「进击Redis」十一、Redis Pipeline这一篇就够了 - 掘金 (juejin.cn)](https://juejin.cn/post/6904433426560974856)

数据刷新到redis,使用管道批量刷新，减少连接获取，资源关闭的开销。 同时因为redis服务是单线程的，需要控制管道的命令量不要过分多，因为管道命令过多执行可能会导致redis线程阻塞，导致其他线程操作redis超时。所以需要控制管道的命令量，并且适当扩大redis的超时时间.  可以改为60s或者100秒应该足够了

#### 为什么使用**pipeline**？

![image-20230923201248553](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20230923201248553.png?lastModify=1697501626)



redis是一个高性能的单线程的key-value数据库。它的执行过程为：

（1）发送命令－〉（2）命令排队－〉（3）命令执行－〉（4）返回结果



Redis也支持Pipeline模式，不同于Ping-pong模式，Pipeline模式类似流水线的工作模式：客户端发送一个命令后无需等待执行结果，会继续发送其他命令；在全部请求发送完毕后，客户端关闭请求，开始接收响应，收到执行结果后再与之前发送的命令按顺序进行一一匹配。在Pipeline模式的具体实现中，大部分Redis客户端采用批处理的方式，即一次发送多个命令，在接收完所有命令执行结果后再返回给上层业务。

如果我们使用redis进行批量插入数据，正常情况下相当于将以上四个步骤批量执行N次。（1）和（4）称为Round Trip Time（RTT，往返时间）。因此我们需要批量操作，把原本需要执行N次的RTT（网络请求与响应时间）变成一次，这样就减少了网络IO的消耗，提高了性能

虽然 Redis 已经提供了像 `mget` 、`mset` 这种批量的命令，但是有些操作如果redis不支持批量的操作，一条一条的执行命令，这时候就需要pipeline，pipeline为redis提供了一系列批量操作的功能。

**它能将一组 Redis 命令进行组装，通过一次传输给 Redis 并返回结果集**

![image-20230922104323811](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20230922104323811.png?lastModify=1697501626)

#### **how**？

`redis-cli` 的 `--pipe`参数实际上就是使用 Pipeline 机制

或者使用客户端（比如jedis）操作管道

![image-20230922104548194](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20230922104548194.png?lastModify=1697501626)

[SpringBoot整合RedisTemplate利用pipeline进行高效率批量操作_Jeremy_Lee123的博客-CSDN博客](https://blog.csdn.net/lixinkuan328/article/details/109656206)

#### **原理**

实现`Pipeline` 功能，需要客户端和服务器端的支持。



客户端new 一个管道对象，每次向管道里放命令，积累到最后一起传给redis服务端，redis服务端与客户端维持一个TCP链接，单线程依次执行命令，同时把每条命令的处理结果缓存到Socket接收缓冲区，最后所有命令都处理完一起打包返回。 （ps：所以不要批量操作太多数据，一是阻塞redis线程，容易使其他线程操作redis超时，二是数据太多存储处理结果时会冲爆缓冲区）

pipeline只是在一个TCP链接里批量发命令而已，并没有保证原子性

因为：

1. 这是纯客户端行为，服务端无感知，也没有进行对应的特殊处理。
2. 不阻塞服务端执行其他客户端的指令，即没有串行化

#### **批量命令、Pipeline 对比**

1. 原生批量命令是原子的，Pipeline 是非原子的。
2. 原生批量命令是一个命令对应多个 key，Pipeline 支持多个命令。
3. 原生批量命令是 Redis 服务端支持实现的，而 Pipeline 是客户端实现

#### **使用场景**

`Peline`是 Redis 的一个提高吞吐量的机制，适用于**多 key 读写场景**，比如同时读取多个`key` 的`value`，或者更新多个`key`的`value`，并且允许一定比例的**写入失败**、**实时性**也没那么高，那么这种场景就可以使用了。比如 10000 条一下进入 redis，可能失败了 2 条无所谓，后期有补偿机制就行了，像短信群发这种场景，这时候用 pipeline 最好了。

（提前把热key放进去就行。其他并发不高的商品，缓存查不到就现场会写）



#### **注意问题**

1. `Pipeline`是非原子的，会出现原子性问题。
2. `Pipeline`中包含的命令不要包含过多。
3. `Pipeline`每次只能作用在一个 Redis 节点上。
4. `Pipeline` 不支持事务，因为命令是一条一条执行的。

## 13.事务

[Redis Pipeline &事务&Lua脚本的区别 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/518223284)

![image-20231013183547695](file:///C:/Users/Azusa/AppData/Roaming/Typora/typora-user-images/image-20231013183547695.png?lastModify=1697501626)

### Redis事务的概念：

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令

### Redis事务支持隔离性

Redis 是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止。因此，Redis 的事务是总是带有隔离性的。

### Redis不保证[原子性](https://so.csdn.net/so/search?q=原子性&spm=1001.2101.3001.7020)：

Redis中，单条命令是原子性执行的，但`事务不保证原子性，且没有回滚`。事务中任意命令执行失败，其余的命令仍会被执行。

***根据[原子性的定义](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Atomicity_(database_systems))：一个事务内，多个操作要么全部执行成功，要么全部不执行。那么 Pipeline、事务、Lua 都不具备原子性，因为单条指令的失败都不会阻碍其他执行的实际执行, 并没有"回滚"概念。***

### Redis事务的三个阶段：

（1）开始事务

（2）命令入队

（3）执行事务

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723212756496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzUyMDQ1MA==,size_16,color_FFFFFF,t_70)

多个命令会被人队到事务队列中， 然后按先进先出(FIFO)的顺序执行。

### **WATCH命令**

`WATCH`命令可以监控一个或多个键，一旦其中有一个键被修改，之后的事务就不会执行（类似于乐观锁）。执行`EXEC`命令之后，就会自动取消监控。