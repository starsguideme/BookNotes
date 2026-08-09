
### 一、核心定位

`ScheduledThreadPoolExecutor` 也继承于 `ThreadPoolExecutor`，其主要用于在给定的延迟之后运行任务，或在定期执行任务时使用。这个 `ScheduledThreadPoolExecutor` 的功能其实与 `Timer` 类似，但其本质更强大也更灵活，因为 `Timer` 是**单线程**的，而 `ScheduledThreadPoolExecutor` 则可以通过设置变成**多线程后台任务**。

### 二、与 Timer 的对比

| 维度 | Timer | ScheduledThreadPoolExecutor |
|------|-------|---------------------------|
| ==**线程模型**== | 单线程 | **多线程**（可配置核心线程数） |
| ==**异常处理**== | 如果某个任务抛出异常，Timer 线程会**终止**，后续任务无法执行 | 某个任务抛出异常不影响其他任务继续执行 |
| ==**灵活性**== | 较低，只支持绝对时间和相对延迟 | 更高，支持固定频率和固定延迟两种模式 |
| ==**适用场景**== | 简单的单线程定时任务 | 需要**多线程并发**执行定时任务的场景 |

### 三、学习路径

以下是运行机制[[运行机制]]，在了解完其运行机制后就可以去了解它具体可以实现什么了：[[具体实现]]。

### 四、面试要点

| 问题                                      | 回答                                                                      |
| --------------------------------------- | ----------------------------------------------------------------------- |
| **ScheduledThreadPoolExecutor 和 Timer 的区别** | Timer 是**单线程**，任务异常会终止；ScheduledThreadPoolExecutor 是**多线程**，任务异常不影响其他任务 |
| **ScheduledThreadPoolExecutor 继承于哪个类**      | ==**`ThreadPoolExecutor`**==                                            |
| **ScheduledThreadPoolExecutor 适用什么场景**      | 需要**多线程并发**执行定时任务的场景                                                    |
