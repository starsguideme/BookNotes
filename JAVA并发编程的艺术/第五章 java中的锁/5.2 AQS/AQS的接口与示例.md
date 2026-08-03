
### 一、什么是模板化设计？

同步器的实现主要依靠于**模板化设计**。换言之，使用者如果要继承并重写同步器的方法，并将这个自定义实现的同步器在方法中实现，一般情况下就要调用以下三种方法来操作同步状态。

### 二、状态操作方法

这三种方法只能用于修改同步器/组件的状态：

| 方法 | 说明 |
|------|------|
| ==**`getState()`**== | 获取当前同步状态的值 |
| ==**`setState(int newState)`**== | 设置当前同步状态的值 |
| ==**`compareAndSetState(int expect, int update)`**== | 使用 CAS 设置当前状态，以此确保修改时的原子性 |

### 三、可重写的方法

以下是使用者可以重写的方法，通过这些方法来实现自定义的同步逻辑：

| 方法 | 说明 |
|------|------|
| ==**`tryAcquire(int arg)`**== | **独占式**获取同步状态，需要查询当前状态并用 CAS 进行设置状态 |
| ==**`tryRelease(int arg)`**== | **独占式**释放同步状态 |
| ==**`tryAcquireShared(int arg)`**== | **共享式**获取同步状态 |
| ==**`tryReleaseShared(int arg)`**== | **共享式**释放同步状态 |
| ==**`isHeldExclusively()`**== | 当前同步器是否处于独占模式且被线程使用 |

### 四、AQS 提供的模板方法

以下是同步器中提供的模板方法的描述：

| 模板方法 | 说明 |
|---------|------|
| ==**`void acquire(int arg)`**== | 独占式获取同步状态，忽略中断 |
| ==**`void acquireInterruptibly(int arg)`**== | 独占式获取同步状态，响应中断 |
| ==**`boolean tryAcquireNanos(int arg, long nanos)`**== | 独占式获取同步状态，带超时机制 |
| ==**`void acquireShared(int arg)`**== | 共享式获取同步状态，忽略中断 |
| ==**`void acquireSharedInterruptibly(int arg)`**== | 共享式获取同步状态，响应中断 |
| ==**`boolean tryAcquireSharedNanos(int arg, long nanos)`**== | 共享式获取同步状态，带超时机制 |
| ==**`boolean release(int arg)`**== | 独占式释放同步状态 |
| ==**`boolean releaseShared(int arg)`**== | 共享式释放同步状态 |
| ==**`Collection<Thread> getQueuedThreads()`**== | 获取等待队列中的线程集合 |

### 五、AQS 方法的三大分类

由提供的模板方法来看，同步器中的方法大致可以分为 3 类：

| 分类 | 说明 | 典型方法 |
|------|------|---------|
| ==**独占式获取与释放同步状态**== | 同一时刻只有一个线程能获取同步状态 | `acquire()`、`release()` |
| ==**共享式获取与释放同步状态**== | 多个线程可以同时获取同步状态 | `acquireShared()`、`releaseShared()` |
| ==**查询等待线程**== | 获取同步队列中的线程信息 | `getQueuedThreads()` |

**注**：以上的方法**全部基于 `state` 这个状态值**来判断当前是独享的还是共享的，亦或者是不在被使用的。`state == 0` 表示锁空闲，`state >= 1` 表示锁被持有。

### 六、模板化设计的核心思想

```text
AQS 模板化设计的层次结构：

使用者（程序员）
    ↓ 调用模板方法
AQS 模板方法（acquire、release 等）
    ↓ 依赖重写方法
子类重写的方法（tryAcquire、tryRelease 等）
    ↓ 操作同步状态
state + CAS + CLH 队列
```

模板方法定义了"**何时**"调用重写方法，重写方法定义了"**如何**"实现同步逻辑。这种设计让使用者只需要关注如何实现自定义的同步逻辑（重写方法），而不需要关心线程排队、阻塞唤醒等底层细节（模板方法）。

### 七、适用范围

| 场景 | 推荐实现 | 说明 |
|------|---------|------|
| ==**需要独占锁**== | 重写 `tryAcquire` 和 `tryRelease` | 如 `ReentrantLock` |
| ==**需要共享锁**== | 重写 `tryAcquireShared` 和 `tryReleaseShared` | 如 `Semaphore`、`CountDownLatch` |
| ==**需要判断锁的持有状态**== | 重写 `isHeldExclusively` | 如 `ReentrantLock` 判断是否是当前线程持锁 |

### 八、面试要点

| 问题            | 回答                                                                                |
| ------------- | --------------------------------------------------------------------------------- |
| **AQS 采用什么设计模式**  | ==**模板方法模式**==，AQS 定义了同步的模板流程，子类重写特定的逻辑                                           |
| **使用者需要重写哪些方法**   | ==**tryAcquire、tryRelease、tryAcquireShared、tryReleaseShared、isHeldExclusively**== |
| **AQS 提供哪三种状态操作** | ==**getState、setState、compareAndSetState**==                                      |
| **AQS 方法分为哪三类**   | ==**独占式获取与释放、共享式获取与释放、查询等待线程**==                                                  |
| **所有方法基于什么判断状态**  | ==**state 状态值**==，`0` 表示空闲，`≥1` 表示被持有                                             |
