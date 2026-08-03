**Java 线程的 `join()` 方法**

### 一、`join()` 的作用

当线程执行了 `thread.join()` 方法后，其含义是：当前线程会**阻塞等待**，直到 `thread` 线程执行完毕（终止）之后，才从 `join()` 方法中返回并继续执行。

除了 `join()` 方法之外，线程还提供了两个具备超时特性的方法：

| 方法                                     | 说明                                    |
| -------------------------------------- | ------------------------------------- |
| ==`join()`==                       | 无限期等待，直到目标线程终止                        |
| ==`join(long millis)`==            | 最多等待 `millis` 毫秒，超时后强制返回              |
| ==`join(long millis, int nanos)`== | 最多等待 `millis` 毫秒 + `nanos` 纳秒，超时后强制返回 |

### 二、`join()` 的底层实现

`join()` 的底层并不是类似栈的数据结构，而是基于[[等待和通知机制]]实现的。核心源码逻辑如下：

public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;
    if (millis == 0) {
        // 无限等待，直到目标线程终止
        while (isAlive()) {
            wait(0);
        }
    } else {
        // 超时等待
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;  // 超时，强制返回
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}

**关键解读**：

- **`join()` 方法本身是 `synchronized` 修饰的**，锁对象是目标线程对象本身。
    
- 它的核心逻辑是：==**只要目标线程还活着（`isAlive()` 返回 `true`），当前线程就调用 `wait()` 进入等待状态。**==
    
- 当目标线程终止时，JVM 会自动调用 `this.notifyAll()` 唤醒所有在该线程对象上等待的线程（这是 JVM 的内部机制，不需要程序员手动调用）。
    
- 这也是为什么 `join()` 不叫 `waitForThread` 的原因—**—它其实就是 `wait` 在目标线程对象上的应用。**
    

### 三、`join()` 的执行流程


==**主线程调用 thread.join()**
        **↓**
**主线程获取 thread 对象的锁（join 方法是 synchronized 的）**
        **↓**
**循环检查 thread.isAlive()**
        **↓**
    **┌── 活着 → 调用 thread.wait()，主线程进入等待状态，释放锁**
    **│         ↓**
    **│   thread 线程继续执行**
    **│         ↓**
    **│   thread 执行完毕，JVM 调用 thread.notifyAll()**
    **│         ↓**
    **│   主线程被唤醒，重新获取锁**
    **│         ↓**
    **│   再次检查 isAlive()**
    **│         ↓**
    **└── 已终止 → 退出循环，从 join() 返回，主线程继续执行**==

### 四、`join()` 与等待/通知机制的关系

`join()` 是等待/通知机制的一个**封装应用**：

| 角色   | 对应                             |
| ---- | ------------------------------ |
| **等待方**  | **调用 `thread.join()` 的线程**         |
| **锁对象**  | **目标 `thread` 对象本身**               |
| **等待条件** | **`thread.isAlive()` 返回 `false`**  |
| **通知时机** | **JVM 在目标线程终止时自动调用 `notifyAll()`** |

### 五、面试要点

**Q：`join()` 为什么要用 `while (isAlive())` 循环而不是 `if`？**

和 `wait()` 必须用 `while` 检查条件一样，为了防止虚假唤醒。如果线程被意外唤醒，而目标线程还没结束，必须再次等待。

**Q：`join()` 和 `wait()` 有什么关系？**

`join()` 本质上就是对 `wait()` 的封装。调用 `thread.join()` 等价于让当前线程在 `thread` 对象上调用 `wait()`，等待 JVM 在目标线程结束后触发 `notifyAll()`。

**Q：`join()` 和 `sleep()` 的区别？**

`join()` 等待的是**目标线程终止**这个事件，而 `sleep()` 等待的是**固定时间**。`join()` 底层用的是 `wait()` 机制，会释放锁；`sleep()` 不释放锁。