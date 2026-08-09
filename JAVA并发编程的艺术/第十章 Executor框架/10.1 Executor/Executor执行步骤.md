
### 一、任务对象的创建

主线程先要创建 `Runnable` 和 `Callable` 接口的任务对象。工具类中的 `Executors` 可以把 `Runnable` 对象封装成一个 `Callable` 对象：

| 方法                                                         | 说明                                              |
| ---------------------------------------------------------- | ----------------------------------------------- |
| ==**`Executors.callable(Runnable task)`**==                | 将 `Runnable` 封装成 `Callable`，没有返回值               |
| ==**`Executors.callable(Runnable task, Object result)`**== | 将 `Runnable` 封装成 `Callable`，执行完成后返回指定的 `result` |

### 二、任务提交到线程池

把任务对象提交给 `ExecutorService` 执行有两种方式：

| 方式           | 方法                                                   | 说明                                |
| ------------ | ---------------------------------------------------- | --------------------------------- |
| ==**直接执行**== | `execute(Runnable command)`                          | 无返回值，无法感知任务执行结果                   |
| ==**提交执行**== | `submit(Runnable task)` 或 `submit(Callable<T> task)` | 返回 `Future<T>`，可以通过 `get()` 获取返回值 |

### 三、FutureTask 的使用

如果执行的是 `submit`，`ExecutorService` 会返回一个实现了 `Future` 接口的对象。`FutureTask` 同时实现了 `Runnable` 和 `Future` 接口，因此同样可以创建 `FutureTask` 这个任务类，将其交给 `ExecutorService` 接口执行。

**FutureTask 的两种使用方式**：

| 方式 | 说明 |
|------|------|
| ==**直接交给线程池执行**== | `executorService.submit(futureTask)` |
| ==**单独启动线程执行**== | `new Thread(futureTask).start()` |

### 四、完整示例

```java
// 方式一：直接 execute
executorService.execute(() -> {
    // 执行任务，无返回值
});

// 方式二：submit Callable
Future<String> future = executorService.submit(() -> {
    return "任务执行结果";
});
String result = future.get();  // 获取返回值

// 方式三：使用 FutureTask
FutureTask<String> futureTask = new FutureTask<>(() -> {
    return "任务执行结果";
});
executorService.submit(futureTask);
String result = futureTask.get();  // 获取返回值
```

### 五、面试要点

| 问题                       | 回答                                                |
| ------------------------ | ------------------------------------------------- |
| **如何把 Runnable 转成 Callable** | ==**`Executors.callable(Runnable task)`**==       |
| **execute 和 submit 的区别**     | `execute` 无返回值；`submit` 返回 `Future`，可以获取任务执行结果    |
| **FutureTask 的作用**           | ==**同时实现 Runnable 和 Future**==，既可以作为任务执行，也可以获取返回值 |
| **FutureTask 有哪两种使用方式**      | 直接交给线程池执行，或单独启动线程执行                               |
