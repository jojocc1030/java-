## 2. Redis数据类型

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

