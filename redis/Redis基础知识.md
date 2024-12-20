# Redis 基础知识

Date：2024-12-04

作者：Cary

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Redis 基础知识](#redis-基础知识)
  - [Redis 基础](#redis-基础)
    - [Redis 基本数据类型](#redis-基本数据类型)
    - [Redis 的线程模型](#redis-的线程模型)
    - [什么是缓存击穿、缓存穿透、缓存雪崩？](#什么是缓存击穿-缓存穿透-缓存雪崩)
      - [缓存穿透](#缓存穿透)
      - [缓存雪崩](#缓存雪崩)
      - [缓存击穿](#缓存击穿)
    - [Redis 的过期策略](#redis-的过期策略)
      - [过期策略](#过期策略)
    - [持久化机制](#持久化机制)
    - [Redis 的高可用](#redis-的高可用)
    - [Redisson](#redisson)
    - [MySQL 与 Redis 如何保证双写一致性](#mysql-与-redis-如何保证双写一致性)
    - [为什么 Redis 6.0 之后改多线程呢？](#为什么-redis-60-之后改多线程呢)
    - [Redis 的 Hash 冲突怎么办](#redis-的-hash-冲突怎么办)

<!-- /code_chunk_output -->

- 定义
  Redis 是非关系型数据库，以 Key-Value 方式存储，基于内存。

- 优势

redis 因为存储在内存中，所以具有极高的读写速度

## Redis 基础

### Redis 基本数据类型

- String（字符串）

- Hash（哈希）

在 Redis 中，Hash 是指值本身又是一个键值对结构

- List（列表）

值可以存储多个有序的字符串

- Set（集合）

保存多个字符串元素，但不允许重复

- zset（有序集合）

已排序的字符串集合，同时元素不能重复

### Redis 的线程模型

- I/O 多路复用模型

I/O 多路复用模型的优势是可以通过一个线程处理大量连接

Redis 在不同操作系统有不同的模型，在 Linux 系统中使用的是 epoll

服务器用事件循环的方式处理 I/O 事件，事件循环会不断调用 epoll 函数来等待事件发生并处理。

- 单线程模型

Redis 是单线程模型的，如果一个命令执行过长，会造成阻塞。

### 什么是缓存击穿、缓存穿透、缓存雪崩？

#### 缓存穿透

指查询一个一定不存在的数据，每次查询不存在的数据就会绕过 redis 去查询 mysql 等，会给数据库造成压力。

防范措施：

- 非法请求可以在 API 入口加入参数校验

- 当查询数据库没用找到对应数据后，讲一个空值存入缓存，并设置短期的过期时间，下次请求就会从缓存中获取空值，不会再去查询数据库。

#### 缓存雪崩

缓存中大量数据过期，同时查询量巨大，请求就会直接访问数据库，导致数据库压力过大。

防范措施：

- 可以通过均匀的设置数据的过期时间，不让他们同时过期，通过较大的固定值+较小的随机值实现。

- 构造 redis 高可用集群

#### 缓存击穿

指热点 Key 在某个时间点过期时，恰巧在这个时候有大量的并发请求访问到数据库造成压力。

防范措施：

- 不设置过期时间，在热点数据快要过期时，异步线程更新和设置过期时间。

### Redis 的过期策略

#### 过期策略

- 定时过期：为每个 key 创造一个定时器，到期后对 key 清除。

- 惰性过期：当访问一个 key 时，判断是否过期，过期则清除。

- 定期过期： 每隔一定的时间，会扫描数据库中过期的 key 并清除。

### 持久化机制

防止 redis 的数据丢失，有两种主要的持久化方式

- RDB

ROB 把内存数据以快照的形式保存到磁盘上

在指定的时间间隔内，执行指定次数写的操作，将内存中的数据集写入磁盘。执行完后，会在指定目录生成文件，Redis 重启会重新加载这个文件生成数据。

- AOF

：AOF 可以支持实时的数据持久化。AOF 持久化功能通过保存 redis 服务器所执行的写命令来记录数据库状态，默认存储在 appendonly.aof 中，AOF 文件中存储的都是 redis 相关命令。

### Redis 的高可用

Redis 有三种模式可以防止单机挂掉无法提供服务

- 主从模式

通过在 redis.conf 拷贝出 redis-6371.conf，配置如下，同时从节点会全量负责 master 的数据

```
replicaof 127.0.0.1 6379   # 从本机6379的redis实例复制数据

replica-read-only yes  # 配置从节点只读

```

完成主从模式的配置后，如果 master 的数据发生更新，会触发增量复制向从节点同步。

- 哨兵模式

由一个或多个 Sentinel 实例组成的 Sentinel 系统，可以在主节点挂掉时，自动将某个子节点升级为新的 master

- 集群模式

集群模式可以基于分布式存储来对数据进行切片，每台 redis 节点上都可以存储不同的内容，集群节点通讯通过 Gossip 协议。

Gossop 消息常见的有四种：

- Ping

- Pong

- Meet

- fail

### Redisson

Redisson 框架是 Redis 官方推荐的客户端，提供了 Redis 最方便的使用方式。

只需要添加依赖

```

<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.37.0</version>
</dependency>
```

然后配置 redis 信息，就可以对 redis 进行操作

```
RedissonClient redisson = Redisson.create(config);


RMap<MyKey, MyValue> map = redisson.getMap("myMap");
```

对应 Redisson 的使用，及 API 调用，这有一个非常全的文章 `https://blog.csdn.net/A_art_xiang/article/details/125525864`

Redisson 对于分布式锁的功能也非常强大，实现原理是基于 redis 的 SETNX 命令和 Lua 脚本。

### MySQL 与 Redis 如何保证双写一致性

在业务中，如果 redis 和 mysql 有一方故障，有可能会导致数据不一致的事情发生。

解决方案：

- 可以在一个事务中同时写入 redis 的命令和 mysql 的命令，如果有一方失败，回滚事务

- 使用消息队列，将写入 redis 的消息发送到消息队列，再由消费者服务拉取消息，获取消息成功时写入 mysql。

### 为什么 Redis 6.0 之后改多线程呢？

redis 在 2020 年发布了 6.0 的版本，引入了多线程模型。

单线程无法发挥多核 cpu 的优势，多线程模型可以将读取和写入 I/O 套接字费时间的操作委托给其他线程。

### Redis 的 Hash 冲突怎么办

什么是哈希冲突？

- 通过不同的 key，计算出一样的哈希值，导致落在一个哈希桶中。

什么是哈希桶？

![](https://pic3.zhimg.com/80/v2-7cb2b211b8d343894954eae71e66e9d8_720w.webp)

解决方法：

采用链式哈希，在一个哈希桶中，多个 entry 用指针连接。
