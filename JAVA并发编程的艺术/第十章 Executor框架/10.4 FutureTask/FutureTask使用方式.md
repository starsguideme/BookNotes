
### 一、三种使用方式

| 方式 | 说明 |
|------|------|
| ==**交给执行器执行**== | 把 `FutureTask` 任务交给 `ExecutorService` 执行 |
| ==**通过 `submit` 方法返回**== | 通过 `submit` 方法返回一个 `FutureTask` 任务，然后执行 `get` 或 `cancel` 方法 |
| ==**单独使用 FutureTask**== | 不依赖线程池，直接创建 `FutureTask` 并启动线程执行 |

### 二、适用场景

当一个线程需要等待另一个线程的任务执行完之后才可以继续执行时，可以使用 `FutureTask`。假设有多个线程执行若干个任务，由于 `FutureTask` 处于**无界队列**中，因此其**只能被执行一次**。如果多个线程试图同时执行同一个任务时，也只允许**一个线程**执行该任务，其他线程必须等待这个任务执行完之后才能继续执行。

### 三、代码示例

```java
// 创建一个 FutureTask 任务
FutureTask<String> futureTask = new FutureTask<>(() -> {
    Thread.sleep(2000);
    return "任务执行结果";
});

// 方式一：交给执行器执行
ExecutorService executor = Executors.newFixedThreadPool(3);
executor.submit(futureTask);

// 方式二：单独使用 FutureTask，启动线程执行
new Thread(futureTask).start();

// 方式三：等待任务完成
try {
    String result = futureTask.get();  // 阻塞等待，直到任务完成
    System.out.println("任务结果：" + result);
} catch (Exception e) {
    futureTask.cancel(true);  // 出现异常时取消任务
}
```

### 四、面试要点

| **问题**                         | 回答                                           |
| -------------------------- | -------------------------------------------- |
| **FutureTask 有哪三种使用方式**        | ==**交给执行器执行、通过 submit 返回、单独使用 FutureTask**== |
| **多个线程同时执行同一个 FutureTask 会怎样** | 只有一个线程能执行该任务，其他线程**必须等待**该任务执行完              |
| **`get()` 方法的作用**              | **阻塞当前线程**，直到任务完成并返回结果                       |
| **何时使用 `cancel` 方法**           | 当任务出现异常或需要**提前终止**任务时                        |