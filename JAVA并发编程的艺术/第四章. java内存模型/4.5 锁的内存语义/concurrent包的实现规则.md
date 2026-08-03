

### 一、四种线程通信方式

由于 CAS 同时具有 volatile 写与读操作的内存语义，因此 Java 中的线程通信共有四种模式：

| 通信模式                                | 说明                                                             |
| ----------------------------------- | -------------------------------------------------------------- |
| ==**A 写 volatile → B 读 volatile**== | A 写入 volatile 变量后，B 读取该变量，能立即看到 A 写入的最新值                       |
| ==**A 写 volatile → B 写 volatile**== | A 写入 volatile 变量后，B 基于这个新值进行后续的写操作                             |
| ==**A CAS 更新 → B CAS 更新**==         | A 通过 CAS 原子更新一个 volatile 变量，B 再通过 CAS 原子更新同一个 volatile 变量      |
| ==**A CAS 更新 → B 读 volatile**==     | A 通过 CAS 原子更新一个 volatile 变量后，B 读取该 volatile 变量，能立即看到 A 更新后的最新值 |

### 二、JUC 的通用通信模式

在并发包中，JUC 的通用做法是：

1.  将共享变量声明为 `volatile` 修饰的变量，保证其可见性。
2.  通过 CAS 执行原子操作来更新线程状态，并以此完成同步操作（被 volatile 修饰的变量对其他共享线程具有可见性）。
3.  结合 volatile 的写/读操作与 CAS 具有的 volatile 写-读内存语义，来完成线程之间的通信。

### 三、三个核心基础类

AQS、非阻塞数据结构、原子变量类——这些 JUC 中的基础类都是使用这种模式来实现的：

| 基础类 | volatile 变量 | CAS 操作 |
|--------|-------------|---------|
| ==**AQS**== | `volatile int state` | `compareAndSetState()` 原子更新同步状态 |
| ==**原子变量类**==（`AtomicInteger` 等） | `volatile int value` | `compareAndSet()` 原子更新值 |
| ==**非阻塞数据结构**==（`ConcurrentLinkedQueue` 等） | `volatile` 修饰的节点引用 | `compareAndSetNext()` 原子更新指针 |

AQS、非阻塞数据结构和原子变量类，本质上都是对 **volatile 的读与写 + CAS 操作** 的封装。它们通过 volatile 保证可见性，通过 CAS 保证原子性更新，从而在不使用互斥锁的前提下实现高效的线程通信和同步。

### 四、面试要点

| 问题 | 回答 |
|------|------|
| JUC 中线程通信有哪四种方式 | A 写 volatile → B 读/写 volatile；A CAS 更新 → B CAS 更新/读 volatile |
| JUC 的通用通信模式 | volatile 保证可见性 + CAS 保证原子性更新，两者配合完成线程通信 |
| AQS 如何应用这个模式 | `volatile int state` 记录同步状态，`compareAndSetState()` 原子更新 |