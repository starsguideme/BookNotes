### 一、`synchronized` 的三种锁对象

|使用形式|锁对象|说明|
|---|---|---|
|==**普通同步方法**==|当前**实例对象**（`this`）|同一实例的多线程访问会被同步，不同实例互不影响|
|==**静态同步方法**==|当前类的 **`Class` 对象**|所有实例共享同一把类锁|
|==**同步块**==|`synchronized(对象)` 中**括号内指定的对象**|可以指定任意对象作为锁，粒度更灵活|

### 二、锁的本质与存储位置

当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。那么锁到底存在哪里呢？

锁的信息存储在 **[[Java 对象头]]（Object Header）** 中，具体在对象头的 **Mark Word** 部分。Mark Word 中存储了对象的哈希码、分代年龄、**锁标志位**和**指向 Monitor 的指针**等信息。

### 三、Monitor 对象与底层指令

在 JVM 层面，`synchronized` 的实现依赖于 **Monitor（监视器）** 对象。每个 Java 对象在 JVM 内部都关联着一个 Monitor，这个 Monitor 负责管理锁的获取、释放以及等待队列。

`synchronized` 的两种使用形式，在字节码层面的实现方式不同：

|使用形式|底层实现|说明|
|---|---|---|
|==**同步块**==|`monitorenter` 和 `monitorexit` 指令|进入同步块时执行 `monitorenter`，退出时执行 `monitorexit`（正常退出和异常退出各有一个）|
|==**同步方法**==|方法修饰符上的 `ACC_SYNCHRONIZED` 标志位|JVM 在方法调用时检查该标志，如果有则自动进入 Monitor|

但无论哪种形式，本质都是对一个对象的 Monitor 进行获取。这个获取是**排他**的——同一时刻只有一个线程能持有该 Monitor，没有获取到的线程会进入 **BLOCKED** 状态，在 Monitor 的同步队列中等待。

### 四、从 Java 代码到底层实现的完整链路


**Java 层面：synchronized 修饰方法或同步块**
        **↓**
**字节码层面：**
  **├── 同步块 → monitorenter / monitorexit 指令**
  **└── 同步方法 → ACC_SYNCHRONIZED 标志位**
        **↓**
**JVM 层面：获取对象关联的 Monitor**
        **↓**
**Monitor 内部：**
  **├── 持有锁的线程（Owner）**
  **├── 同步队列（EntryList，存放 BLOCKED 状态的线程）**
  **└── 等待队列（WaitSet，存放调用 wait() 的线程）**
        **↓**
**CPU 层面：通过 CAS 或锁定指令保证 Monitor 状态的原子性更新**

### 五、面试要点

**Q：`synchronized` 的锁信息存在哪里？**

存在 Java 对象头中的 **Mark Word** 里。Mark Word 会根据锁的状态（无锁、偏向锁、轻量级锁、重量级锁）不同，存储不同的信息——无锁时存哈希码和分代年龄，重量级锁时存指向 Monitor 的指针。

**Q：`monitorenter` 和 `monitorexit` 为什么通常配对出现？**

每个 `monitorenter` 会对应两个 `monitorexit`——一个用于正常退出同步块，一个用于异常退出。这是为了确保无论同步块中的代码是否抛出异常，锁都能被正确释放，避免线程永久持有锁导致死锁。

**Q：`synchronized` 和 `volatile` 的底层实现有什么区别？**

`synchronized` 依赖 JVM 层面的 Monitor 机制，涉及线程的阻塞和唤醒，会产生上下文切换。`volatile` 依赖 CPU 层面的 Lock 前缀指令和缓存一致性协议，不阻塞线程，没有上下文切换。两者解决的问题也不同：`synchronized` 解决原子性+可见性，`volatile` 只解决可见性