# 全局垃圾回收

如果堆锁分配失败，或者对 System.gc() 显示调用，就会执行垃圾回收，发生分配失败的线程 或 调用 System.gc() 的线程就会掌控应用并执行垃圾回收。

垃圾回收的第一步是获取虚拟机的唯一控制权，以阻止任何进一步的 Java 操作。然后，垃圾回收将经历三个阶段：标记、清扫和整理（如果需要）。垃圾收集器 (GC) 所执行的是一种 stop-the-world 的操作，因为在回收垃圾时将停止所有应用程序线程。

在使用均衡的垃圾回收策略时，只有在以下特殊情况，才会发生全局垃圾回收：
* 调用 System.gc()
* 工具请求垃圾回收
* 将堆的大小、占用的堆内存和供不应求的回收比率相结合。

## 标记阶段
在标记阶段，会标记所有活动对象。由于不能标记所有的不可达对象，因此必须标记所有可达的对象。这样，所有其他对象都为垃圾。 标记所有可达对象的进程也称为跟踪。

标记阶段使用：
* 工作包池。每个工作包包含一个标记栈，一个标记栈中包含对尚未跟踪的活动对象的引用。每一个标记线程都会引用2个工作包
* 一个输入工作包，用来弹出引用
* 一个输出工作包，刚刚发现的未标记对象会推入该工作包
* 在引用被推入输出包时对其进行标记。当输入包变为空时，会将其添加到一个空包列表中，然后输入包替换成一个非空包。当输出包已满时，会将其添加到一个非空包列表中，然后输出包替换成一个空包。
* 一个标记对象的位数组，这些对象都是访问过且可达的对象。该位数组也被称为 mark map，JVM 在启动时根据最大堆大小 (-Xmx) 分配该位数组。标记位数组的每1位，表示 8 个字节的堆空间。可达对象的起始地址在堆中对应的位，会在首次访问可达对象时被标记。
跟踪的第一个阶段是标识根对象。 JVM 中活动状态包括：
* 为每个线程保存的寄存器
* 代表线程的堆栈集
* Java 类中的静态字段
* 局部和全局 JNI 引用集
* 在 JVM 本身内调用的所有函数将使得帧位于 C 堆栈上。该帧可能包含对以下对象的引用：作为局部变量分配结果的对象，或作为从调用程序发送的参数结果的对象。跟踪例程将同等对待所有这些引用。
将设置所有根对象的所有标记位，并将对根的引用推入输出工作包。然后，通过重复以下过程继续跟踪：使引用离开标记线程的输入工作包，然后扫描所引用的对象以获取对其他对象的引用。如果标记位结束，对未标记的对象加以引用。 通过在标记位数组设置合适的位来标记对象。 然后将该引用推送至标记线程的输出工作包。 该过程将一直继续，直至所有工作包都位于空表上，此时，所有可访问对象都已标识。

All functions that are called in the JVM itself cause a frame on the C stack. This frame might contain references to objects as a result of either an assignment to a local variable, or a parameter that is sent from the caller. All these references are treated equally by the tracing routines.
All the mark bits for all root objects are set and references to the roots pushed to the output work packet. Tracing then proceeds by iteratively popping a reference off the marking thread's input work packet and then scanning the referenced object for references to other objects. If the mark bit is off, there are references to unmarked objects. The object is marked by setting the appropriate bit in the mark bit array. The reference is then pushed to the output work packet of the marking thread. This process continues until all the work packets are on the empty list, at which point all the reachable objects have been identified.

### 标记栈溢出

由于工作包集大小有限，因此可能会溢出，如果溢出垃圾回收器将执行一系列操作。

如果发生溢出，那么 GC 会通过以下方式腾空一个工作包：一次弹出一个引用，并通过使用对象头中的类指针属性所引用的对象，使它们脱离所属的类。具有溢出对象的所有类也将链接在一起。然后，跟踪就能够像以前一样继续。如果发生进一步的标记栈溢出，那么以相同的方式腾空更多的包。

在标记线程请求新的非空包并且所有工作包都为空时，GC 将检查溢出类列表。如果该列表不为空，那么 GC 将遍历此列表，并使用溢出列表上对象的引用来重新填充工作包。 然后按照前面所述处理这些包。当所有工作包都为空并且溢出列表也为空时完成跟踪。

Because the set of work packets has a finite size, it can overflow and the Garbage Collector (GC) then performs a series of actions.

If an overflow occurs, the GC empties one of the work packets by popping its references one at a time, and chaining the referenced objects off their owning class by using the class pointer field in the object header. All classes with overflow objects are also chained together. Tracing can then continue as before. If a further mark stack overflow occurs, more packets are emptied in the same way.

When a marking thread asks for a new non-empty packet and all work packets are empty, the GC checks the list of overflow classes. If the list is not empty, the GC traverses this list and repopulates a work packet with the references to the objects on the overflow lists. These packets are then processed as described previously. Tracing is complete when all the work packets are empty and the overflow list is empty.

## 清理阶段

