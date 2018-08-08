## Redis深入了解

### 1. 数据类型

- string : 最常用的简单数据类型，主要的命令有 `get` ， `set` ，`del` ， `exists` 等等，除了单个操作以外也提供批量操作，如 `mget` ，`mset` 。
- list : `list` 是一个链表，可以通过 `lpush` 和 `rpush` 来控制左进还是右进，弹出也分位左出的 `lpop` 和右出的 `rpop` ，通过这个属性可以利用 `redis` 做简单的消息队列，使用左进右出或者右进左出模式都可以，`list` 集合可以通过 `lrange` 来获取元素，方法是 `lrange key start_index end_index` ，如果是 `lrange key 0 -1` 就代表列出所有元素。
- hash : 对应 `java` 中的 `HashMap` ，常用的命令有 `hset` ，`hget` ，对应的也有批量的 `hmset` 和 `hmget` ，另外 `hgetall` 可以展示 `hash` 中所有的键值对。
- set : 对应 `java` 中的 `HashSet`，内部元素是唯一且无序的。常用命令是 `sadd` ，`smembers` ，`sismember` ，`scard`（获取set元素数量），`spop` 等等。
- zset : `zset` 是个有序的 `set` 集合，每个元素都有特定的 `score` 属性来排序，添加格式为 `zadd key score value` 。常用命令有 `zadd` ，`zrange`（遍历，正向逆向都有），`zrank`（获取排名），`zscore`，`zrem`（删除）。

### 2. 分布式锁
分布式锁本质上就是 `setnx` 了一个值，另外一个线程获取锁时就是再 `setnx` 了一次，如果这个值已经存在就什么都不做返回0表示已存在，也就是获取锁失败，直到这个锁被 `del` 。为了防止程序报错导致死锁，都会设置一个过期时间，防止设置锁和设置过期时间之间出错，要保证加锁和设置时间是一步完成，是一个原子操作，命令如下：`set lock true ex 5 nx，ex` 表示过期时间。

### 3. 位图
实际开发过程中有时会需要记录大量的bool类型的数据，比如一年中用户的签到情况，每一天是否签到，这时间使用数据库的话成本会很大且销量地下，这种情况就可以使用位图。使用方法有点类似zset，添加方法：`setbit key 1 0 2 1 ...`，这样的命令可以表示第一天没签到（0），第二天签到了（1）。

### 4. HyperLogLog
当遇到大型网站需要统计浏览用户人数的需求的时候，可以使用 `HyperLogLog` ，它比使用 `set` 集合更加的节约空间，但是缺点是有些许误差，误差率在0.81%左右，如果没有人数精确的要求使用它是更效率的原则。使用方法： `pfadd` , `pfcount` 等等，用法类似 `set` 。

### 5. GeoHash
GeoHash可以根据两个经纬度来计算具体的距离，像附近的人这种功能开发时就会需要用到它。添加命令是 `geoadd`（单个或批量都可以），`geodist` 可以计算距离，还可以指定距离单位是米，公里等等。

### 6. 主从同步
通过设置从 `redis` 的 `redis.conf` 中的 `slaveof:ip port` 参数即可指定复制哪个主 `redis`（如果主 `redis` 设置了密码，也需要配置密码 `masterauth` ），从 `redis` 是只读的。

### 7. Sentinel哨兵
一般项目中配置主从 `redis` 还需要使用哨兵功能，没有哨兵的话一旦主 `redis` 挂了，这时系统再往 `redis` 中写数据时就会报错，这种情况多个 `redis` 也于事无补。有了 `Sentinel` 就可以解决这个问题，一般是一个 `redis` 一个哨兵，在主 `redis` 挂了的时候，几个哨兵检测到就会推举出新的主 `redis` ，这个新的主 `redis` 会完全取代之前的主 `redis` ，这样就可以保证系统的正常运行。
