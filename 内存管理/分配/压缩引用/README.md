# 引用压缩

在启用引用压缩特性时，虚拟机会将所有对于对象、类、线程和监控器的引用作为 32 位值存储。使用 -Xcompressedrefs 和 -Xnocompressedrefs 命令行选项可在 64 位虚拟机中启用或禁用引用压缩。这些选项仅在 64 位 虚拟机中才能生效。

由于对象越小，会导致垃圾回收的频率越低，并且可提升内存高速缓存的利用率，因此启用引用压缩特性能够提高多个应用程序的性能。某些应用程序可能无法从引用压缩特性中获益。你可以在启用引用压缩和禁用引用压缩这两种情况下测试应用程序的性能，以确定引用压缩是否合适。

在启用引用压缩特性时，以下结构分配在最低 4 GB 的地址空间中：
* 类
* 线程
* 监视器

此外，操作系统和本地库也会使用其中部分地址空间。较小的 Java 堆也分配在低 4 GB 的地址空间中。更大的 Java 堆分配在更高的地址空间中。

在启用引用压缩特性时，如果低 4 GB 的地址空间已满，尤其是在类加载器、启动线程或使用监视器时，可能发生本地内存 OutOfMemoryError 异常。通常，您可以使用较大的 -Xmx 选项使 Java 堆在更高的地址空间来解决这些错误。

在 Windows 上，缺省情况下，操作系统在最低 4 GB 的地址空间中分配内存，直至该区域已满为止。较大的 -Xmx 值可能不足以避免发生 OutOfMemoryError 异常。高级用户可以通过将名为 HKLM\System\CurrentControlSet\Control\Session Manager\Memory Management\AllocationPreference 的注册表键设置为 (REG_DWORD)0x100000 来更改缺省 Windows 分配选项。有关此注册表键的更多信息，请参阅[这里](https://msdn.microsoft.com/en-us/library/bb190527.aspx)。

[-Xmcrs](../../../命令行参数/JVM-X参数/-Xmcrs.md) 选项允许你设置一个初始内存区域，该内存区域是在低 4 GB 地址空间用来保存引用压缩。设置此选项可以保证引用压缩所使用的本机类、监视器和线程的空间。在堆中获取空间的另一种方法是使用 [-Xgc:preferredHeapBase](../../../命令行参数/JVM-X参数/-Xgc.md)，它允许你选择内存地址范围，对使用 -Xmx 选项指定的堆进行分配。

64 位虚拟机也可以使用 Oracle JVM 选项：
    * -XX:+UseCompressedOops：这将在 64 位虚拟机中启用引用压缩特性。等同于指定 -Xcompressedrefs 选项。
    * -XX:-UseCompressedOops：这将在 64 位虚拟机中禁用引用压缩特性。
    
注：提供以上两个选项，主要是针对 64 位平台将应用程序从 Oracle JVM 移植到 J9 VM 时的场景。后续发行版可能不支持这些选项。
