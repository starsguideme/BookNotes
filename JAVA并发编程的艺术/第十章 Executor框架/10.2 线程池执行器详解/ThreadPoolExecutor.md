其是执行器这个框架的核心类，其是线程池的实现工具，其内部有四种东西
- corePool:核心线程池的大小
- maximumPool: 最大线程池的大小
- BlockingQueue:用来暂时保存任务的工作队列
- RejectedExecutionHandler:已经关闭或线程池执行器已经饱和时，execute方法会调用Handler
- 线程池执行器根据线程数的不同可以分为
   - [[FixedThreadPool]]
   - [[SingleThreadExecutor]]
   - [[CacheThreadPool]]