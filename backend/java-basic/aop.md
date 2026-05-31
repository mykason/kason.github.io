---
layout: page
title: "AOP"
permalink: /backend/java-basic/aop/
category: 后端知识
subcategory: JAVA 基础
---

# AOP

AOP（Aspect Oriented Programming，面向切面编程）是一种编程思想，用于将系统中与核心业务无关、但会被多个业务模块共同使用的逻辑抽取出来统一处理，例如日志、权限校验、事务、限流、缓存等。

它的核心目标是：**在不修改业务代码的前提下，对业务方法进行增强**，从而减少重复代码，降低模块耦合度，提高系统的可维护性和扩展性。

## 为什么需要 AOP

假设很多接口都需要记录请求日志，如果在每个业务方法中都手动写日志代码，会带来几个问题：

- 重复代码多，维护成本高
- 日志逻辑和业务逻辑混在一起，代码不清晰
- 后续修改日志格式时，需要改动大量业务代码

使用 AOP 后，可以把日志逻辑统一放到一个切面中，由框架在目标方法执行前后自动织入日志逻辑。

## AOP 核心概念

### 切面（Aspect）

切面是对横切关注点的抽象，通常是一个类。

例如：日志切面、权限切面、事务切面、限流切面。

### 连接点（Join Point）

连接点表示程序执行过程中的某个点。在 Spring AOP 中，连接点通常指的是**方法的执行**。

例如：调用 `UserService.getUser()` 方法这个动作就是一个连接点。

### 切点（Pointcut）

切点用于定义哪些连接点需要被增强。

例如：

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

表示匹配 `com.example.service` 包下所有类的所有方法。

### 通知（Advice）

通知表示在目标方法的什么时机执行增强逻辑。

常见通知类型：

- `@Before`：目标方法执行前执行
- `@After`：目标方法执行后执行，不管是否异常都会执行
- `@AfterReturning`：目标方法正常返回后执行
- `@AfterThrowing`：目标方法抛出异常后执行
- `@Around`：环绕通知，可以在目标方法执行前后都增强，也可以控制是否执行目标方法

### 目标对象（Target）

目标对象是被代理的原始业务对象。

例如：`UserServiceImpl`。

### 代理对象（Proxy）

代理对象是 AOP 框架生成的对象，外部调用代理对象时，代理对象会在合适的时机执行增强逻辑，再调用目标对象。

### 织入（Weaving）

织入是指把切面逻辑应用到目标对象上的过程。

Spring AOP 的织入发生在运行期，通过动态代理实现。

## Spring AOP 的实现原理

Spring AOP 底层主要基于动态代理实现：

- 如果目标类实现了接口，默认使用 **JDK 动态代理**
- 如果目标类没有实现接口，使用 **CGLIB 动态代理**

### JDK 动态代理

JDK 动态代理基于接口生成代理类，要求目标类必须实现接口。

特点：

- Java 原生支持
- 只能代理接口方法
- 通过 `InvocationHandler` 处理方法调用

### CGLIB 动态代理

CGLIB 通过继承目标类生成子类代理对象。

特点：

- 不要求目标类实现接口
- 不能代理 `final` 类
- 不能代理 `final` 方法，因为子类无法重写

## AOP 常见使用场景

### 统一日志管理

通过 AOP 在接口调用前后统一记录请求参数、响应结果、耗时、异常信息等。

```java
@Aspect
@Component
public class LogAspect {

    @Around("execution(* com.example.controller.*.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            Object result = joinPoint.proceed();
            long cost = System.currentTimeMillis() - start;
            System.out.println("method=" + joinPoint.getSignature() + ", cost=" + cost + "ms");
            return result;
        } catch (Throwable e) {
            System.out.println("method=" + joinPoint.getSignature() + ", error=" + e.getMessage());
            throw e;
        }
    }
}
```

### 接口防刷和限流

可以自定义注解，例如 `@RateLimit`，再通过 AOP 拦截带有该注解的方法，结合 Redis 或 Redisson 统计访问次数，实现接口防刷。

核心思路：

1. 定义限流注解，配置时间窗口和最大访问次数
2. AOP 拦截带注解的方法
3. 根据用户 ID、IP、接口路径等生成限流 key
4. 使用 Redis 记录访问次数并设置过期时间
5. 超过阈值时拒绝请求

### 权限控制

Spring Security 中的 `@PreAuthorize` 就是典型的 AOP 应用。

```java
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/users")
public List<User> listUsers() {
    return userService.listUsers();
}
```

方法执行前会先进行权限判断，权限不足时直接拒绝访问。

### 事务管理

