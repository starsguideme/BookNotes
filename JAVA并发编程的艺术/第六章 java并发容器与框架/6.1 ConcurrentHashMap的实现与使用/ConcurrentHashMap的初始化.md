ConcurrentHashMap的初始化是通过intitalCapacity,loadFactor和concurrencyLevel等几个参数来初始化Segment数组，段偏移量segmentShift，段掩码segmentMask和每个Segment的HashEntry数组来实现的.
-  [[初始化Segment数组]]: 
- [[初始化segmentShift和segmentMask]] 
- [[定位Segment]].