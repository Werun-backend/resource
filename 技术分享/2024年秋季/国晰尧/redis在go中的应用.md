# 技术分享

# 安装与连接

## 安装

`go get [github.com/go-redis/redis/v8](http://github.com/go-redis/redis/v8)`

## 连接

### 单机连接

```c
rdb := redis.NewClient(&redis.Options{
    Addr:     "localhost:6379",
    Password: "", // no password set
    DB:       0,  // use default DB
})
```

### 主从复制

```c
func InitRedis() {
	// 初始化主节点
	masterRdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:6380", // 主节点地址
		Password: "",
		DB:       0,
	})

	// 测试主节点连接
	pong, err := masterRdb.Ping(Ctx).Result()
	if err != nil {
		log.Fatalf("could not connect to Redis master: %v", err)
	}
	log.Printf("Redis master connected: %v", pong)

	// 初始化从节点
	slaveRdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:6381", // 从节点地址
		Password: "",
		DB:       0,
	})
	}
```

### 哨兵模式

```c
// InitRedis 使用哨兵模式初始化 Redis 客户端
func InitRedis() {
	masterRdb = redis.NewFailoverClient(&redis.FailoverOptions{
		MasterName:    "mymaster",                                                        // 哨兵监控的主节点名称
		SentinelAddrs: []string{"127.0.0.1:26379", "127.0.0.1:26380", "127.0.0.1:26381"}, // 哨兵地址列表
		Password:      "",                                                                // Redis 密码
		DB:            0,                                                                 // 数据库编号
	})
```

| **特性** | **单机模式（Standalone）** | **主从复制（Master-Slave）** | **哨兵模式（Sentinel）** |
| --- | --- | --- | --- |
| **架构** | 只有一个 Redis 实例 | 一个主节点和多个从节点 | 哨兵监控多个 Redis 实例，自动故障转移 |
| **故障转移** | 无故障转移机制 | 无自动故障转移，需要手动切换 | 自动故障转移，哨兵自动切换主节点 |
| **高可用性** | 无 | 只有从节点备份主节点，手动恢复 | 自动高可用性，支持主节点故障恢复 |
| **读写分离** | 不支持读写分离 | 支持读写分离，从节点负责读操作 | 支持读写分离，且可以通过哨兵自动切换主节点 |
| **适用场景** | 小型应用，开发环境 | 中型应用，读操作较多 | 大型应用，要求高可用性和高吞吐量 |

<aside>
💡

- 单机模式：只有一个 Redis 实例运行，所有的读写操作都在这个实例上进行。
- 主从复制模式：主节点负责处理写操作，数据会异步同步到从节点。从节点处理读操作，减轻主节点负担。
- 哨兵模式：监控 Redis 主节点和从节点的健康状态。如果主节点宕机，哨兵会自动将一个从节点提升为新的主节点
</aside>

## 测试

`ls -ah /home/guoxiyao`

`ps aux | grep redis`     //检查正在运行的进程

`ps aux | grep redis-server` //查看redis服务端口

sudo kill 66

sudo kill -9 66

### 检查配置文件

```c
nano /home/guoxiyao/redis-master.conf
nano /home/guoxiyao/redis-slave.conf
```

### 启动主从节点

```c
redis-server /home/guoxiyao/redis-master.conf
redis-server /home/guoxiyao/redis-slave.conf
redis-server /home/guoxiyao/redis-slave2.conf
```

### 检查主从节点的复制状态

```c
redis-cli -p 6380 info replication  # 检查主节点
redis-cli -p 6381 info replication  # 检查从节点1
redis-cli -p 6382 info replication  # 检查从节点2
```

### 启动哨兵进程

```c
redis-sentinel /home/guoxiyao/sentinel-26379.conf
redis-sentinel /home/guoxiyao/sentinel-26380.conf
redis-sentinel /home/guoxiyao/sentinel-26381.conf
```

### 检查哨兵状态

```c
redis-cli -p 26379 sentinel masters
redis-cli -p 26380 sentinel masters
redis-cli -p 26381 sentinel masters
```

### 测试故障转移

```c
//停止redis主节点
redis-cli -p 6380 shutdown
//查看Sentinel输出
redis-cli -p 26379 sentinel masters
```

### 检查连接问题

```c
redis-cli -p 6379
```

### 启动redis端口

```c
redis-server --port 6380//指定端口
redis-server /path/to/redis.conf//使用配置文件启动
```

# 缓存

## 概念

将频繁访问的数据保存在内存中，提升响应速度

## 流程

- 获取查询参数
- 生成缓存键：唯一的字符串，它基于查询参数生成

```go
cacheKey := fmt.Sprintf("diaries:%s:%s:%s:%s:%d:%d", queryParams.TagID, queryParams.Content, queryParams.StartTime, queryParams.EndTime, queryParams.Page, queryParams.PageSize)
```

- 检查缓存是否存在： 使用 **`rdb.Get`** 从 Redis 获取缓存，如果缓存命中（没有报错），则直接返回缓存数据，避免执行数据库查询。

```go
cachedData, err := rdb.Get(ctx, cacheKey).Result()
if err == nil {
    // 如果缓存中有数据，直接返回缓存数据
    response.WriteJSON(c, response.NewResponse(200, cachedData, "success"))
    return
}
```

- 查询数据库并缓存结果：如果缓存中没有数据，执行数据库查询，并将查询结果缓存

