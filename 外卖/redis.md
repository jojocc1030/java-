### 2. Redis数据类型

### 2.1 五种常用数据类型介绍

Redis存储的是**key-value**结构的数据，其中key是字符串类型，value有5种常用的数据类型：

- 字符串(string)：普通字符串，Redis中最简单的数据类型
- 哈希(hash)：也叫散列，类似于Java中的HashMap结构  (存储对象)
- 列表(list)：按照插入顺序排序，可以有重复元素，类似于Java中的LinkedList
- 集合(set)：无序集合，没有重复元素，类似于Java中的``HashSet``
- 有序集合(sorted set/zset)：集合中每个元素关联一个分数(score)，根据分数升序排序，没有重复元素   **（应用:排行榜）**
![[Pasted image 20240318130603.png]]
### 2.2 Spring Data Redis

使用**Spring Data Redis**作为redis在java的客户端


### Spring Cache

Spring Cache 是一个框架，实现了**基于注解的缓存功能**，只需要简单地加一个注解，就能实现缓存功能。
目的是为了简化缓存代码开发。
![[Pasted image 20240320141415.png]]
@CachePut 和 @Cacheable 区别： @CachePut 只是放； @Cacheable 如果没有就放，有的话就取

```java
@Cacheable(cacheNames = "setmealCache",key = "#categoryId") //key: setmealCache::100
```
```java
@CacheEvict(cacheNames = "setmealCache",key = "#setmealDTO.categoryId")

```

#### 在controller 层的方法上加上@CacheEvict(cacheNames = "userCache",allEntries = true) 进行缓存删除，是先更新了数据库，还是先删除了缓存？
在Spring框架中，当你在Controller层或任何Spring管理的Bean的方法上使用`@CacheEvict`注解时，该方法的执行行为（包括它是在方法执行之前还是之后清除缓存）取决于`@CacheEvict`注解的`beforeInvocation`属性的设置。

- 如果`beforeInvocation`设置为`false`（默认值），则缓存清除操作会在方法成功执行**之后**进行。这意味着，如果方法中有任何异常导致方法失败，缓存清除操作将不会执行。**这意味着默认是先更新数据库再删除缓存。**
- 如果`beforeInvocation`设置为`true`，则缓存清除操作会在方法执行**之前**进行。这样即使方法执行过程中发生异常，缓存也已经被清除。

因此，是否是“先更新数据库再删除缓存”还是“先删除缓存再更新数据库”，取决于你如何设置`@CacheEvict`注解的`beforeInvocation`属性，以及你的方法具体是如何实现数据库更新操作的。




## Redis实现分布式锁
https://blog.csdn.net/m0_71777195/article/details/126419861