标记阶段完成后，标记位向量标识了所有活动对象在堆中的位置。清理阶段依据此来标识可回收以供未来分配的堆存储块，这些块被添加到可用空间池。

可以通过标记位向量中连续的0序列，来查找空闲块。GC 将忽略小于最小可用大小的长度对应的任何的连续的0序列。在找到长度足够的序列后，GC 将在序列开始处检查对象的长度，以确定可回收的实际可用空间大小。如果该空间量大于或等于空闲块的最小大小，那么将进行回收并将它添加到可用空间池。默认的，空闲块的最小大小定义为 512 个字节（32 位平台上）和 768 个字节（64 位平台上）。

不在空闲列表上的小存储区块称为“暗物质”，它们在相邻对象变为空闲时或者在整理堆时“复活”。没必要释放空闲块中的各个对象，这是因为整个块都是可用存储空间。当某个块被回收时，GC 并不知道块中所包含的对象。

*作者注：在垃圾回收概念中，并行是指 GC 多线程处理，英文是 Parallel ，并发是指与应用程序并发进行，英文是 Concurrent *

### 并行逐位清理

并行逐位清理通过多处理器并行处理缩短了清理时间。在并行逐位清理过程中，垃圾回收器会使用在并行标记中所使用的相同的助手线程，因此默认的助手线程数量也与并行标记中线程数量相同，并可使用[-Xgcthreads](../../命令行参数/JVM-X参数/-Xgcthreads.md) 选项修改。

堆被分为多个 256 KB 大小的段，每个线程（助手线程或主线程）每次获取一个段并对其进行扫描，执行一个修改后的逐位清理。每个段的扫描的结果将存储起来，在所有段完成扫描后，空闲列表就构建起来了。

如果使用的是均衡垃圾回收策略 [-Xgcpolicy:balanced](../../命令行参数/JVM-X参数/-Xgcpolicy.md)，Java 堆将被分成大约 1000 个段，为并行逐位清扫提供一个 granular base 。

### 并发清理
与并发标记一样，并发清理能够在堆大小增加时缩短垃圾回收暂停时间。并发清理在执行“stop-the-world”回收后不久启动，并且必须在允许并发标记开始前，至少完成其工作的一部分，这是因为用于并发标记的mark map（mark bit array）也将用于清理。

并发清理过程的操作可分为两种类型：
* 清理分析：针对可用或潜在可用的内存范围，分析mark map（mark bit array）中的数据部分。
* 连接： 堆的已分析部分将和空闲列表相连接。
堆段的计算方式与并行逐位清理的计算方式相同。

STW（stop-the-world） 回收最初执行最少的清理操作，即搜索并查找足够大的可用空间来应对当前的分配失败。剩余未处理的部分堆和
标记映射（mark map）将留给并发清理阶段进行分析和连接。这项工作是在分配过程中通过 Java 线程来完成的。为实现成功的分配，将分析相对于分配大小的堆，并在分配锁外执行。在分配过程中，如果当前空闲列表无法满足请求，将查找已分析的堆段并将它们连接到空闲列表。如果段存在但未分析，那么分配线程在连接之前还必须对其进行分析。

由于在 STW（stop-the-world） 回收结束时清理还未完成，所以报告的可用内存量（通过详细的垃圾回收或 API）是基于过去的堆占用率和未处理的堆大小占堆总大小的比例而得出的一个估算值。另外，整理阶段要求应先完成清理，才会进行整理。 因此，一次整理类型的 STW（stop-the-world） 回收在下一个执行周期内不会执行并发清理操作。

要启用并发清理，请使用 [-Xgcpolicy](../../命令行参数/JVM-X参数/-Xgcpolicy.md): 参数 optavgpause。 它将与并发标记一起工作。其他垃圾回收策略：optthruput、balanced 和 gencon 不支持并发清理。

## 整理阶段

从堆中清理垃圾后，垃圾收集器可能考虑将堆上的对象移动在一起，以得到更大的空闲空间。压缩过程非常复杂，这是因为如果移动任何一个对象，那么 GC 都必须更改
存在的所有指向该对象的引用。所以默认行为是不整理。

以下类比或许可以帮助您了解整理过程。将堆视作一个仓库，其中部分装满了几件大小不同的家具。可用空间是家具之间的间隔。空闲列表仅包含大于特定大小的间隔。
整理操作是向一个方向推动所有家具，并使所有间隔为零。从最靠近墙的对象开始，推动该对象使其紧靠着墙。然后，将第二个对象排成一行，然后推动该对象使其紧靠着第一个对象。然后是第三个对象，推动该对象使其紧靠着第二个对象，以此类推。最后，所有家具都位于仓库的一端，而所有可用空间都位于另一端。

为了使缩短整理时间，会再次使用助手线程。

