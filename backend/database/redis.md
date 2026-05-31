---
layout: page
title: "Redis"
permalink: /backend/database/redis/
category: 后端知识
subcategory: 数据库
---

## 概述

Redis（Remote Dictionary Server）是基于内存的高性能 key-value 数据库，支持丰富的数据结构，广泛用于缓存、消息队列、分布式锁等场景。

## 数据类型

### 基本类型

| 类型 | 底层结构 | 典型场景 |
|------|----------|----------|
| String | SDS | 缓存、计数器、分布式锁 |
| List | quicklist（ziplist + 链表） | 消息队列、最新列表 |
| Hash | ziplist / hashtable | 对象存储、购物车 |
| Set | intset / hashtable | 去重、交集并集运算、标签 |
| ZSet | ziplist / skiplist + hashtable | 排行榜、延迟队列 |

### 特殊类型
- **Bitmap**：位操作，签到统计
- **HyperLogLog**：基数估算，UV 统计
- **GEO**：地理位置，附近的人
- **Stream**：消息队列（5.0+）

## 持久化

### RDB（快照）
- 定时将内存数据快照写入磁盘
- 优点：恢复速度快，文件紧凑
- 缺点：可能丢失最后一次快照后的数据

```bash
# 配置
save 900 1      # 900秒内至少1次修改则触发
save 300 10
save 60 10000
```

### AOF（Append Only File）
- 每次写操作追加到日志文件
- 三种刷盘策略：
  - **always**：每条命令都刷盘（最安全，最慢）
  - **everysec**：每秒刷盘（推荐）
  - **no**：由操作系统决定

### 混合持久化（4.0+）
- RDB + AOF 结合
- 重写时以 RDB 格式写入基础数据，增量部分用 AOF

## 内存淘汰策略

| 策略 | 说明 |
|------|------|
| noeviction | 默认，内存满时拒绝写入 |
| allkeys-lru | 所有 key 中淘汰最近最少使用 |
| allkeys-lfu | 所有 key 中淘汰最不经常使用（4.0+） |
| volatile-lru | 设了过期时间的 key 中 LRU |
| volatile-lfu | 设了过期时间的 key 中 LFU |
| allkeys-random | 随机淘汰 |
| volatile-random | 过期 key 中随机淘汰 |
| volatile-ttl | 淘汰 TTL 最短的 |

## 集群方案

### 主从复制
- 全量同步 + 增量同步（repl_backlog）
- 从节点只读

### 哨兵模式（Sentinel）
- 监控主从节点
- 自动故障转移
- 配置提供者

### Cluster 模式
- 数据分片（16384 个 hash slot）
- 节点间 Gossip 协议通信
- 每个 key 通过 `CRC16(key) % 16384` 计算所属 slot

## 常见应用场景

### 分布式锁
```bash
# 加锁
SET lock:order:123 "unique_value" NX EX 30
# 释放锁（Lua 脚本保证原子性）
EVAL "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end" 1 lock:order:123 "unique_value"
```

> 生产环境推荐使用 Redisson 实现分布式锁。

### 缓存击穿/穿透/雪崩

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 缓存穿透 | 查询不存在的数据 | 布隆过滤器、缓存空值 |
| 缓存击穿 | 热点 key 过期 | 互斥锁、热点 key 永不过期 |
| 缓存雪崩 | 大量 key 同时过期 | 过期时间加随机值、多级缓存 |

### 其他场景
- **排行榜**：ZSet + ZRANGEBYSCORE
- **计数器**：INCR / INCRBY
- **消息队列**：List + BRPOP / Stream
- **限流**：滑动窗口（ZSet）/ 令牌桶

## Redisson

### 特点
- 基于 Netty 的 Redis Java 客户端
- 提供分布式锁、限流器、布隆过滤器等高级功能
- 支持多种锁：可重入锁、公平锁、读写锁、红锁

### Watchdog 机制
- 默认 30 秒锁过期
- 后台线程每 10 秒续期，防止业务未完成锁过期

## 常见面试题

1. Redis 为什么快？
2. Redis 的持久化方式？如何选择？
3. 缓存穿透、击穿、雪崩的解决方案？
4. Redis Cluster 的分片原理？
5. 如何实现一个分布式锁？
6. redis 跳表的实现原理，并说说跳表为啥这么快?
- **定义**:跳表 = 多层有序链表，是一种有序、可随机近似二分查找的数据结构。Redis 里 zset（有序集合）底层就是跳表 + 哈希表。
- **快**:一是接近二分查找效率；二是结构简单，无树的旋转，CPU 缓存友好；三是天然适合有序范围查询、排行榜，非常契合 Redis zset 的场景；四是插入删除局部修改，并发性能好。

## 参考资料

- [Redis 官方文档](https://redis.io/docs/)
- 《Redis 设计与实现》
