# 高速缓存分配

高速缓存分配专门用于为小对象提供可能的最佳分配性能。

直接从线程先前从堆分配的线程本地分配缓冲区分配对象。将从该高速缓存分配新对象，而不需要抓取堆锁；因此，高速缓存分配效率非常高。

将从高速缓存分配大小小于 512 个字节（64 位 JVM 上为 768 个字节）的所有对象。如果较大的对象包含在现有高速缓存中，那么将从该高速缓存进行分配；否则，将执行锁定的堆分配。

有时，高速缓存块称为线程局部堆 (TLH)。TLH 的大小在 512 字节（64 位 JVM 上为 768 字节) 到 128 KB 的范围内，这取决于线程分配速度。为分配大量对象的线程提供了更大的 TLH，以进一步减少对堆锁的争用。

Cache allocation is specifically designed to deliver the best possible allocation performance for small objects.

Objects are allocated directly from a thread local allocation buffer that the thread has previously allocated from the heap. A new object is allocated from this cache without the need to grab the heap lock; therefore, cache allocation is very efficient.

All objects less than 512 bytes (768 bytes on 64-bit JVMs) are allocated from the cache. Larger objects are allocated from the cache if they can be contained in the existing cache; if not a locked heap allocation is performed.

The cache block is sometimes called a thread local heap (TLH). The size of the TLH varies from 512 bytes (768 on 64-bit JVMs) to 128 KB, depending on the allocation rate of the thread. Threads which allocate lots of objects are given larger TLHs to further reduce contention on the heap lock.