如果以下任何一个条件为 true，并且未指定 [-Xnocompactgc](../../命令行参数/JVM-X参数/-Xnocompactgc.md)，那么将发生整理：
* 显示设置 [-Xcompactgc](../../命令行参数/JVM-X参数/-Xcompactgc.md)。
* 在一次清理阶段之后，仍没有足够的可用空间用于满足分配需求。
* 已显示调用 System.gc()，并且上一次分配失败触发了全局垃圾回收且未进行整理操作，或者设置了选项 [-Xcompactexplicitgc](../../命令行参数/JVM-X参数/-Xcompactexplicitgc.md)。
* TLH 分配至少消耗了一半之前可用的内存（确保一个精确的样本），并且 TLH 的平均大小下降至低于 1024 个字节。
* 已启用scavenger，在最近一次清除中清除器无法占有的最大对象大于老年代中的最大可用项。
* 堆已完全扩展并且可用旧空间不足 4%。
* 可用堆大小小于 128 KB。
* 使用均衡的垃圾回收策略时，只有需要全局垃圾回收时，[-Xcompactgc](../../命令行参数/JVM-X参数/-Xcompactgc.md) 和 
 [-Xnocompactgc](../../命令行参数/JVM-X参数/-Xnocompactgc.md) 选项才会生效。 正如之前全局垃圾回收的介绍，全局垃圾回收很少发生。 所有针对
 均衡策略的其他回收活动都可能会进行堆整理或对象移动。

## 引用对象
*作者注：引用对象（reference object）是指SoftReference、WeakReference 和 PhantomReference实例； referent可以翻译为引用目标
指引用对象指向的对象。比较绕*

在创建引用对象（reference object）时，会将其添加到相同类型的引用对象（reference object）列表中。引用目标（referent）是指引用对象（reference object）指向的对象。

SoftReference、WeakReference 和 PhantomReference 实例由用户创建并且不能进行更改；除创建时引用的对象之外，不能引用其他对象。

如果一个对象包含一个有 finalize 方法的类，那么指向该对象的指针会添加到需要析构的对象列表中。

在垃圾回收期间，紧接在标记阶段之后，将按照特定顺序处理这些引用列表：
1. 软引用
2. 弱引用
3. 强引用
4. 虚引用

## 软引用、弱引用、虚引用的处理

垃圾回收器 (GC) 确定引用对象是否为回收的候选对象，如果是，那么会分别针对每种引用类型执行不同的回收过程。如果在多个垃圾回收周期内，
引用目标（referent）未被标记，并且未对引用对象调用 #get() 方法，那么将回收软引用。而弱引用和虚引用不同，只要引用目标（referent）未被标记，
就会立即回收。

对于列表上的每个元素，GC 确定引用对象（reference object）是否符合处理条件，然后确定是否符合回收条件。

如果元素已进行标记并且具有非空引用目标（referent）属性，那么符合处理条件。否则，将从引用列表中除去该引用对象，这会导致它在清理阶段被释放。

如果确定元素符合处理条件，那么 GC 必须确定是否符合回收条件。此处的第一个条件非常简单，是否标记了引用目标（referent）？如果已标记，那么引用对象不符合回收条件，并且 GC 将转向处理列表的下一个元素。


如果未标记引用目标（referent），那么 GC 具有一个回收的候选项。 此时，对于每种引用类型，该过程各不相同。如果在许多个垃圾回收周期内未标记引用目标（referent），那么将回收软引用。垃圾回收周期的数量取决于可用堆空间的百分比。 可以使用 [-Xsoftrefthreshold](../../命令行参数/JVM-X参数/-Xsoftrefthreshold.md) 选项来调整回收频率。 如果可用存储空间不足，那么将清除所有软引用。在抛出 OutOfMemoryError 之前，必须已清除所有软引用。

弱引用和虚引用只要引用目标（referent）未被标记就会回收。在处理虚引用时，将标记其引用目标（referent），这样，它将一直存活直至下一个垃圾回收周期或者直至处理完虚引用（如果它与引用队列相关联）。在确定了引用符合回收条件时，它可能会排队进入关联的引用队列，或者从引用列表中除去。

## 需要析构对象的处理

*作者注：强引用的处理中只有需要析构对象的处理需要额外考虑*

对于需要析构的对象处理更加直接。

1. 处理对象列表。未标记的任何元素通过以下方式处理：
    a. 标记和跟踪对象
    b. 在对象的可析构对象列表上创建项
2. GC 从未析构的对象列表移除元素。
3. 对象的 final 方法将由引用处理程序线程在未来不确定的时间点运行。

## JNI弱引用
JNI 弱引用提供与 WeakReference 对象相同的功能，但处理方式大不相同。JNI 例程可以创建对某个对象的 JNI 弱引用，并在稍后删除该引用。
垃圾回收器将清除引用目标（referent）未被标记的任何弱引用，但没有排队机制。

删除 JNI 弱引用失败将导致内存泄漏，并会导致发生性能问题。对于 JNI 全局引用也是如此。在引用处理过程中，对 JNI 弱引用的处理是在最后进行的。
其结果是，对于一个已经最终化并且在此之前已将一个虚引用放入队列并进行处理的对象，可以存在一个 JNI 弱引用。
