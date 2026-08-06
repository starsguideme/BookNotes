
### 一、核心定位

`Future` 接口和实现这个接口的 `FutureTask` 类一般是用来表示**异步计算的结果**。当线程池执行器（`ThreadPoolExecutor`）和 `ScheduledThreadPoolExecutor` 接收到 `Runnable` 接口或 `Callable` 接口的实现类时，会向我们返回一个 `FutureTask` 对象。

### 二、`submit` 方法的调用链

当我们调用线程池的 `submit(Callable task)` 方法时，内部的执行流程如下：

```text
① 调用 ExecutorService.submit(Callable<T> task)
    ↓
② 内部调用 AbstractExecutorService.newTaskFor(Callable<T> task)
    ↓
③ newTaskFor 返回 new FutureTask<T>(callable)
    ↓
④ 调用 execute(Runnable command) 将 FutureTask 作为任务提交到线程池
    ↓
⑤ 返回 FutureTask 对象给调用者（作为 Future<T> 接口）
```

**关键点**：`submit` 方法内部通过 `newTaskFor` 将 `Callable` 包装成 `FutureTask`，然后调用 `execute` 提交任务，最后返回 `FutureTask` 对象给调用者。

### 三、FutureTask 的状态转换

`FutureTask` 内部维护了一个状态机，在整个生命周期中会经历以下状态转换：

```text
NEW（初始状态）
    ↓ 任务开始执行
COMPLETING（正在完成中，结果正在被设置）
    ↓ 结果设置完成
NORMAL（任务正常完成）
    ↓ 或被取消
    ↓ CANCELLED（任务被取消）
    ↓ 或抛出异常
EXCEPTIONAL（任务执行异常）
```

| 状态                    | 说明                           |
| --------------------- | ---------------------------- |
| ==**`NEW`**==         | 初始状态，任务尚未开始执行                |
| ==**`COMPLETING`**==  | 任务正在完成，结果正在被设置到 `outcome` 字段 |
| ==**`NORMAL`**==      | 任务正常执行完毕，结果可用                |
| ==**`EXCEPTIONAL`**== | 任务执行过程中抛出异常                  |
| ==**`CANCELLED`**==   | 任务被取消                        |
| ==**`INTERRUPTED`**== | 任务正在运行时被中断（取消）               |

### 四、FutureTask 的 `get()` 和 `cancel()` 实现

**`get()` 方法的底层逻辑**：当调用 `get()` 方法时，`FutureTask` 会检查当前状态。如果状态 ≤ `COMPLETING`（即任务还未完成），则调用 `awaitDone()` 方法，内部通过 `LockSupport.park()` 阻塞当前线程，等待任务完成或超时。

**`cancel(mayInterruptIfRunning)` 方法的底层逻辑**：当调用 `cancel` 时，会先判断当前状态是否为 `NEW`。如果不是 `NEW`，则直接返回 `false`（任务已开始执行无法取消）。如果是 `NEW`，则通过 CAS 将状态更新为 `INTERRUPTING` 或 `CANCELLED`，如果需要中断正在运行的线程，则调用 `t.interrupt()`。

### 五、面试要点

| 问题                           | 回答                                                                          |
| ---------------------------- | --------------------------------------------------------------------------- |
| **`submit` 方法内部如何创建 FutureTask** | 通过 `newTaskFor()` 将 `Callable` 包装成 `FutureTask`，再调用 `execute` 提交任务          |
| **FutureTask 有哪些核心状态**           | ==**`NEW`、`COMPLETING`、`NORMAL`、`EXCEPTIONAL`、`CANCELLED`、`INTERRUPTED`**== |
| **`get()` 方法的阻塞原理**              | 调用 `awaitDone()`，内部通过 **`LockSupport.park()`** 阻塞当前线程                       |
| **`cancel` 方法在什么情况下返回 `false`**  | ==**任务状态不是 `NEW`**==（任务已开始执行无法取消）                                           |

