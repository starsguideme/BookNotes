
### 一、Object 监视器方法与 Condition 的对比

任意一个 Java 对象都有其对应的监视器方法，这些方法包括 `wait()`、`wait(long timeout)`、`notify()`、`notifyAll()`，这些方法都可以与 `synchronized` 进行配合，实现完成等待/通知模式。

虽然 `Condition` 接口也提供了类似 Object 的监视器方法，也可以与 `Lock` 配合实现完成等待/通知模式，但这两者的性能相差还是很大的。

### 二、Object 监视器方法与 Condition 的对比

| 对比项            | Object 监视器方法         | Condition                           |
| -------------- | -------------------- | ----------------------------------- |
| ==**配合的锁**==   | `synchronized`       | `Lock`（如 `ReentrantLock`）           |
| ==**等待方法**==   | `wait()`             | `await()`                           |
| ==**超时等待**==   | `wait(long timeout)` | `await(long time, TimeUnit unit)`   |
| ==**唤醒单个线程**== | `notify()`           | `signal()`                          |
| ==**唤醒所有线程**== | `notifyAll()`        | `signalAll()`                       |
| ==**多个等待队列**== | ==**不支持**==，只有一个等待队列 | ==**支持**==，一个 Lock 可以创建多个 Condition |
| ==**JDK 版本**== | JDK 1.0 引入           | JDK 5 引入                            |

### 三、Condition 的核心优势

`Condition` 相比 Object 监视器方法的最大优势在于**支持多个等待队列**。一个 `Lock` 可以调用 `newCondition()` 创建多个 `Condition` 实例，不同的 `Condition` 用于控制不同条件的等待和唤醒。

例如在生产者-消费者模式中，可以使用两个 Condition——一个用于"缓冲区为空时消费者等待"，另一个用于"缓冲区已满时生产者等待"。这样可以精准唤醒对应的线程，而不是像 `notifyAll()` 那样唤醒所有等待线程，从而减少不必要的线程上下文切换开销。

### 四、面试要点

| 问题                          | 回答                                                                            |
| --------------------------- | ----------------------------------------------------------------------------- |
| **Condition 和 Object 监视器方法的区别** | Condition 配合 `Lock` 使用，支持**多个等待队列**；Object 监视器方法配合 `synchronized` 使用，只有一个等待队列 |
| **Condition 的核心优势**             | ==**支持多个等待队列**==，可以精准唤醒对应条件的线程                                                |
| **Condition 有哪些核心方法**           | `await()`、`await(timeout)`、`signal()`、`signalAll()`                           |
