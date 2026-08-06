
### 一、两种类型的创建方式

使用工厂类 `Executors` 来创建，其可以创建两种类型的 `ScheduledThreadPoolExecutor`：

| 类型 | 说明 |
|------|------|
| ==**`ScheduledThreadPoolExecutor`**== | 包含**若干个线程**的定时任务线程池 |
| ==**`SingleThreadScheduledExecutor`**== | 只包含**一个线程**的定时任务线程池 |

### 二、适用场景

其中具有较多线程的 `ScheduledThreadPoolExecutor` 主要适用于满足资源管理的需求下，依旧限制后台线程的数量的应用场景中需要**多个任务在后台执行周期性执行的类**。

| 场景 | 推荐类型 |
|------|---------|
| ==**需要多个定时任务并发执行**== | `ScheduledThreadPoolExecutor` |
| ==**只需要单个定时任务串行执行**== | `SingleThreadScheduledExecutor` |

### 三、核心方法

| 方法 | 说明 |
|------|------|
| ==**`schedule(Runnable command, long delay, TimeUnit unit)`**== | 在指定延迟后执行一次任务 |
| ==**`scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`**== | 在初始延迟后，以固定**频率**周期执行任务（两次任务开始时间间隔固定） |
| ==**`scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`**== | 在初始延迟后，以固定**延迟**周期执行任务（上次任务结束到下次任务开始间隔固定） |

### 四、`scheduleAtFixedRate` 与 `scheduleWithFixedDelay` 的区别

| 维度 | `scheduleAtFixedRate` | `scheduleWithFixedDelay` |
|------|----------------------|--------------------------|
| ==**时间间隔基准**== | 两次任务**开始时间**的间隔固定 | 上次任务**结束**到下次任务**开始**的间隔固定 |
| ==**任务超时的影响**== | 如果任务执行时间超过周期，下一个任务会**推迟执行**，不会并发 | 不会受影响，始终在上次任务结束后等待固定延迟再执行 |

### 五、使用示例

```java
// 创建包含 3 个线程的定时任务线程池
ScheduledExecutorService executor = Executors.newScheduledThreadPool(3);

// 延迟 1 秒后执行一次
executor.schedule(() -> {
    System.out.println("延迟任务执行");
}, 1, TimeUnit.SECONDS);

// 初始延迟 1 秒后，每 3 秒执行一次（固定频率）
executor.scheduleAtFixedRate(() -> {
    System.out.println("周期性任务执行（固定频率）");
}, 1, 3, TimeUnit.SECONDS);

// 初始延迟 1 秒后，每次任务结束后等待 3 秒再执行（固定延迟）
executor.scheduleWithFixedDelay(() -> {
    System.out.println("周期性任务执行（固定延迟）");
}, 1, 3, TimeUnit.SECONDS);
```

### 六、面试要点

| 问题                                                   | 回答                                                                               |
| ---------------------------------------------------- | -------------------------------------------------------------------------------- |
| **ScheduledThreadPoolExecutor 有哪两种类型**                   | ==**`ScheduledThreadPoolExecutor`（多线程）和 `SingleThreadScheduledExecutor`（单线程）**== |
| **`scheduleAtFixedRate` 和 `scheduleWithFixedDelay` 的区别** | 前者以**任务开始时间**为间隔；后者以**任务结束时间**为间隔                                                |
| **`scheduleAtFixedRate` 任务超时会怎样**                        | 下一个任务会**推迟执行**，不会并发执行                                                            |
| **多线程的 ScheduledThreadPoolExecutor 适用场景**                | 需要**多个定时任务并发执行**的场景                                                              |
