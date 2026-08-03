**BGSAVE 与 BGREWRITEAOF 的调度机制**

### 一、为什么要延迟执行？

BGSAVE 和 BGREWRITEAOF 都会创建子进程来执行磁盘写入操作。如果两个子进程同时运行，会引发两个严重问题：

1. **磁盘 I/O 竞争**：两个子进程同时大量写入磁盘，导致磁盘 I/O 负载飙升，性能急剧下降。
    
2. **内存翻倍风险**：`fork` 创建子进程时，操作系统使用写时复制机制。两个子进程同时存在，任何写操作都会触发两份内存拷贝，内存压力巨大。
    

因此，Redis 设计了互斥机制：**同一时间最多只有一个子进程在执行持久化操作**。

### 二、延迟执行的触发条件

当服务器正在执行 BGSAVE 命令（子进程正在生成 RDB 文件）期间，如果客户端发来 BGREWRITEAOF 命令：

- 服务器**不会直接拒绝**该命令，而是将其标记为**延迟执行**。
    
- 具体做法：将服务器状态的 ==`aof_rewrite_scheduled`== 属性设为 1，表示"有一个 BGREWRITEAOF 在排队等待执行"。
    

### 三、延迟执行的触发时机

每次 `serverCron` 函数执行时，都会检查以下三个条件是否同时满足：

|条件|检查方式|含义|
|---|---|---|
|==**BGSAVE 没有在执行**==|`server.rdb_child_pid == -1`|RDB 子进程已结束|
|==**BGREWRITEAOF 没有在执行**==|`server.aof_child_pid == -1`|AOF 重写子进程已结束|
|==**延迟标志已设置**==|`server.aof_rewrite_scheduled == 1`|有 BGREWRITEAOF 在排队|

当以上三个条件全部满足时，说明当前没有任何持久化子进程在运行，且之前有一个 BGREWRITEAOF 被延迟了，此时服务器就会执行之前被推迟的 BGREWRITEAOF 命令。

### 四、另一种情况：BGREWRITEAOF 期间收到 BGSAVE

反过来，如果服务器正在执行 BGREWRITEAOF 期间，客户端发来 BGSAVE 命令，服务器会**直接拒绝**执行 BGSAVE，而不是延迟。因为 BGSAVE 可以由 `serverCron` 根据 `save` 配置自动触发，不需要像 BGREWRITEAOF 那样进行排队。

### 五、状态判断完整逻辑

**客户端发来 BGREWRITEAOF 命令**
         **↓**
**检查：是否有子进程正在执行？**
         **↓**
    **├── BGSAVE 正在执行 → 设置 `aof_rewrite_scheduled` = 1（延迟等待）**
    **│**
    **├── BGREWRITEAOF 正在执行 → 直接拒绝，返回错误**
    **│**
    **└── 无子进程在执行 → 立即执行 BGREWRITEAOF**

**`serverCron` 每次执行**
         **↓**
**检查：**
  **① `rdb_child_pid` == -1？**
  **② `aof_child_pid` == -1？**
  **③ `aof_rewrite_scheduled` == 1？**
         **↓**
   **全部满足 → 执行 BGREWRITEAOF，重置`aof_rewrite_scheduled` = 0**