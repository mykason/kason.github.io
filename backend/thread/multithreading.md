---
layout: page
title: "多线程"
permalink: /backend/java-basic/multithreading/
category: 后端知识
subcategory: JAVA 基础
---

## 概述

Java 多线程是 Java 并发编程的基础，通过线程实现并行执行，提升程序性能和资源利用率。

## 线程基础

### 创建线程的方式
1. 继承 `Thread` 类
2. 实现 `Runnable` 接口
3. 实现 `Callable` 接口（有返回值）
4. 使用线程池 `ExecutorService`

### 线程状态

```
NEW → RUNNABLE → BLOCKED / WAITING / TIMED_WAITING → TERMINATED
```

| 状态 | 说明 |
|------|------|
| NEW | 线程已创建但未启动 |
| RUNNABLE | 可运行（含 READY 和 RUNNING） |
| BLOCKED | 等待获取锁 |
| WAITING | 无限期等待（wait/join/park） |
| TIMED_WAITING | 限期等待（sleep/wait(timeout)） |
| TERMINATED | 线程执行完毕 |

## 线程同步

### synchronized
- 修饰实例方法：锁当前对象
- 修饰静态方法：锁 Class 对象
- 修饰代码块：锁指定对象
- 底层：monitorenter / monitorexit 字节码指令

### Lock 接口
```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock();
}
```

### synchronized vs ReentrantLock
| 特性 | synchronized | ReentrantLock |
|------|-------------|---------------|
| 实现方式 | JVM 内置 | API 层面 |
| 锁释放 | 自动 | 手动 unlock |
| 可中断 | 不可 | 可中断 |
| 公平锁 | 非公平 | 可选公平/非公平 |
| 条件变量 | 单一（wait/notify） | 多个 Condition |

## 线程池

### 核心参数（ThreadPoolExecutor）
```java
public ThreadPoolExecutor(
    int corePoolSize,       // 核心线程数
    int maximumPoolSize,    // 最大线程数
    long keepAliveTime,     // 空闲线程存活时间
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,  // 任务队列
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler    // 拒绝策略
)
```

### 拒绝策略
- **AbortPolicy**：抛出异常（默认）
- **CallerRunsPolicy**：由调用线程执行
- **DiscardPolicy**：直接丢弃
- **DiscardOldestPolicy**：丢弃队列中最老的任务

### Executors 提供的线程池
- `newFixedThreadPool`：固定大小，无界队列（OOM 风险）
- `newCachedThreadPool`：可缓存，线程数无上限
- `newSingleThreadExecutor`：单线程
- `newScheduledThreadPool`：定时/周期执行

> 生产环境推荐手动创建 ThreadPoolExecutor，避免 OOM。

## volatile 关键字

- 保证可见性：修改后立即刷新到主内存
- 禁止指令重排序
- **不保证原子性**

## CAS 与原子类

### CAS（Compare And Swap）
- 乐观锁思想：比较当前值与期望值，相同则更新
- 底层调用 CPU 指令（cmpxchg），保证原子性
- ABA 问题 → 用 `AtomicStampedReference` 解决

### 常用原子类
- `AtomicInteger`、`AtomicLong`、`AtomicBoolean`
- `AtomicReference`、`AtomicStampedReference`
- `LongAdder`（高并发下优于 AtomicLong）

## AQS（AbstractQueuedSynchronizer）

- 并发包的核心框架
- 维护一个 volatile int state 和一个 FIFO 等待队列
- ReentrantLock、Semaphore、CountDownLatch、ReentrantReadWriteLock 均基于 AQS

## 并发工具类

| 工具 | 用途 |
|------|------|
| CountDownLatch | 等待 N 个线程完成 |
| CyclicBarrier | N 个线程互相等待到齐后继续 |
| Semaphore | 控制并发访问数量 |
| Exchanger | 两个线程交换数据 |

## 并发集合

| 集合 | 特点 |
|------|------|
| ConcurrentHashMap | 分段锁/ CAS 高并发 Map |
| CopyOnWriteArrayList | 写时复制，适合读多写少 |
| BlockingQueue | 生产者-消费者模型 |
| ConcurrentSkipListMap | 跳表实现，有序并发 Map |

## 常见面试题

1. 线程池的核心参数和工作流程？
2. synchronized 和 Lock 的区别？
3. volatile 的作用和局限性？
4. 什么是 AQS？原理是什么？
5. ThreadLocal 原理及内存泄漏问题？
