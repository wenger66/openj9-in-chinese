# 全局垃圾回收

在堆锁分配中发生分配故障，或者发生对 System.gc() 的特定调用时，执行垃圾回收。发生分配故障或 System.gc() 调用的线程控制并执行垃圾回收。

垃圾回收的第一步是获取虚拟机的唯一控制权，以阻止任何进一步的 Java™ 操作。然后，垃圾回收将经历三个阶段：标记、清扫和整理（如果需要）。垃圾收集器 (GC) 所执行的是一种停止执行所有应用程序 (STW) 的操作，因为在回收垃圾时将停止所有应用程序线程。

在使用均衡的垃圾回收策略这种特殊的情况下，才会发生全局垃圾回收。 可能导致这种稀有事件发生的情况包括：
* 调用 System.gc()
* 工具请求垃圾回收
* 将堆的大小、占用的堆内存和供不应求的收集率相结合。


Garbage collection is performed when an allocation failure occurs in heap lock allocation, or if a specific call to System.gc() occurs. The thread that has the allocation failure or the System.gc() call takes control and performs the garbage collection.

The first step in garbage collection is to acquire exclusive control on the Virtual machine to prevent any further Java™ operations. Garbage collection then goes through the three phases: mark, sweep, and, if required, compaction. The Garbage Collector (GC) is a stop-the-world (STW) operation, because all application threads are stopped while the garbage is collected.

A global garbage collection occurs only in exceptional circumstances when using the Balanced Garbage Collection policy. Circumstances that might cause this rare event include:
A System.gc() call.
A request by tooling.
A combination of heap size, occupied heap memory, and collection rates that cannot keep up with demand.

