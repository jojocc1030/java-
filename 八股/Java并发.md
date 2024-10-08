### [乐观锁和悲观锁](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#乐观锁和悲观锁)

悲观锁： 多写少读
乐观锁：多读少写

### [公平锁和非公平锁有什么区别？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#公平锁和非公平锁有什么区别)
- **公平锁** : 锁被释放之后，先申请的线程先得到锁。性能较差一些，因为公平锁为了保证时间上的绝对顺序，上下文切换更频繁。
- **非公平锁**：锁被释放之后，后申请的线程可能会先获取到锁，是随机或者按照其他优先级排序的。性能更好，但可能会导致某些线程永远无法获取到锁。


### [synchronized 和 ReentrantLock 有什么区别？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#synchronized-和-reentrantlock-有什么区别)

- 两者的都是可重入锁
- synchronized 依赖于JVM, ReentrantLock依赖于JDK层（API）
- Reetrantlock可以指定是公平锁还是非公平锁；而synchronized只能实现非公平锁
- ReentrantLock是可中断锁，而synchronized是不可中断锁

### [线程池处理任务的流程了解吗？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#线程池处理任务的流程了解吗)

![[thread-pool-principle.png]]


### 令牌桶算法
令牌桶算法（Token Bucket Algorithm）是一种网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）的机制，广泛用于网络带宽管理和服务的访问控制。这个算法允许在指定的时间间隔内以固定速率向令牌桶中添加令牌，同时，每个传出的数据包都需要消耗一定数量的令牌才能被发送。如果令牌桶中没有足够的令牌，则数据包需要等待，直到桶中累积足够的令牌。令牌桶算法的优点是允许一定程度的突发流量，因为桶中可以暂时存储多余的令牌。

#### 工作原理

令牌桶算法的工作过程可以描述为以下几个步骤：

1. **令牌的生成**：以恒定的速率向桶中添加令牌。桶的容量有限，如果桶已满，新生成的令牌就会被丢弃。
2. **数据包的发送**：当一个数据包到达时，它会尝试从桶中取出一定数量的令牌（取出的数量通常取决于数据包的大小或其他因素）。如果桶中有足够的令牌，则数据包被发送，同时从桶中移除相应数量的令牌；如果令牌不足，数据包则根据具体的策略（如排队、丢弃等）进行处理。
3. **允许突发**：算法允许数据包以超过平均速率的速度发送，只要桶中有足够的令牌。这对于处理突发流量特别有用。

#### 应用场景

令牌桶算法广泛应用于网络带宽管理和服务的访问控制，例如：

- **限制网络流量**：在网络路由器或防火墙上实现，以限制经过的流量不超过预设的带宽。
- **API速率限制**：在Web服务中实现，以限制客户端的请求速率，防止过度使用或滥用API。
- **流控制**：在多媒体传输和实时通信中，控制数据流的速率，确保通信的平滑性和质量。

#### 与漏桶算法的比较

令牌桶算法和漏桶（Leaky Bucket）算法都是流量整形和速率限制的机制，但它们有一些关键的不同：

- **令牌桶算法**允许突发流量，因为它允许在短时间内以超过平均速率的速度发送数据，只要桶中累积了足够的令牌。
- **漏桶算法**则以恒定的速率输出流量，不管输入流量的速率如何变化，输出流量始终保持恒定，不允许突发。