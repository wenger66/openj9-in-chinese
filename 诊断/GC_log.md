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
* maxHeapSize，最大堆大小
* initialHeapSize, 初始堆大小
* compressedRefs，压缩引用
* pageSize，页面大小
* requestedPageSize，请求页面大小
* gcthreads，参考 -Xgcthreads 选项