```go

//通过 Paginate 函数进行分页查询，获取数据库结果。
paginationResult, err := service.Paginate(ctrl.DB, queryParams.Page, queryParams.PageSize, &models.Diary{}, modifier)
//将查询结果存入 Redis 中，设置缓存过期时间（这里是 10 分钟）。这样，下次相同的查询会从缓存中获取，减少数据库的负载。
err = rdb.Set(ctx, cacheKey, paginationResult, 10*time.Minute).Err()

```

### 缓存过期与缓存失效

**原理：** Redis 会自动管理缓存的生命周期，在缓存过期后，数据会被清除。当用户再次访问时，Redis 不再返回过期的缓存数据，而是通过查询数据库获取新的数据。避免占用大量内存，同时确保缓存能在需要时刷新

# 消息队列

## 概念

### 消息队列

异步通信的机制。

- **异步处理**：发布消息的系统（发送方）不需要等待接收方处理完消息后再继续操作，从而减少了系统间的耦合。
- **消息持久化**：消息可以被持久化在队列中，直到被消费处理，确保消息不会丢失。

### **发布/订阅（Pub/Sub）模式**

**发布者**将消息发送到一个**频道**，而**订阅者**订阅这个频道以接收消息。

- **发布者**：发布消息到特定的频道（例如 "notifications"）。
- **订阅者**：监听一个或多个频道，一旦消息发布，订阅者就能接收到。
- **消息中间件**：负责存储和转发消息，Redis 就是我们使用的消息中间件。

## 实现

- 定义通知消息结构
- **发布通知：`PublishNotification`** 函数将通知消息发布到 Redis 中的一个特定频道 **`"notifications"`**。

```go
func PublishNotification(client *redis.Client, message NotificationMessage) error {
	msg, err := json.Marshal(message)
	if err != nil {
		return err
	}
	return client.Publish(ctx, "notifications", msg).Err()
}
//将 NotificationMessage 对象序列化为 JSON 格式，然后通过 client.Publish 发布到 "notifications" 频道。
```

- **订阅通知：`SubscribeNotifications`** 函数用来订阅 Redis 中的 **`"notifications"`** 频道，并使用回调函数处理接收到的通知。

```go
func SubscribeNotifications(client *redis.Client, handler func(NotificationMessage)) {
	pubsub := client.Subscribe(ctx, "notifications")
	//client.Subscribe创建与redis的订阅连接，返回一个 pubsub 对象。通过这个对象，可以访问频道的消息流。
	ch := pubsub.Channel()
	//pubsub.Channel() 返回一个消息通道（channel），它是一个可以读取消息的 Go channel。每当 Redis 推送一条消息到订阅的频道时，该消息会被推送到 ch 通道中，for 循环会从这个通道中取出消息并处理。
	for msg := range ch {
		var message NotificationMessage
		if err := json.Unmarshal([]byte(msg.Payload), &message); err == nil {//消息处理
			handler(message)
		}
	}
}
//通过 client.Subscribe 订阅 "notifications" 频道，然后进入一个 for 循环，监听并处理接收到的消息
```

# 分布式锁

## 概念

解决分布式系统中，多个节点之间共享资源访问时的同步问题。确保多个分布式系统中的客户端或进程能按照某个顺序访问共享资源。

## 实现

### **`LikeDiary` 控制器**

这个函数实现了日记的点赞操作，并使用 Redis 分布式锁来确保同一用户不能重复点赞。

- **获取锁**

```go
lockKey := "like:" + strconv.Itoa(diaryID) + ":" + strconv.Itoa(int(userID.(uint)))
lock := redis.NewDistributedLock(ctrl.RedisClient, lockKey, 10*time.Second)
acquired, err := lock.Acquire()
```

**`lockKey`**：每个用户对每个日记的点赞操作都是独立的，所以锁的key由 **`diaryID`** 和 **`userID`** 组成。

**`Acquire()`**：尝试获取锁，超时时间为10秒。锁的TTL设置为10秒，防止死锁。

- **执行操作**： 如果成功获取到锁，执行点赞操作

```go
if acquired {
    // 执行点赞逻辑
    err := service.LikeDiary(userID.(uint), uint(diaryID))
}
```

- **释放锁**： 当操作完成后，释放锁，确保其他请求可以访问

```go
defer func(lock *redis.DistributedLock) {
    err := lock.Release()
    if err != nil {
        log.Printf("Failed to release lock: %v", err)
    }
}(lock)
```

### **`DistributedLock` 实现**

在 **`redis`** 包中，**`DistributedLock`** 的实现通过 Redis 客户端的 **`SETNX`** 命令来实现获取锁和释放锁

- **获取锁**

```go
success, err := l.Client.SetNX(ctx, l.Key, l.Value, l.TTL).Result()
//SetNX 会尝试设置锁的值，成功则返回 true，否则返回 false
```

- **释放锁**： 使用 Lua 脚本来保证锁是由持有者释放的，避免误删

```go
script := `if redis.call("get", KEYS[1]) == ARGV[1] then return redis.call("del", KEYS[1]) else return 0 end`
result, err := l.Client.Eval(ctx, script, []string{l.Key}, l.Value).Result()
//如果 Redis 中的值和当前客户端的值匹配，则删除锁。
//否则，返回错误，表示锁不是当前客户端持有的。
```

### **死锁问题**：

通过设置锁的 TTL 来避免死锁问题。若客户端在处理请求时失败或崩溃，TTL 超时后，其他客户端可以重新获取锁。

### **锁的释放失败**：

使用 Lua 脚本保证锁的原子性操作，以避免误删锁。

### 单体项目中使用

- 数据库唯一约束、
- 业务逻辑判断、
- 乐观/悲观锁
- Redis 缓存