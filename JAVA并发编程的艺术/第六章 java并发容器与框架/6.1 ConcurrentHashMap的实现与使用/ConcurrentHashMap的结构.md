

### 一、核心数据结构

JDK 7 中的 `ConcurrentHashMap` 是由 **`Segment` 数组结构**和 **`HashEntry` 数组结构**组成的。

| 组件 | 说明 |
|------|------|
| ==**`Segment`**== | 一种**可重入锁（`ReentrantLock`）**，负责守护一段哈希表 |
| ==**`HashEntry`**== | 一种 **K-V 数据结构**，用于存储键值对 |

一个 `Segment` 包含了一个 `HashEntry` 数组，每个 `HashEntry` 是一个**链表性**的数据结构元素。每个 `Segment` 守护着一个 `HashEntry` 数组，当想要修改 `HashEntry` 数组时，就得先获得与之对应的 `Segment` 锁。

### 二、分段锁的结构图

```text
ConcurrentHashMap
    │
    ├── Segment[0] (ReentrantLock)
    │       └── HashEntry[ ] → HashEntry → HashEntry (链表)
    │
    ├── Segment[1] (ReentrantLock)
    │       └── HashEntry[ ] → HashEntry → HashEntry (链表)
    │
    ├── Segment[2] (ReentrantLock)
    │       └── HashEntry[ ] → HashEntry → HashEntry (链表)
    │
    └── ...
```

### 三、分段锁的核心思想

**分段锁的本质**：将整个哈希表分成多个段（Segment），每个段独立加锁。不同段之间的修改操作可以**并发执行**，从而提升并发性能。

**锁粒度**：`Segment` 的个数就是并发度（`concurrencyLevel`），默认是 16。这意味着最多可以同时有 16 个线程并发修改 `ConcurrentHashMap`。

### 四、ConcurrentHashMap 与 HashMap、Hashtable 的对比

| 维度 | HashMap | Hashtable | ConcurrentHashMap（JDK 7） |
|------|---------|----------|--------------------------|
| ==**线程安全**== | ❌ 不安全 | ✅ 安全 | ✅ 安全 |
| ==**锁机制**== | 无锁 | 整个表一把锁（`synchronized`） | **分段锁**（`Segment` + `ReentrantLock`） |
| ==**并发度**== | 无 | 低（同一时刻只能有一个线程操作） | 较高（默认 16 个 Segment 可同时操作） |
| ==**性能**== | 高（单线程） | 低（多线程） | 较高（多线程） |

### 五、面试要点

| 问题                                | 回答                                                           |
| --------------------------------- | ------------------------------------------------------------ |
| **JDK 7 中 ConcurrentHashMap 的核心数据结构** | ==**Segment 数组 + HashEntry 数组**==，Segment 是锁，HashEntry 存储键值对 |
| **分段锁的核心思想**                          | 将整个表分成多个段，**每个段独立加锁**，不同段之间可以并发操作                            |
| **Segment 是什么类型的锁**                   | ==**ReentrantLock**==，可重入锁                                   |
| **并发度是什么意思**                          | `Segment` 的个数，默认 **16**，最多同时有 16 个线程并发修改                     |
| **与 Hashtable 的区别**                   | Hashtable 是整个表一把锁，ConcurrentHashMap 是**分段锁**，并发度更高           |
