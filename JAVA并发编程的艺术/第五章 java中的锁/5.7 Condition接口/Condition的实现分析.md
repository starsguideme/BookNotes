ConditionObject实现了Condition接口，因为Condition的操作需要获取相关的锁，因此ConditionObject以同步器AQS内部类的形式存在着，每个Condition对象都包含一个等待队列，以下是队列实现等待/通知的方式
- [[等待队列]]
- [[等待]]
- [[通知]]