`set key value px milliseconds nx`
- **setnx + 过期时间解决**：当程序执行到try代码块中某个位置服务宕机或者服务重新发布，也即是占用锁的业务逻辑宕机后，无法释放锁。
- value唯一性解决：
	**执行业务逻辑所需要的时间大于锁的过期时间**问题。当前线程处理 执行业务逻辑所需要的时间大于锁的过期时间 ，这时候锁会自动释放，又会被其他线程抢占到锁，当前线程执行完之后，会把其他线程抢占到的锁误释放。然而，为什么会误打开别的线程的锁呢？因为锁的唯一性，每个锁的编号都是123456，线程只认锁的编号，看见编号为123456的就开，结果把别人的锁打开了，这种情况就是锁 **没确保唯一性**导致的。
	
	解决上述问题其实也很简单，让每个线程加的锁时给Redis设置一个唯一id的value，每次释放锁的时候先判断一下线程的唯一id与Redis 存的值是否相同，若相同即可释放锁。**redisson的value是怎样保证value的唯一性呢？**答案是[UUID](https://so.csdn.net/so/search?q=UUID&spm=1001.2101.3001.7020)+threadId**。
- **锁续命**：
	分布式锁的key过期时间不管设置成多少都不合适，比如设置为10秒，如果业务代码执行链路较长，亦或是代码存在慢SQL、数据查询量大的情况，那么过期时间就不好设置，那么这里有没有什么更好的方案呢？答案是有的：锁续命。

	那么锁续命方案的原来就在于当线程加锁成功时，会开一个分线程，取锁过期时间的1/3时间点定时执行任务，每10s判断一次锁是否存在(即Redis的key)，若锁还存在那么就直接重新设置锁的过期时间，若锁已经不存在了那么就直接结束当前的分线程。
### redission  
上面说的续命锁看起来简单，但是实际上实现还是有一定的难度的，于是类似 Redission 开源框架已经帮我们实现好了，所以不需要再重复造轮子自己去写一个分布式锁了，下面会拿Redission框架举例，学习一下Redission分布式锁的设计思想。
#### Redission分布式锁的实现原理

Redission实现分布式锁的原理流程图如下图所示，当线程一加锁成功获取到锁，并开启执行业务代码时，Redission框架会开启一个后台线程（所谓的Watch Dog看门狗），每隔锁过期的1/3时间去定时判断一次是否还持有锁（Redis的key是否还存在），若不持有那么久执行结束当前的后台线程，若还持有锁，那么久重新设置锁的过期时间，当线程一加锁成功后，那么线程二就自然获取不到锁，此时线程二就会做类似CAS的自旋操作，一直等待线程一释放锁之后才能加锁成功。

另外，Redison底层实现分布式锁时使用了大量的lua脚本保证了其加锁操作的各种原子性。Redison实现分布式锁使用lua脚本的好处主要是能保证Redis的操作是**原子性**的，Redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。

![[Pasted image 20240321172056.png]]
![[Pasted image 20240325122141.png]]
使用**redlock**使所有主从节点都获得锁，以防出现只有主节点获得锁后宕机主从节点锁信息同步不一致问题。
![[Pasted image 20240325122526.png]]





## Redis和持久化数据库层数据不一致问题的解决方式
### 删除缓存 or 更新缓存？
删除。因为缓存的**更新成本**比删除高得多，而且删除缓存操作简单，副作用只是增加了一次**cache miss**

### 先操作数据库 后操作缓存
#### 数据不一致
![[Pasted image 20240326224445.png]]
**更推荐**：只会造成短时间内的数据不一致，但可以保证最终一致性。

###  先操作缓存 后操作数据库


#### 数据不一致![[Pasted image 20240326223243.png]]

会出现数据库中存的是老数据，数据库存的是新数据情况，后续查询请求就一直在读老数据（脏读）。
解决方式：**延迟双删**  （eg. 500毫秒）在更新数据库后再次删除redis中的内容，虽然不能保证所有线程都可以读到新数据（图中线程2依然会读到脏数据，在延迟的500ms内其他线程读到的都是脏数据），但是可以保证后续线程读到的是新数据。

### 延迟双删 和 先更新数据库再删除缓存 的 两种做法对比：
![[Pasted image 20240326231438.png]]
### 删除重试
![[Pasted image 20240326224740.png]]
删除redis失败时，需要进行重试操作，把重试操作放到MQ里；系统监听当删除失败的消息后执行重试操作。**（异步方式）**
#### cannal解耦
![[Pasted image 20240326225253.png]]
cannal客户端（SpringBoot）可以监听Mysql的变化（通过binlog日志），发现mysql发生变化后会执行删除redis的操作。
## 布隆过滤器 #布隆过滤器
![[Pasted image 20240325115526.png]]
- 底层数据结构是bitmap，使用bit存储判断海量数据是否存在。
- 哈希碰撞：使用多种哈希算法解决哈希碰撞问题。只是减小误判几率，不能做到绝对不误判。**不存在一定不存在，存在不一定存在。**
- 解决误判问题：1.增加位数长度；2.增加hash个数
	![[Pasted image 20240325120245.png]]
- 开发实现：
使用redisson组件进行开发：参数值{预计的插入数量，可接受的容错率}![[Pasted image 20240325120501.png]]


### 乐观锁和悲观锁

### [Redis 为什么这么快？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-为什么这么快)
### [说一下 Redis 和 Memcached 的区别和共同点](https://javaguide.cn/database/redis/redis-questions-01.html#说一下-redis-和-memcached-的区别和共同点)
数据类型
数据持久化
集群模式支持
线程模型：Memcached:多线程，非阻塞IO复用；redis: 单线程的多路IO复用

## [Redis 持久化机制（重要）](https://javaguide.cn/database/redis/redis-questions-01.html#redis-持久化机制-重要)
## redis 使用跳表实现有序集合