# 垃圾回收器诊断数据

作者：垃圾回收器下文简称为GC

## 输出GC详细数据

尝试诊断GC问题时，详细日志(verbose logging)是最重要的工具，还可以通过调用一个或多个-Xtgc（trace garbage collector）选项获取更多信息

缺省情况下，-verbose:gc选项会将日志输出到stderr中，可以使用-Xverbosegclog选项将日志重定向到文件中，然后通过IBM的GCMV工具来分析内容。

GC日志是基于事件的，每个垃圾回收操作进行时会生成日志

一个垃圾回收周期由一个或多个垃圾回收操作组成，包括一个或多个增量回收。可导致垃圾回收周期的事件有很多，包括：
* 调用System.gc()。
* 分配内存空间失败。
* 完成并发回收。
* 基于资源分配开销来作出决策。

每个事件的详细日志中包括一个递增的ID和本地时间戳，ID会按事件（不论事件类型如何）递增，因此，你可使用此标记在日志中搜索。

## GC事件

### 初始化（initialized）

初始化GC时，日志会显示生效的垃圾回收参数。可以使用-Xgcthreads选项修改这些参数

日志显示的第一个标签是<initialized>，属性包括id和timestamp的值。<initialized>部分显示的信息包括垃圾回收策略、策略选项和任何JVM命令行选项。

    <initialized id="1" timestamp="2010-11-23T00:41:32.328">
      <attribute name="gcPolicy" value="-Xgcpolicy:gencon" />
      <attribute name="maxHeapSize" value="0x5fcf0000" />
      <attribute name="initialHeapSize" value="0x400000" />
      <attribute name="compressedRefs" value="false" />
      <attribute name="pageSize" value="0x1000" />
      <attribute name="requestedPageSize" value="0x1000" />
      <attribute name="gcthreads" value="2" />
      <system>
        <attribute name="physicalMemory" value="3214884864" />
        <attribute name="numCPUs" value="2" />
        <attribute name="architecture" value="x86" />
        <attribute name="os" value="Windows XP" />
        <attribute name="osVersion" value="5.1" />
      </system>
      <vmargs>
        <vmarg name="-Xoptionsfile=C:\jvmwi3270\jre\bin\default\options.default" />
        <vmarg name="-Xlockword:mode=default,noLockword=java/lang/String,noLockword=
         java/util/MapEntry,noLockword=java/util/HashMap$Entry,noLockword..." />
        <vmarg name="-XXgc:numaCommonThreadClass=java/lang/UNIXProcess$*" />
        <vmarg name="-Xjcl:jclscar_26" />
        <vmarg name="-Dcom.ibm.oti.vm.bootstrap.library.path=C:\jvmwi3270\jre\bin\
         default;C:\jvmwi3270\jre\bin" />
        <vmarg name="-Dsun.boot.library.path=C:\jvmwi3270\jre\bin\default;C:\
         jvmwi3270\jre\bin" />
        <vmarg name="-Djava.library.path=C:\jvmwi3270\jre\bin\default;C:\
         jvmwi3270\jre\bin;.;c:\pwi3260\jre\bin;c:\pwi3260\bin;C:\WINDOWS\syst..." />
        <vmarg name="-Djava.home=C:\jvmwi3270\jre" />
        <vmarg name="-Djava.ext.dirs=C:\jvmwi3270\jre\lib\ext" />
        <vmarg name="-Duser.dir=C:\jvmwi3270\jre\bin" />
        <vmarg name="_j2se_j9=1119744" value="7FA9CEF8" />
        <vmarg name="-Dconsole.encoding=Cp437" />
        <vmarg name="-Djava.class.path=." />
        <vmarg name="-verbose:gc" />
        <vmarg name="-Dsun.java.command=Foo" />
        <vmarg name="-Dsun.java.launcher=SUN_STANDARD" />
        <vmarg name="_port_library" value="7FA9C5D0" />
        <vmarg name="_bfu_java" value="7FA9D9BC" />
        <vmarg name="_org.apache.harmony.vmi.portlib" value="000AB078" />
      </vmargs>
    </initialized>
    
