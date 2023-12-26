- -Xmx

- -Xms

- -Xss。

  - 线程栈空间大小。
  - 实例：-Xss1m。表示分配1m空间给每个线程使用。

- -server

- -XX:+DoExcapeAnalysis

  - 逃逸分析。需在-server 模式下使用。

  - -XX:+DoExcapeAnalysis启用，-XX:-DoExcapeAnalysis禁用。默认禁用

- -XX:+PrintGC

  - 打印GC日志

- -XX:+EliminateAllocations

  - 标量替换（默认打开）。作用是允许对象打散分配在栈上。
  - 比如对象拥有id和name两个字段，那么这两个字段会被视为两个独立的局部变量进行分配。

- -XX:-UserTLAB（关闭TLAB）

  - TLAB(Thread Local Allocation Buffer)，线程本地分配缓冲区。
  
    

