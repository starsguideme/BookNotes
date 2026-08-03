
### 一、什么是 Lock？

锁是用来实现控制线程资源共享的一种实现手段，常见锁有 `synchronized`、`Lock`、`ReentrantLock` 锁等。一个锁能够防止多个线程同时访问共享资源。

随着 `JDK` 的发展，出现了与 `synchronized` 类似的东西，但其是一个需要被**显式使用**的接口，这个接口叫做 **`Lock` 接口**。

### 二、Lock 与 synchronized 的对比

| 维度 | synchronized | Lock |
|------|-------------|------|
| ==**获取与释放**== | 隐式获取，自动释放 | **显式获取，必须手动释放**（在 `finally` 块中调用 `unlock()`） |
| ==**灵活性**== | 低，无法中断、无法超时 | **高**，支持中断、超时、非阻塞尝试 |
| ==**可扩展性**== | 弱，功能固定 | **强**，可通过 `Condition` 实现多条件等待 |
| ==**使用复杂度**== | 简单 | 复杂，需要手动管理锁的生命周期 |
| ==**性能（JDK 6+）**== | 优化后与 Lock 差距很小 | 略高，但差异不大 |

虽然在使用过程中，Lock 时常都需要比 `synchronized` 多一个释放锁的操作，但也带来了一个很大的好处——可以对锁进行操作，包括但不限于**锁的中断、超时之后获得锁、一般的锁获取和释放锁的控制权**。因此 Lock 接口下的锁机制更灵活，但也更复杂。

而对一般意义上的 `synchronized` 则是采取了**隐式获取锁**，虽然在操作上很方便，但它也因此没有了灵活性和强大的可扩展性。

### 三、Lock 接口的核心特性

在名为 Lock 的接口中，拥有着以下 `synchronized` 不具备的特性：

| 特性 | 说明 |
|------|------|
| ==**尝试非阻塞的获取锁**== | 通过 `tryLock()` 方法，如果锁可用则立即获取，否则立即返回 `false`，不会阻塞 |
| ==**能被中断的获取锁**== | 通过 `lockInterruptibly()` 方法，等待锁的线程可以响应中断，避免无限期等待 |
| ==**超时获取锁**== | 通过 `tryLock(timeout, unit)` 方法，在指定时间内等待，超时后自动返回 |

### 四、Lock 接口的 API

在 Lock 接口中，有一些共有的 API：

| API | 说明 |
|-----|------|
| ==**`void lock()`**== | 获取锁，如果锁不可用则当前线程阻塞等待 |
| ==**`void lockInterruptibly()`**== | 可中断地获取锁，等待期间可以响应中断 |
| ==**`boolean tryLock()`**== | 尝试非阻塞获取锁，成功返回 `true`，失败立即返回 `false` |
| ==**`boolean tryLock(long time, TimeUnit unit)`**== | 带超时的尝试获取锁，在指定时间内等待，超时返回 `false` |
| ==**`void unlock()`**== | 释放锁，必须在 `finally` 块中调用以确保锁被释放 |
| ==**`Condition newCondition()`**== | 返回一个绑定到此 Lock 的 Condition 实例，用于线程间的等待/通知 |

以上 API 的实现均是由于 Lock 接口中的 **[[AQS的定义与特点]]（AbstractQueuedSynchronizer）** 的子类来完成线程控制访问的。AQS 是 Lock 接口的底层实现框架，通过 `volatile int state` + CAS + CLH 同步队列来管理锁的获取和释放。

### 五、Lock 的使用模板

```java
Lock lock = new ReentrantLock();
lock.lock();  // 获取锁
try {
    // 访问共享资源
} finally {
    lock.unlock();  // 确保锁被释放
}
```

### 六、适用范围

| 场景 | 推荐方案 | 说明 |
|------|---------|------|
| ==**简单同步需求**== | `synchronized` | 代码简洁，自动释放锁，不易出错 |
| ==**需要可中断的锁获取**== | `Lock.lockInterruptibly()` | 适合长时间任务需要取消的场景 |
| ==**需要超时获取锁**== | `Lock.tryLock(timeout, unit)` | 避免无限期阻塞 |
| ==**需要非阻塞尝试**== | `Lock.tryLock()` | 尝试获取锁，失败立即返回 |
| ==**需要多个条件变量**== | `Lock.newCondition()` | 实现更精细的线程等待/通知机制 |

### 七、面试要点

| 问题                        | 回答                                                                                              |
| ------------------------- | ----------------------------------------------------------------------------------------------- |
| **Lock 和 synchronized 的核心区别** | Lock 显式获取、手动释放，更灵活；synchronized 隐式获取、自动释放，更简单                                                   |
| **Lock 的三个核心特性**              | ==**尝试非阻塞获取锁、能被中断的获取锁、超时获取锁**==                                                                 |
| **Lock 有哪些核心 API**            | `lock()`、`lockInterruptibly()`、`tryLock()`、`tryLock(timeout, unit)`、`unlock()`、`newCondition()` |
| **Lock 的底层实现依赖什么**            | ==**AQS（AbstractQueuedSynchronizer）**==，通过 `volatile int state` + CAS + CLH 队列实现                |
