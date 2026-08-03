### 一、ThreadLocal 是什么

ThreadLocal 也叫线程本地变量，它提供了一种**线程级别的变量隔离机制**。每个线程都可以通过同一个 `ThreadLocal` 对象，**独立地**存取属于自己的变量副本，线程之间互不干扰。

它的核心存储结构是：以 `ThreadLocal` 对象为 Key，以任意 `Object` 为 Value 的键值对。这个键值对**不是存储在 `ThreadLocal` 内部**，而是存储在每个线程自己的 `ThreadLocalMap` 中。因为本质上是一个 Map 结构，通过哈希定位，所以存取速度很快。

### 二、ThreadLocal 的基本使用

通过 `set(T value)` 方法可以在当前线程中设置一个值，之后在当前线程中调用 `get()` 方法就可以获取到之前设置的那个值。


ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("hello");          // 设置值
String value = threadLocal.get();  // 获取值，返回 "hello"
threadLocal.remove();              // 使用完毕后移除，防止内存泄漏

不同线程通过同一个 `ThreadLocal` 对象存取的数据是彼此隔离的。线程 A `set("A")` 之后 `get` 到的是 `"A"`，线程 B `get` 到的是 `null`，除非 B 也 `set` 了自己的值。

### 三、底层存储结构

每个线程 `Thread` 内部维护了一个 `ThreadLocalMap`，这个 Map 的 Entry 以 `ThreadLocal` 对象为 Key，以用户设置的值为 Value。

Thread A
  └── ThreadLocalMap
        ├── Entry<ThreadLocal1, "valueA">
        └── Entry<ThreadLocal2, 123>
Thread B
  └── ThreadLocalMap
        ├── Entry<ThreadLocal1, "valueB">
        └── Entry<ThreadLocal2, 456>

**执行流程**：调用 `threadLocal.set(value)` 时，会先获取当前线程的 `ThreadLocalMap`，然后将 `(this, value)` 存入这个 Map。调用 `threadLocal.get()` 时，也是从当前线程的 `ThreadLocalMap` 中，以自己为 Key 查找对应的 Value。

### 四、典型应用：Profiler 耗时统计

一个常用的场景是性能分析，比如 `Profiler` 类，用来统计方法的执行耗时：

public class Profiler {
    private static final ThreadLocal<Long> TIME_THREADLOCAL = new ThreadLocal<>();
    public static void begin() {
        TIME_THREADLOCAL.set(System.currentTimeMillis());
    }
    public static long end() {
        return System.currentTimeMillis() - TIME_THREADLOCAL.get();
    }
}

`begin()` 方法记录当前时间戳存入 `ThreadLocal`，`end()` 方法取出时间戳并计算时间差。由于 `ThreadLocal` 天然是线程隔离的，多个线程同时调用 `begin()` 和 `end()` 也不会相互覆盖数据。

在实际项目中，可以通过 AOP 方式对需要监控的方法统一埋点：


@Around("@annotation(com.example.Monitor)")
public Object profile(ProceedingJoinPoint pjp) throws Throwable {
    Profiler.begin();
    Object result = pjp.proceed();
    long elapsed = Profiler.end();
    System.out.println("方法耗时: " + elapsed + "ms");
    return result;
}

### 五、`ThreadLocalMap` 的 Entry 设计与内存泄漏

`ThreadLocalMap` 的 Entry 继承了 `WeakReference`，它的 Key（即 `ThreadLocal` 对象）是**弱引用**，而 Value（用户设置的值）是**强引用**。

弱引用的含义是：==**如果 `ThreadLocal` 对象在外部没有其他强引用指向它，下次 GC 时它就会被回收。**== 当 Key 被回收变成 `null` 后，Value 却依然存在一条强引用链（`Thread → ThreadLocalMap → Entry → Value`）。如果线程长时间运行（如线程池中的线程），这些 Key 为 `null` 的 Entry 就会一直占用内存，导致**内存泄漏**。

因此，使用 `ThreadLocal` 后，务必在不再需要时主动调用 `remove()` 方法，它会同时清理掉 Key 和 Value，从根本上避免内存泄漏。每次使用 `set` 或 `get` 操作时，`ThreadLocalMap` 也会顺便清理一些 Key 为 `null` 的脏 Entry，但这个机制并不完全可靠，主动调用 `remove()` 仍然是最好的习惯。

### 六、面试要点

**Q：`ThreadLocal` 的实现原理是什么？**

每个线程内部维护了一个 `ThreadLocalMap`，它的 Key 是 `ThreadLocal` 对象，Value 是用户设置的值。`get()` 和 `set()` 操作都是通过当前线程找到这个 Map，再以 `ThreadLocal` 对象为 Key 进行存取。正是因为存储结构在每个线程内部，所以天然实现了线程隔离。

**Q：`ThreadLocal` 为什么会导致内存泄漏？**

`ThreadLocalMap` 的 Entry 中，Key 是弱引用，Value 是强引用。当 `ThreadLocal` 对象在外部的强引用被清除后，GC 会回收 Key，但 Value 仍然存在强引用链。如果线程一直存活（如线程池线程），这些 Key 为 `null` 的 Value 就永远不会被回收，导致内存泄漏。解决方案是使用完毕后主动调用 `remove()`。

**Q：为什么 Entry 的 Key 要设计成弱引用？**

因为如果 Key 是强引用，只要线程还活着，`ThreadLocalMap` 中的 `ThreadLocal` 对象就永远不会被 GC 回收。即使业务代码中已经没有任何地方引用这个 `ThreadLocal` 了，内存依然无法释放。设计成弱引用后，一旦外部不再使用该 `ThreadLocal`，它就会被 GC 回收，减少了内存泄漏的风险。