# 遗留问题

## Javadump的MEMINFO部分，Segment是什么意思
## Javadump的MEMINFO部分，为什么Class Memory占用率总是那么高

## Javadump的MEMINFO部分，为什么JIT Code Cache闲置率那么高

## Javadump的LOCKS部分，flat & inflated object-monitors什么意思

## Javadump的LOCKS部分，kafka-coordinator-heartbeat-thread 是干什么的

    2LKMONINUSE      sys_mon_t:0x00007F38D4007508 infl_mon_t: 0x00007F38D4007588:
    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091AA00: <unowned>
    3LKNOTIFYQ            Waiting to be notified:
    3LKWAITNOTIFY            "kafka-coordinator-heartbeat-thread | rm_modelChange_consumer_group1546065147167" (J9VMThread:0x00000000016F0E00)
    2LKMONINUSE      sys_mon_t:0x00007F3878005058 infl_mon_t: 0x00007F38780050D8:
    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005FB6233D8: Flat locked by "kafka-coordinator-heartbeat-thread | 8a688f83-407d-4ac4-8995-e48efc62b247" (J9VMThread:0x000000000171D200), entry count 1
    2LKMONINUSE      sys_mon_t:0x00007F3878005B58 infl_mon_t: 0x00007F3878005BD8:
    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091AC30: <unowned>
    3LKNOTIFYQ            Waiting to be notified:
    3LKWAITNOTIFY            "kafka-coordinator-heartbeat-thread | rm_tenantChange_consumer_group1546065147170" (J9VMThread:0x0000000001556000)
    2LKMONINUSE      sys_mon_t:0x00007F3878006868 infl_mon_t: 0x00007F38780068E8:
    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerNetworkClient@0x00000005FB623350: Flat locked by "configclient-consumer-thread" (J9VMThread:0x0000000001701A00), entry count 1
    3LKWAITERQ            Waiting to enter:
    3LKWAITER                "kafka-coordinator-heartbeat-thread | 8a688f83-407d-4ac4-8995-e48efc62b247" (J9VMThread:0x000000000171D200)
    2LKMONINUSE      sys_mon_t:0x00007F3878006F48 infl_mon_t: 0x00007F3878006FC8:
    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005FB63AC50: Flat locked by "kafka-coordinator-heartbeat-thread | sm_logout_7ca43635-724e-49ee-a4c3-7eb0a83c8479" (J9VMThread:0x000000000170D000), entry count 1
    2LKMONINUSE      sys_mon_t:0x00007F3878007158 infl_mon_t: 0x00007F38780071D8:
    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091AE60: <unowned>
    3LKNOTIFYQ            Waiting to be notified:
    3LKWAITNOTIFY            "kafka-coordinator-heartbeat-thread | rm_br_consumer_group1546065147180" (J9VMThread:0x000000000190F500)
    2LKMONINUSE      sys_mon_t:0x00007F3878007208 infl_mon_t: 0x00007F3878007288:
    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerNetworkClient@0x00000005FB63ABC8: Flat locked by "AuthLogoutListener" (J9VMThread:0x0000000001705500), entry count 1
    3LKWAITERQ            Waiting to enter:
    3LKWAITER                "kafka-coordinator-heartbeat-thread | sm_logout_7ca43635-724e-49ee-a4c3-7eb0a83c8479" (J9VMThread:0x000000000170D000)
    
    
    
## Javadump的LOCKS部分，java.util.concurrent.locks.AbstractOwnableSynchronizer

## Javadump的LOCKS部分，sys_mon_t和infl_mon_t分别是什么
     2LKMONINUSE      sys_mon_t:0x00007FF4B0001D78 infl_mon_t: 0x00007FF4B0001DF8:
        3LKMONOBJECT       java/lang/ref/ReferenceQueue@0x00000000FFE26A10: <unowned>
        3LKNOTIFYQ            Waiting to be notified:
        3LKWAITNOTIFY            "Common-Cleaner" (J9VMThread:0x0000000000FD0100)
        
        
## Javadump的THREADS部分，Heap bytes allocated since last GC cycle 什么意思

自上次垃圾回收以来该线程所分配的堆字节数，有什么用呢？

## Javadump的THREADS部分， CPU usage total: 524.216900183 secs, current category="Application" 

该线程总的CPU占用时间

## Javadump的CPU信息部分，Online CPUs和Target CPUs什么意思

## NativeMEMINFO部分，Classes: 4,682,840 bytes / 141 allocations，allocations是什么含义


## 3LKWAITNOTIFY 什么含义？    

 "dw-104 - GET /api/res/v1/subnetworks/06670f87-1e4f-49df-9910-858028199616/tddv3.enbfunctions?includeAttr=nbiIdDn&queryDn=true" (J9VMThread:0x00000000018F6F00)