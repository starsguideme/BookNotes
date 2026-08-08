
### 一、核心定位

`SingleThreadExecutor` 是使用**单个工作线程**的执行器，通常情况下其核心线程数和最大线程数都会被设置为 **1**。其他参数与[[FixedThreadPool]]相同。

### 二、与 FixedThreadPool 的关系

`SingleThreadExecutor` 可以看作是 `FixedThreadPool` 的一个特例（当 `corePoolSize = maximumPoolSize = 1` 时）。单例线程池的执行会使用**无界阻塞队列**（`LinkedBlockingQueue`）作为线程池的工作队列，因此与[[FixedThreadPool]]的特性是几乎一样的。

| 维度 | FixedThreadPool | SingleThreadExecutor |
|------|----------------|---------------------|
| ==**核心线程数**== | `n`（自定义） | **1** |
| ==**最大线程数**== | `n`（等于核心线程数） | **1** |
| ==**工作队列**== | 无界 `LinkedBlockingQueue` | 无界 `LinkedBlockingQueue` |
| ==**线程存活时间**== | 0（无效） | 0（无效） |
| ==**不会拒绝任务**== | ✅ | ✅ |

### 三、核心特性

| 特性 | 说明 |
|------|------|
| ==**串行执行**== | 所有任务按照提交顺序依次执行，保证任务执行的有序性 |
| ==**原子性保证**== | 只有一个工作线程，同一时刻只有一个任务在执行，天然保证原子性 |
| ==**不会拒绝任务**== | 使用无界队列，不会拒绝任何任务（除非内存耗尽导致 OOM） |

### 四、面试要点

| 问题                                         | 回答                                                             |
| ------------------------------------------ | -------------------------------------------------------------- |
| **SingleThreadExecutor 的核心特点**                 | ==**只有一个工作线程**==，所有任务**串行执行**                                  |
| **SingleThreadExecutor 和 FixedThreadPool 的关系** | SingleThreadExecutor 是 FixedThreadPool 的特例，核心线程数和最大线程数都为 **1** |
| **SingleThreadExecutor 如何保证原子性**               | 只有一个工作线程，同一时刻只有一个任务在执行，**天然保证原子性**                             |
| **SingleThreadExecutor 会拒绝任务吗**                | ==**不会**==，使用无界队列，除非内存耗尽导致 OOM                                 |