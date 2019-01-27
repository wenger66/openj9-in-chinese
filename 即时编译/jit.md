
# 即时编译

即时编译器（JIT，下文直接使用JIT指代即时编译）是OpenJ9虚拟机（JVM，下文直接使用JVM指代OpenJ9虚拟机）
一个关键的组件，它可以在程序运行时将字节码
(操作系统无关的抽象)
编译为机器码，以改进程序的性能。
如果禁用JIT，JVM将逐条将字节码翻译为机器码并执行，解释期间会占用额外的CPU和内存，会拖慢应用的运行速度。

启用JIT的情况下，程序运行到每个方法，JVM会直接调用该方法的已编译代码，而不是对代码进行解释。
理论上，如果没有编译，就不需要占用CPU和内存，那么运行每个方法都可能使Java程序速度接近于应用程序的速度。

JIT并不会在第一次调用方法，在调用任何方法都进行编译。在JVM 
首次启动时，将调用数千种方法，即使通过JIT最终实现了较高的峰值性能，但编译所有这些方法也会对启动时间产生显著影响。

JVM
会保留一个方法调用计数，初始为预定义的即时编译阈值，每次调用方法时递减。当调用计数达到0
时，就触发方法的即时编译。只要方法被即时编译过，再次执行该方法时，JVM就不会解释而是直接调用已编译的字节码。

## 优化级别
JIT编译器可以在不同的优化级别编译方法，级别包括：cold、warm、hot、veryHot（带profiling）
 或 scorching，可以参考[-Xjit](../命令行参数/JVM-X参数/-Xjit)。
 优化级别越高，预期的性能也越高，但它们在CPU和内存方面的性能开销也越高。 
 
 * cold：在大型应用程序启动时会降级到cold，以缩短启动时间。
 * warm：方法的初始或缺省优化级别为warm，在启动成功后，大部分方法会因为达到编译阈值而被即时编译。

JIT向更高级别级别进化时，需要使用一个采样线程，该线程会定期唤醒并确定哪些方法更频繁出现在堆栈顶部。
（拿不准如何翻译）Methods that consume more than 1% are compiled at hot. Methods that 
consume more than 12.5% are scheduled for a scorching compilation. However, before that happens the methods are compiled at very hot with profiling to collect detailed profile data that is used by the scorching compilation.


（拿不准如何翻译）The higher optimization levels use special techniques such as escape 
analysis and partial redundancy elimination, or loop through certain optimization sequences more times. Although these techniques use more CPU and memory, the improved performance that is delivered by the optimizations can make the tradeoff worthwhile.

## 故障诊断

缺省情况下，JIT编译器将启用以改进应用性能。即便如此，如果你怀疑JIT出现问题，临时禁用JIT，将可以帮助你诊断问题。

由于JVM启动的同时也启动了JIT，因此你只能在启动前调整JIT参数。

有多种方法禁用JIT：
* 在命令行指定-Djava.compiler=NONE
* 在命令行指定[-Xjit](../命令行参数/JVM-X参数/-Xjit)
，该参数可以关闭JIT和AOT编译器。如果想要确定是哪种编译器导致的问题，也可以选择性的使用-Xnojit和-Xnoaot参数
* 在编程期间调用java.lang.Compiler API，不过这个API在Java SE9中已经被移除

如果关闭JIT解决了你的问题，你可以进一步通过一些参数来观察JIT的详细信息。

Turning on verbose logging with the verbose suboption causes the JIT to record all compiler operations. However, the log file can be difficult to read because there are so many complex operations occuring in rapid succession. Follow these steps to simplify operations, which helps you pinpoint the root cause:

### 理解JIT的详细信息

