**定义与生命周期**

- Daemon 线程是**支持性线程**，主要负责后台调度和支持工作。
    
- **关键特征**：当 Java 虚拟机中**只剩下 Daemon 线程在运行时，JVM 会直接退出**。这意味着 Daemon 线程的生命周期是依附于非 Daemon 线程的，它本身无法阻止 JVM 关闭。
    

**2. 设置规则（铁的纪律）**

- **必须在 `start()` 之前设置**：通过 `Thread.setDaemon(true)` 设置，如果线程已经启动再设置，会抛出 `IllegalThreadStateException`。
    
- **新线程继承父线程属性**：在 Daemon 线程中创建的新线程，**默认也是 Daemon 线程**。同理，在非 Daemon 线程中创建的线程，默认也是非 Daemon。
    

**3. 最危险的坑：`finally` 块并不完全可靠**  

- **现象**：当 JVM 中只剩下 Daemon 线程时，JVM 会立即退出，**由于Daemon 线程的存在导致jvm会不经过finally模块前提前结束程序，这样会导致无法执行 `finally` 块中的代码，以造成资源的泄露**。
    
- **原因**：JVM 退出是靠所有非 Daemon 线程结束触发的，这个过程不会等待 Daemon 线程优雅收尾。
    
- **结论**：**不要把释放关键资源、关闭连接、写日志等核心收尾逻辑，单独寄托在 Daemon 线程的 `finally` 上。**