有关垃圾回收策略部分中一些项
* gcPolicy，参考 -Xgcpolicy 选项
* maxHeapSize，即-Xmx设置，堆最大值，参考[这里](../内存管理/README.md#初始堆大小和最大堆大小)
* initialHeapSize, 即-Xms设置，初始堆大小，参考[这里](../内存管理/README.md#初始堆大小和最大堆大小)
* compressedRefs，压缩引用
* pageSize，页面大小
* requestedPageSize，请求页面大小
* gcthreads，参考 -Xgcthreads 选项


### Stop-the-world（exclusive）

Stop-the-world意味着应用被暂停，GC独占JVM进程，当发生这个事件时，日志会显示exclusive-start和exclusive-end标签

    <exclusive-start id="3663" timestamp="2015-12-07T14:25:14.704" intervalms="188.956">
    <response-info timems="0.131" idlems="0.048" threads="3" lastid="000000000258CE00" lastname="Pooled Thread #2"/>
    </exclusive-start>
    ......
    <exclusive-end id="3674" timestamp="2015-12-07T14:25:14.732" durationms="27.513" />

以下是对日志这一部分中各个项的说明：

* \<exclusive-start> 和 <exclusive-end>
    这两个标签表示“Stop-the-world”操作，这两个标签具有以下属性：
   
   * timestamp：“Stop-the-world”操作的开始或结束时的本地时间戳
   
   * intervalms：<exclusive-start>标签的属性，距离上次此类回收的耗时，单位是毫秒
   
   * durationms：<exclusive-end>标签的属性，GC独占的总时间，单位是毫秒。
   

* <response-info>此标签提供GC独占JVM进程的详细信息。此标签具有以下属性：
    
    * timems：GC获取独占权所需的时间，单位是毫秒。要获取独占权，GC会请求所有其他线程停止处理，然后等待这些线程响应请求。如果此时间过长，可以使用 
        -Xdump:system:events
        选项创建系统转储文件。该转储文件可以帮助你识别独享请求响应缓慢的线程。例如，当某线程对 JVM 请求的响应时间超过1秒，那么以下选项将触发创建系统转储文件：
        
        
        -Xdump:system:events=slow,filter=1000ms
        
 
   * idlems：某个线程响应GC独占请求，到最后一个线程响应GC独占请求，这段时间内，该线程其实是在等待或说是空闲的。这个属性的值是所有线程的平均空闲时间，单位是毫秒。

   * threads：请求释放 JVM 访问权的线程数，所有线程必须响应。
   * lastid：最后一个响应的线程ID。
   * lastname：最后一个响应的线程名称。
   
   
### 垃圾回收周期（cycle）

GC日志显示每个垃圾回收周期，以成对的标签<cycle-start> 和 <cycle-end> 标识，每个垃圾回收周期至少包含一个垃圾回收增量。

<cycle-end> 标记包含 context-id 属性，该属性与<cycle-start> 标记的 id 属性匹配，值相同即一个垃圾回收周期

    <cycle-start id="4" type="scavenge" contextid="0" timestamp="2015-12-07T14:21:11.196" intervalms="225.424" />
    <cycle-end id="10" type="scavenge" contextid="4" timestamp="2015-12-07T14:21:11.421" />
    
在该示例中，<cycle-end> 标记的 context-id 值为 4，与 <cycle-start>  id 值相等。

以下是对日志这一部分中各个项的说明：

* \<cycle-start> 和 <cycle-end>：这两个标签表示垃圾回收周期。每个标签具有以下属性：
    * type 垃圾回收类型。此属性可以具有以下值：
        * scavenge：新生代的回收被称为 Scavenge。
        * global：在整个堆进行标记/清理，可选是否做整理操作。有关全局垃圾回收的更多信息，请参阅：全局垃圾回收的详细描述
    * contextid：<cycle-end> 标签中的 contextid 属性与对应的 <cycle-start> 标签中 id 属性匹配。 
    * timestamp：垃圾回收周期开始或结束时的本地时间戳。
    * intervalms：距离上次此类回收的耗时，单位是毫秒。对于 <cycle-start> 标签中的intervalms值，包括前一次垃圾回收周期持续时间，以及上一次回收周期结束到本次回收周期开始之间的时间间隔。 <cycle-end>没有该属性。
   
   
如果你在使用 balanced 垃圾回收策略，可能会在 <cycle-start> 标签之前看到以下行：

    <allocation-taxation id="28" taxation-threshold="2621440" timestamp="2014-02-17T16:21:44.325" intervalms="319.068">
    </allocation-taxation>
    
这行意思是本次垃圾收集周期是由于上个周期结束时设置的分配阈值触发的。taxation-threshold即阈值的值
    
### 垃圾回收增量（gc）
  
GC日志显示每个垃圾回收增量，以成对的标签<gc-start> 和 <gc-end> 标识。每个垃圾回收增量至少包含一个垃圾回收操作。

    <gc-start id="5" type="scavenge" contextid="4" timestamp="2015-12-07T14:21:11.196">
      <mem-info id="6" free="13649296" total="22151168" percent="61">
        <mem type="nursery" free="10608" total="4587520" percent="0">
          <mem type="allocate" free="10608" total="4128768" percent="0" />
          <mem type="survivor" free="0" total="458752" percent="0" />
        </mem>
        <mem type="tenure" free="13638688" total="17563648" percent="77">
          <mem type="soa" free="13638688" total="17563648" percent="77" />
          <mem type="loa" free="0" total="0" percent="0" />
        </mem>
        <remembered-set count="1449" />
      </mem-info>
    </gc-start>
    ...
    <gc-op id="7" type="scavenge" timems="3.107" contextid="4" timestamp="2015-12-07T14:21:11.199">
    ...
    <gc-end id="8" type="scavenge" contextid="4" durationms="3.300" usertimems="14.998" systemtimems="3.000" timestamp="2015-12-07T14:21:11.200" activeThreads="10">
      <mem-info id="9" free="16156512" total="22151168" percent="72">
        <mem type="nursery" free="3604480" total="4587520" percent="78">
          <mem type="allocate" free="3604480" total="4063232" percent="88" />
          <mem type="survivor" free="0" total="524288" percent="0" />
        </mem>
        <mem type="tenure" free="12552032" total="17563648" percent="71">
          <mem type="soa" free="12552032" total="17563648" percent="71" />
          <mem type="loa" free="0" total="0" percent="0" />
        </mem>
        <pending-finalizers system="0" default="0" reference="2" classloader="0" />
        <remembered-set count="1415" />
      </mem-info>
    </gc-end> 
    
以下是对日志这一部分中各个项的说明：

* <gc-start>：该标签表示垃圾回收增量开始。此标签具有以下属性：
    * type：垃圾回收类型。此属性可以具有以下值：
        * scavenge：新生代的回收被称为 Scavenge。
        * global：在整个堆进行标记/清理，可选是否做整理操作。有关全局垃圾回收的更多信息，请参阅：全局垃圾回收的详细描述
    * contextid：contextid 属性与对应的垃圾回收周期的 id 属性匹配。在该示例中，值 4 表示该垃圾回收增量是 <cycle-start id="4"> 的垃圾回收周期的一部分。
    * timestamp：垃圾回收增量开始时的本地时间戳。
    * <mem-info>：在<gc-start>和<gc-end>之间，提供有关 Java堆的当前状态信息。
    
* <gc-end>：该标签表示垃圾回收增量的结束。此标签具有以下属性：
    * type：垃圾回收类型。此属性可以具有以下值：
        * scavenge：新生代的回收被称为 Scavenge。
        * global：在整个堆进行标记/清理，可选是否做整理操作。有关全局垃圾回收的更多信息，请参阅：全局垃圾回收的详细描述
    * contextid：contextid 属性与对应的垃圾回收周期的 id 属性匹配。在该示例中，值 4 指示该垃圾回收增量是包含标记 <cycle-start id="4"> 的垃圾回收周期的一部分。
    * timestamp：垃圾回收增量结束时的本地时间戳。
    * usertimems：垃圾回收线程在用户态下花费的总 CPU 时间，单位是毫秒。
    * systemtimems：垃圾回收线程在内核态下花费的总 CPU 时间，单位是毫秒。较高的 systemtimems 值意味着在垃圾回收线程之间共享工作具有更高的开销。如果遇到这种情况，你可以使用 -Xgcthreads 选项来减少垃圾回收线程计数。
    * activeThreads：此垃圾回收增量期间活动的垃圾回收线程数。该数目可能小于GC日志初始化（initialized）阶段显示的垃圾回收线程数。
    * <mem-info>：在<gc-start>和<gc-end>之间，提供有关 Java堆的当前状态信息。
    
* <mem-info>：该标签显示 Java 堆中的空闲空间和总空间的总数值，该数值通过累加新生代堆大小和老年代堆大小来计算。在以前的 Java 虚拟机版本中，只要使用分代并发垃圾收集器，就需要计算覆盖率（？），因为总数值不包括新生代的残存空间。从 VM V2.8 开始，不再需要这些计算。

    * <mem>：在每个 <mem-info> 标签内，会有多个 <mem> 标签，用来显示各个内存区域中可用内存的分布。 每个 <mem> 标签显示垃圾回收事件前后该内存区域中空闲容量和总容量。空闲容量显示为一个数字和一个四舍五入的百分比。内存区域通过 type 属性标识，该属性具有以下某个值：
        * nursery：如果使用分代并发垃圾收集器，nursery 类型指的是Java堆的新生区域，[分代垃圾收集器](../垃圾回收/分代垃圾收集器/README.md)
            * allocate：新生代空间中用来分配给新对象的区域。
            * survivor：垃圾回收周期内，从allocate区域中选择还没有达到老年代年龄的对象移动到survivor区域。当 Java 应用程序运行时，会始终显示 0% 的空闲空间。出现这种数据的原因是，虽然空间为空，但已被保留。
        * tenure：tenure 类型说明此 <mem> 标签表示的是Java堆的老年带区域，该区域用于存储达到一定年龄的对象。该区域可以进一步划分为 soa 和 loa 区域。
            * soa：表示老年代堆中小对象区域。该区域用于对象的首次分配尝试（？）。
            * loa：表示老年代堆中大对象区域。该区域用于大对象的分配。这部分更多信息，请参阅[大对象区域](../内存管理/分配/大对象区域/README.md)
<gc-end> 标签还包含 <pending-finalizers> 标签。这部分更多信息，请参阅[最终化](GC_log.md#最终化)。

### 垃圾回收操作（gc-op）

每个垃圾回收增量至少包含一个垃圾回收操作，通过 <gc-op> 标签来标识一个垃圾回收操作，记录在GC日志中

<gc-op> 包含若干子章节，这些子章节描述了特定于垃圾回收操作类型的操作。随着技术的改进或新数据的引入，这些子章节可能在各版本中有所变化。

以下是一个GC日志中垃圾回收操作的示例：

    <gc-op id="7" type="scavenge" timems="1.127" contextid="4" timestamp="2010-11-23T00:41:32.515">
       ...
       ... subsections that are determined by the operation type
       ... 
    </gc-op>
    
以下是对日志这一部分中各个项的说明：

* <gc-op>：该标签表示垃圾回收操作，具有以下属性：
    * type：垃圾回收类型。该属性的值依赖于垃圾回收周期的阶段和正在使用的垃圾回收策略。这个值将决定 <gc-op> 标签中显示的自标签。
    以下 type 值可能会在全局垃圾回收周期的任何阶段出现，可能的垃圾回收策略包括：gencon、optthruput、optavgpause
        * mark：垃圾回收的标记阶段。在该阶段中，垃圾回收器会标记所有存活对象。具体请参阅[标记](#标记)
        * sweep：垃圾回收的清理阶段。在该阶段中，垃圾回收器将将堆的空闲部分做标记。具体请参阅[清理](#清理)
        * compact：垃圾回收的整理阶段。在该阶段中，垃圾回收器通过移动对象以创建更大的、集中的空闲内存区域。垃圾回收器还会将对移动对象的引用指向新的位置。未必会发生此类型的操作，因为并不总是需要整理。具体请参阅[整理](#整理)
        * classunload：垃圾回收器会卸载不包含存活对象实例的类和类加载器。未必会发生此类型的操作，因为并不总是需要卸载。具体请参阅[类卸载](#类卸载)
Final stop-the-world part of a concurrent global garbage collection
    
    以下 type 值只会在最终的“stop-the-world”阶段出现，这是并行全局垃圾回收周期中的一部分，可能的垃圾回收策略包括：gencon、optavgpause。 这些操作按以下所示的顺序，会在必需的标记/清理操作之前发生。
        * trace：垃圾回收器会在最终的card-cleaning ？ 阶段之前跟踪和标记存活对象。通常情况下，跟踪和标记会同时完成，出现此类型仅当全局垃圾回收周期的并行阶段出现了异常中止。
        * rs-scan：垃圾回收器会根据remembered set 在新生区跟踪和标记对象。remembered set 是老年代中的对象列表，这些对象引用了新生代中的对象。
        * card-cleaning：发生在最终“stop-the-world”标记之前的card-cleaning阶段。 该阶段是增量更新并行标记中的正常步骤。 该阶段用于对并行跟踪和标记期间的存活对象突变进行补偿。
    
    以下 type 值仅应用于 gencon 垃圾回收策略，并出现在本地和新生代回收期间。
        * scavenge：该操作包括跟踪新生代存活的对象，并将这些对象移到存活区。该操作还包括修正或调整整个堆的对象引用。具体请参阅[scavenge](#scavenge)
        
    * timems：完成垃圾回收操作所需的时间，单位是毫秒。
    * contextid：contextid 属性与对应的垃圾回收周期的 id 属性匹配。在该示例中，值 4 指示该垃圾回收增量是包含标记 <cycle-start id="4"> 的垃圾回收周期的一部分。
    * timestamp：执行垃圾回收操作时的本地时间戳。

#### 标记

以下GC日志显示了 mark 操作的示例：

    <gc-op id="9016" type="mark" timems="14.563" contextid="9013" timestamp="2015-09-28T14:47:49.927">
      <trace-info objectcount="1896901" scancount="1307167" scanbytes="34973672" />
      <finalization candidates="1074" enqueued="1" />
      <references type="soft" candidates="16960" cleared="10" enqueued="6" dynamicThreshold="12" maxThreshold="32" />
      <references type="weak" candidates="6514" cleared="1" enqueued="1" />
      <references type="phantom" candidates="92" cleared="0" enqueued="0" />
      <stringconstants candidates="18027" cleared="1" />
    </gc-op>
    
以下是对 mark操作类标签中的各个子标签的说明：
    * <trace-info>：包含与被跟踪对象有关的通用信息。此标签具有以下属性：
        * objectcount：stop-the-world 阶段中发现的对象数量。
        * scancount：非叶对象的数量，非叶对象指的是至少持有一个引用的对象。
        * scanbytes：所有可扫描对象的总大小，单位是字节。（所有存活对象称为存活集合，此值小于存活集合的总大小。）
    * <finalization>：参考[最终化](#最终化)
    * <references>：参考[引用处理](#引用处理)
    * <stringconstants>：包含与被跟踪对象有关的通用信息。此标签具有以下属性：
        * candidates：字符串常量总数。
        * cleared：在垃圾回收周期中移除的字符串常量数。（不会显示自上一次全局垃圾回收以来添加的字符串常量数）

#### 清理
以下GC日志显示了 sweep 操作的示例：

    <gc-op id="8979" type="sweep" timems="1.468" contextid="8974" timestamp="2015-09-28T14:47:49.141" />
    
sweep操作类标签没有子标签

#### 整理
以下GC日志显示了 compact 操作的示例：

    <gc-op id="8981" type="compact" timems="43.088" contextid="8974" timestamp="2015-09-28T14:47:49.184">
      <compact-info movecount="248853" movebytes="10614296" reason="compact on aggressive collection" />
    </gc-op>
    
compact操作类标签只有1个子标签：
    * <compact-info>：此标签具有以下属性：
        * movecount：移动的对象数量。
        * movebytes：移动的对象大小，单位是字节。
        * reason：触发整理操作的原因：
            * compact to meet allocation：在标记/清理之后仍无法满足分配需求。
            * compact on aggressive collection：主动垃圾回收是一种全局垃圾回收，为了尽可能释放更多内存，会包含额外步骤和GC操作。这其中一项操作为compact。主动垃圾回收一般发生在一次正常（非主动）全局垃圾回收之后，因为这次正常全局垃圾回收仍无法满足分配需求。请注意，显式回收（例如，通过调用 System.gc() ）是一种主动垃圾回收。
            * heap fragmented：根据内部度量值，堆存在大量碎片，因此需要整理。有多种原因需要减少碎片，例如，为防止分配大对象失败、增强对象和引用的局部性、降低分配争用或降低全局垃圾回收的频率。
            * forced gc with compaction：显式发起了一次包含 compact 操作的全局垃圾回收，例如，使用代理或工具，堆转储之前执行。
            * low free space：可用内存小于 4%。
            * very low free space：可用内存小于 128 KB。
            * forced compaction：使用一个 JVM 选项（例如 -Xcompactgc）显式请求了整理。
            * compact to aid heap contraction：在堆中按照从高到低地址范围的顺序移动对象以创建连续的空闲内存空间，从而帮助收缩堆。

#### 类卸载

以下GC日志显示了 classunload 操作的示例：

    <gc-op id="8978" type="classunload" timems="1.452" contextid="8974" timestamp="2015-09-28T14:47:49.140">
      <classunload-info classloadercandidates="1147" classloadersunloaded="3" classesunloaded="5"
                    anonymousclassesunloaded="0" quiescems="0.000" setupms="1.408" scanms="0.041" postms="0.003" />
    </gc-op>
    
classunload操作类标签只有1个子标签：
    * <classunload-info>：此标签具有以下属性：
        * classloadercandidates：类加载器的总数。
        * classloadersunloaded：该垃圾回收周期中卸载的类加载器数量。
        * classesunloaded：卸载的类的数量。
        * anonymousclassesunloaded：卸载的匿名类的数量。（匿名类单独卸载并单独报告。）
        * quiescems、setupms、scanms、postms：总时间细分到四个子步骤中，单位是毫秒。
        
#### 清扫
清扫操作仅在 gencon 垃圾回收策略中发生。当新生代中的分配区满时将运行清扫操作。在清扫期间，可达的对象将被复制到新生代的存活区，或者达到年龄后直接复制到老年代。更多信息请参阅[分代并发垃圾收集器](../垃圾回收/分代垃圾收集器/README.md)。

以下GC日志显示了 清扫 操作的示例：

    <gc-op id="9029" type="scavenge" timems="2.723" contextid="9026" timestamp="2015-09-28T14:47:49.998">
      <scavenger-info tenureage="3" tenuremask="ffb8" tiltratio="89" />
      <memory-copied type="nursery" objects="11738" bytes="728224" bytesdiscarded="291776" />
      <memory-copied type="tenure" objects="6043" bytes="417920" bytesdiscarded="969872" />
      <finalization candidates="266" enqueued="0" />
      <references type="soft" candidates="94" cleared="0" enqueued="0" dynamicThreshold="12" maxThreshold="32" />
      <references type="weak" candidates="317" cleared="25" enqueued="17" />
    </gc-op>
    
以下是对 scavenge 操作类标签中的各个子标签的说明：
    * <scavenger-info>：包含与操作有关的通用信息。此标签具有以下属性：
        * tenureage：对象被提升到老年代的年龄。更多信息请参阅[分代并发垃圾收集器](../垃圾回收/分代垃圾收集器/README.md)。
        * tenuremask
        * tiltratio：上次清扫事件和空间调整之后的倾斜率（百分比）。清扫器使用名为“倾斜”的进程在分配区和存活区之间重新分配内存。 “倾斜”进程控制分配区和存活区的相对大小，并调整倾斜率以最大程度地延长两次scavenge操作之间的时间。倾斜率为 60% 表示新生代的 60% 空间用于分配区，40% 空间用于存活区。更多信息请参阅[分代并发垃圾收集器](../垃圾回收/分代垃圾收集器/README.md#倾斜率)。
    * <memory-copied>：拷贝到新生代存活区或晋升到老年代的对象数量。此标签具有以下属性：
        * type：值为 nursery 或 tenure。
        * objects：拷贝到新生代存活区或晋升到老年代的对象数量。
        * bytes：拷贝到新生代存活区或晋升到老年代的对象大小，单位是字节。
        * bytesdiscarded：没有成功晋升到老年代的对象The number of bytes consumed in the nursery or tenure area but not successfully used for flipping or promotion。对于每个区域，已消耗内存总量是 bytes 和 bytesdiscarded 值的总和。
    * <finalization>：参考[最终化](#最终化)
    * <references>：参考[引用处理](#引用处理)

#### 最终化


#### 引用处理


