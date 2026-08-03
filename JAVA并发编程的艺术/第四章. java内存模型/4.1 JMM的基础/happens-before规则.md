**happens-before 规则**

### 一、什么是 happens-before

happens-before（先行发生）是 JMM 定义的规则，用于判断两个操作之间是否存在可见性保证。如果一个操作 A happens-before 操作 B，那么操作 A 的结果对操作 B **可见**。

==**happens-before 同时保证可见性和有序性**==——它不仅确保 A 的结果对 B 可见，也确保 A 在 B 之前执行。happens-before 是 JMM 最核心的规则体系，所有同步机制（`volatile`、`synchronized`、`Lock`、`Thread.start()`、`Thread.join()` 等）最终都转化为 happens-before 关系来保证内存可见性。

### 二、happens-before 的六条核心规则

| 规则                    | 说明                                                                                                                                |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| ==**程序次序规则**==        | 一个线程中的每个操作，happens-before 于该线程中的任意后续操作                                                                                            |
| ==**监视器锁规则**==        | 对一个锁的解锁，happens-before 于随后对这个锁的加锁                                                                                                 |
| ==**volatile 变量规则**== | 对一个 `volatile` 域的写操作，happens-before 于任意后续对这个 `volatile` 域的读操作                                                                     |
| ==**传递性**==           | 如果 A happens-before B，且 B happens-before C，则 A happens-before C                                                                   |
| ==**`start()` 规则**==  | 如果线程 A 执行 `ThreadB.start()`，那么线程 A 的 `start()` 操作 happens-before 于线程 B 中的任意操作                                                     |
| ==**`join()` 规则**==   | 如果线程 A 执行 `ThreadB.join()` 并成功返回，那么线程 B 中的任意操作 happens-before 于线程 A 从 `join()` 的返回，其中线程的join()操作是一个栈的方式返回的，详见于[[thread.join()函数]] |
|                       |                                                                                                                                   |

### 三、happens-before 与 JMM 的关系

happens-before 规则可以理解为一个层级结构：
**用户/程序员**
    **↓ 编写同步代码**
**happens-before 规则（父节点）**
    **↓ 定义可见性语义**
**JMM 的具体实现（子节点）**
    **↓ 通过内存屏障、锁内存语义等底层机制落实**
**JMM 的抽象定义（叶子节点）**
    **↓ 最终保证**
**线程间共享变量的可见性与有序性**

### 四、面试要点

| **问题**                       | 回答                                                                  |
| ---------------------------- | ------------------------------------------------------------------- |
| **happens-before 保证什么**      | 同时保证可见性和有序性——如果 A happens-before B，A 的结果对 B 可见，且 A 的执行顺序在 B 之前      |
| **happens-before 的六条规则**     | 程序次序规则、监视器锁规则、volatile 规则、传递性、start() 规则、join() 规则                  |
| **happens-before 与 JMM 的关系** | happens-before 是 JMM 的"接口规范"，定义了程序员可以依赖的可见性保证；内存屏障和锁语义是 JMM 的"底层实现" |
