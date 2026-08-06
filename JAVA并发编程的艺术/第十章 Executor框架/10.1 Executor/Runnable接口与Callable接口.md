
### 一、核心区别

`Runnable` 和 `Callable` 接口的实现类，都可以被线程池执行器和 `ScheduledThreadPoolExecutor` 执行。它们的区别是：

| 接口 | 是否有返回值 | 是否抛出受检异常 |
|------|------------|----------------|
| ==**`Runnable`**== | **无返回值** | **不能抛出受检异常** |
| ==**`Callable<V>`**== | **有返回值**（泛型 `V`） | **可以抛出受检异常** |

### 二、Runnable 转 Callable

除了 `Callable` 本身外，一般情况下也可以使用执行器将 `Runnable` 转化成 `Callable`。以下是其 API：

| API | 说明 |
|-----|------|
| ==**`public static Callable<Object> callable(Runnable task)`**== | 原生自带的，返回的是 `Callable<Object>` 对象，`get()` 返回 `null` |
| ==**`public static <T> Callable<T> callable(Runnable task, T result)`**== | 包装后的，返回的是 `Callable<T>` 对象，`get()` 返回指定的 `result` 对象 |

### 三、使用示例

```java
// 方式一：直接使用 Callable
Callable<String> callable = () -> {
    return "任务执行结果";
};

// 方式二：将 Runnable 包装成 Callable，执行完后返回 null
Callable<Object> callable1 = Executors.callable(() -> {
    // 执行任务，无返回值
});

// 方式三：将 Runnable 包装成 Callable，执行完后返回指定结果
Callable<String> callable2 = Executors.callable(() -> {
    // 执行任务，无返回值
}, "默认返回结果");
```

### 四、面试要点

| 问题                                       | 回答                                                                                                  |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Runnable 和 Callable 的核心区别**                | Runnable **无返回值**，Callable **有返回值**且**可以抛出受检异常**                                                    |
| **如何将 Runnable 转成 Callable**                 | ==**`Executors.callable(Runnable task)`**== 或 ==**`Executors.callable(Runnable task, T result)`**== |
| **`callable(Runnable task)` 的返回值**           | 返回 `Callable<Object>`，`get()` 返回 **`null`**                                                         |
| **`callable(Runnable task, T result)` 的返回值** | 返回 `Callable<T>`，`get()` 返回**指定的 `result` 对象**                                                      |