Spring 的声明式事务 `@Transactional` 也是基于 AOP 实现的。

```java
@Transactional
public void transfer(Long fromUserId, Long toUserId, BigDecimal amount) {
    accountService.deduct(fromUserId, amount);
    accountService.add(toUserId, amount);
}
```

Spring 会在方法执行前开启事务，方法正常执行完成后提交事务，发生异常时回滚事务。

### 缓存处理

Spring Cache 中的 `@Cacheable`、`@CacheEvict`、`@CachePut` 也可以理解为 AOP 思想的应用。

```java
@Cacheable(cacheNames = "user", key = "#id")
public User getUserById(Long id) {
    return userMapper.selectById(id);
}
```

调用方法前先查询缓存，如果缓存命中则直接返回，不再执行目标方法。

## AOP 的优点

- 降低重复代码，提高复用性
- 让业务逻辑和通用逻辑解耦
- 便于统一维护日志、权限、事务等横切逻辑
- 对原有业务代码侵入较小

## AOP 的缺点

- 过度使用会降低代码可读性，真实执行逻辑不够直观
- 动态代理会带来一定性能开销
- 调试链路相对复杂
- Spring AOP 只支持方法级别的增强，不支持字段、构造器等更细粒度的连接点

## Spring AOP 常见失效场景

### 同类方法内部调用

AOP 是通过代理对象生效的，如果在同一个类中直接调用另一个被增强的方法，调用不会经过代理对象，AOP 就不会生效。

```java
@Service
public class UserService {

    public void methodA() {
        methodB(); // 直接调用当前对象方法，不经过代理
    }

    @Transactional
    public void methodB() {
        // 事务可能不生效
    }
}
```

解决思路：

- 将被增强方法拆到另一个 Bean 中调用
- 通过 Spring 容器获取代理对象后再调用

### 方法不是 public

Spring AOP 通常基于代理调用 public 方法。非 public 方法上的 AOP 注解可能不会按预期生效。

### final 类或 final 方法

CGLIB 通过继承目标类并重写方法实现代理，因此 `final` 类和 `final` 方法无法被 CGLIB 正常代理。

### 对象不是 Spring Bean

只有交给 Spring 容器管理的对象，才有机会被 Spring AOP 创建代理。如果对象是手动 `new` 出来的，AOP 不会生效。

## Spring AOP 和 AspectJ 的区别

| 对比项 | Spring AOP | AspectJ |
| --- | --- | --- |
| 实现方式 | 动态代理 | 编译期、类加载期或运行期织入 |
| 连接点 | 主要支持方法执行 | 支持方法、构造器、字段等更多连接点 |
| 使用成本 | 低，和 Spring 集成简单 | 相对更高 |
| 功能能力 | 较轻量 | 更强大 |
| 适用场景 | 常见业务增强，如日志、事务、权限 | 复杂切面需求 |

## 常见面试题

### 1. AOP 是什么？

AOP 是面向切面编程，用于把日志、权限、事务、限流等横切逻辑从业务代码中抽离出来，通过动态代理在目标方法执行前后进行增强。

### 2. Spring AOP 的底层原理是什么？

Spring AOP 底层主要基于动态代理。如果目标类实现了接口，默认使用 JDK 动态代理；如果没有实现接口，则使用 CGLIB 代理。代理对象会在调用目标方法前后执行切面逻辑。

### 3. JDK 动态代理和 CGLIB 有什么区别？

JDK 动态代理基于接口，只能代理接口方法；CGLIB 基于继承生成目标类的子类，不要求接口，但不能代理 final 类和 final 方法。

### 4. AOP 为什么会出现同类方法调用失效？

因为 Spring AOP 是通过代理对象增强的。同类内部方法调用使用的是当前对象 `this`，不会经过 Spring 代理对象，所以切面逻辑不会执行。

### 5. `@Transactional` 为什么有时不生效？

常见原因包括：同类方法内部调用、方法不是 public、异常被捕获没有继续抛出、抛出的异常类型默认不触发回滚、对象不是 Spring Bean 等。

### 6. `@Around` 和 `@Before` 有什么区别？

`@Before` 只能在目标方法执行前增强，不能控制目标方法是否执行；`@Around` 可以在方法执行前后增强，并且可以通过 `proceed()` 控制目标方法是否执行。

## 小结

AOP 的本质是通过代理机制对目标方法进行增强。它适合处理日志、权限、事务、限流、缓存等横切逻辑，但不适合承载核心业务逻辑。使用时要注意代理失效场景，尤其是同类方法内部调用、非 Spring Bean、final 方法等问题。
