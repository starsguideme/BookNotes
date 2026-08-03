
### 一、初始化过程

在 ConcurrentHashMap 的构造函数中，`segmentShift` 和 `segmentMask` 的初始化依赖以下步骤：

```text
① 传入 concurrencyLevel（默认 16）
    ↓
② 计算 ssize：大于或等于 concurrencyLevel 的最小 2 的幂次方值
   例如：concurrencyLevel = 16 → ssize = 16 (1 << 4)
    ↓
③ 计算 sshift：ssize 从 1 向左移位的次数
   例如：16 = 1 << 4 → sshift = 4
    ↓
④ 计算 segmentShift = 32 - sshift
   例如：32 - 4 = 28
    ↓
⑤ 计算 segmentMask = ssize - 1
   例如：16 - 1 = 15 (二进制 1111，低 4 位全为 1)
```

### 二、两个变量的初始化公式

| 变量 | 计算方式 | 示例值 | 作用 |
|------|---------|--------|------|
| ==**`ssize`**== | 大于等于 `concurrencyLevel` 的最小 2 的幂次方 | 16 | Segment 数组的实际长度 |
| ==**`sshift`**== | `ssize` 从 1 向左移位的次数 | 4 | 用于计算 `segmentShift` |
| ==**`segmentShift`**== | `32 - sshift` | 28 | 决定 hash 值的高多少位用于定位 Segment |
| ==**`segmentMask`**== | `ssize - 1` | 15（二进制 1111） | 位运算掩码，用于取模运算 `hash & mask` |

### 三、Segment 定位中使用这两个变量

```text
Segment 定位公式：
segmentIndex = (hash >>> segmentShift) & segmentMask

示例（默认值）：
hash >>> 28  → 取 hash 值的高 4 位
& 15          → 取模 16，定位到 0~15 号 Segment
```

### 四、面试要点

| 问题                                | 回答                                                      |
| --------------------------------- | ------------------------------------------------------- |
| **`segmentShift` 怎么计算**               | ==**`32 - sshift`**==，其中 `sshift` 是 `ssize` 从 1 向左移位的次数 |
| **`segmentMask` 怎么计算**                | ==**`ssize - 1`**==，用于位运算取模                             |
| **为什么 `segmentShift` 用 32**           | ConcurrentHashMap 的 hash 算法输出的最大位数是 **32 位**            |
| **`ssize` 一定等于 `concurrencyLevel` 吗** | 不一定，`ssize` 是**大于等于** `concurrencyLevel` 的最小 2 的幂次方     |
