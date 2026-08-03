

### 一、两次哈希的流程

由于ConcurrentHashMap使用分段锁的`Segment`来保护不同的数据，因此在插入和获取元素时，必须先通过**哈希算法定位到Segment**，然后再一次使用 **Wang/Jenkins hash的变种算法**对哈希码再次进行散列计算。

```text
两次哈希的完整流程：

① 计算 key 的 hashCode
    ↓
② 第一次哈希：通过 Wang/Jenkins hash 变种算法进行散列
    ↓
③ 第二次哈希：取 hash 值的高 segmentShift 位，通过 (hash >>> segmentShift) & segmentMask 定位到 Segment
    ↓
④ 在 Segment 内部，通过 (hash & (table.length - 1)) 定位到具体的 HashEntry
```

### 二、为什么要进行两次哈希？

之所以要进行两次散列算法，核心目的是**减少哈希冲突**，使元素能够均匀地分布在不同的`Segment`上，从而提高容器的存取效率。

| 哈希阶段 | 作用 |
|---------|------|
| ==**第一次哈希（Wang/Jenkins hash）**== | 对原始的 hashCode 进行再散列，让高位和低位都参与运算，减少冲突 |
| ==**第二次哈希（Segment 定位）**== | 用 hash 值的高位定位 Segment，让数据均匀分布到各个 Segment 上 |

### 三、面试要点

| **问题**         | 回答                                          |
| ---------- | ------------------------------------------- |
| **为什么需要两次哈希**  | ==**减少哈希冲突**==，使元素均匀分布在不同的 Segment 上，提高存取效率 |
| **第一次哈希用什么算法** | ==**Wang/Jenkins hash 变种算法**==              |
| **第二次哈希的作用**   | 用高位定位 Segment，让数据均匀分布到各个 Segment 上          |
。