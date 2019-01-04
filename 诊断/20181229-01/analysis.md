##分析

### 关键字

* 1TISIGINFO dump发生的原因时
* UserArgs Java启动参数
* -Xmx
* 2CIUSERLIMIT
* 2CIENVVAR
* 2CIBOUNDCPU
* Direct Byte Buffers
* 1STHEAPTYPE    Object Memory
* 1STGCHTYPE     GC History 
* requestedbytes
* Global GC
* 3LKWAITER
* Deadlock detected 
* 3XMTHREADBLOCK

观察转储原因：OOM，自动输出了转储文件

    Dump Event "systhrow" (00040000) Detail "java/lang/OutOfMemoryError" "Java heap space" received
    
观察openj9的版本：0.9.0

    1CIJ9VMTAG     openj9-0.9.0
       
观察-Xmx和-Xms参数：应用内存初始值是2.25G，最大值是9G

    2CIUSERARG               -Xms2304m
    2CIUSERARG               -Xmx9216m 
    
观察-Xdump参数：-Xdump启动参数表明了是什么事件触发了dump，本例的event是systhrow，dump的类型是java

    2CIUSERARG               -Xdump:heap:events=systhrow+user,filter=java/lang/OutOfMemoryError,request=exclusive+prepwalk+compact,label=/home/zenap/dump/dump-2018-12-29-14-32-13.phd
    2CIUSERARG               -Xdump:system:events=gpf+abort+traceassert,range=1..0,priority=999,request=serial,label=/home/zenap/dump/core-dump-2018-12-29-14-32-13.%seq.dmp
    2CIUSERARG               -Xdump:heap:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=500,request=exclusive+compact+prepwalk,label=/home/zenap/dump/dump-dump-2018-12-29-14-32-13.%seq.phd
    2CIUSERARG               -Xdump:heap:events=user,priority=500,request=exclusive+compact+prepwalk,label=/home/zenap/dump/dump-dump-user-2018-12-29-14-32-13.%seq.phd
    2CIUSERARG               -Xdump:java:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=400,request=exclusive+preempt,label=/home/zenap/dump/javacore-dump-2018-12-29-14-32-13.%seq.txt
    2CIUSERARG               -Xdump:java:events=gpf+abort+traceassert+user,priority=400,request=exclusive+preempt,label=/home/zenap/dump/javacore-dump-2018-12-29-14-32-13.%seq.txt
    2CIUSERARG               -Xdump:snap:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=300,request=serial,label=/home/zenap/dump/snap-dump-2018-12-29-14-32-13.%seq.trc
    2CIUSERARG               -Xdump:snap:events=gpf+abort+traceassert,priority=300,request=serial,label=/home/zenap/dump/snap-dump-2018-12-29-14-32-13.%seq.trc

观察CPU的资源限制：本例中是无限制

    2CIUSERLIMIT   RLIMIT_CPU                       unlimited            unlimited
    
观察注入的环境变量：尤其是OPENPALETTE_开头的变量，观察是否正确
    
    2CIENVVAR      redis_sentinel_host=ha-redis-ad92e9f1-28c6-488b-9ec1-765705b7484e-sentinel-opcs
    2CIENVVAR      KUBERNETES_SERVICE_PORT=443
    2CIENVVAR      KUBERNETES_PORT=tcp://10.254.0.1:443
    2CIENVVAR      dwApp_elasticsearchConfig_autoDiscover=false
    2CIENVVAR      dwApp_elasticsearchConfig_port=9200
    2CIENVVAR      dwApp_elasticsearchConfig_clusterName=elasticsearch5
    2CIENVVAR      dwApp_elasticsearchConfig_password=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      OPENPALETTE_KAFKA_ZOOKEEPER_ADDRESS=cskafka-22259439-d7ec-418e-bf17-31a7e68857de-zk-opcs
    2CIENVVAR      OPENPALETTE_PGCACHE_DBNAME=db_fa3ccb8f66544d75b191be4e26338b27
    2CIENVVAR      CPU_CORE_NUM=3
    2CIENVVAR      dwApp_kafkaClientConf_bootstrapServers=cskafka-22259439-d7ec-418e-bf17-31a7e68857de-opcs:9092
    2CIENVVAR      dwApp_elasticsearchBackupConfig_backupPath=
    2CIENVVAR      HOSTNAME=oes-rm-rm-service-1-r9hnr
    2CIENVVAR      OPENPALETTE_PG_USERNAME=GbVgULrB41qW5l
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_HTTP_PORT=9200
    2CIENVVAR      OPENPALETTE_KAFKA_ZOOKEEPER_PORT=2181
    2CIENVVAR      openpalette_srv_deployname=oes-rm
    2CIENVVAR      LD_LIBRARY_PATH=/lib64
    2CIENVVAR      JVM_TYPE=openj9
    2CIENVVAR      SHLVL=2
    2CIENVVAR      JAVA_GLOBAL_OPTS= -Xdump:heap:events=systhrow+user,filter=java/lang/OutOfMemoryError,request=exclusive+prepwalk+compact,label=/home/zenap/dump/dump-2018-12-29-14-32-13.phd -Xdump:none -Xdump:system:events=gpf+abort+traceassert,range=1..0,priority=999,request=serial,label=/home/zenap/dump/core-dump-2018-12-29-14-32-13.%seq.dmp -Xdump:heap:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=500,request=exclusive+compact+prepwalk,label=/home/zenap/dump/dump-dump-2018-12-29-14-32-13.%seq.phd -Xdump:heap:events=user,priority=500,request=exclusive+compact+prepwalk,label=/home/zenap/dump/dump-dump-user-2018-12-29-14-32-13.%seq.phd -Xdump:java:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=400,request=exclusive+preempt,label=/home/zenap/dump/javacore-dump-2018-12-29-14-32-13.%seq.txt -Xdump:java:events=gpf+abort+traceassert+user,priority=400,request=exclusive+preempt,label=/home/zenap/dump/javacore-dump-2018-12-29-14-32-13.%seq.txt -Xdump:snap:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=300,request=serial,label=/home/zenap/dump/snap-dump-2018-12-29-14-32-13.%seq.trc -Xdump:snap:events=gpf+abort+traceassert,priority=300,request=serial,label=/home/zenap/dump/snap-dump-2018-12-29-14-32-13.%seq.trc -Xverbosegclog:/home/zenap/gclog/gc-2018-12-29-14-32-13.log,10,20000 -Xquickstart  -Dfile.encoding=UTF-8 -Xlp:objectheap:pagesize=4K  -Xlp:codecache:pagesize=4K -XX:+UseContainerSupport 
    2CIENVVAR      HOME=/root
    2CIENVVAR      dwApp_msbClientConfig_msbSvrIp=193.168.0.7
    2CIENVVAR      dwApp_esclientconfig_username=c5ELmd
    2CIENVVAR      openpalette_container_name=rm-service
    2CIENVVAR      dwApp_kafkaClientConf_autoDiscover=false
    2CIENVVAR      redis_sentinel_port=26379
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_PASSWORD=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      JVM_GC_OPTS= -Xgcpolicy:gencon -Xgcthreads5 
    2CIENVVAR      dwApp_elasticsearchBackupConfig_user=
    2CIENVVAR      dwApp_msbClientConfig_msbSvrPort=10081
    2CIENVVAR      OPENPALETTE_LOGSTASH_OEM_TCP_PORT=4522
    2CIENVVAR      dwApp_cacheConfig_cacheServiceName=postgreCacheService
    2CIENVVAR      OPENPALETTE_LOGSTASH_INDEXER_HTTP_PORT=9601
    2CIENVVAR      OPENPALETTE_PGCACHE_PASSWORD=A1C8E13EE923482D12E902067D0CF18B767D872BB24D36F52A58D91529D78005
    2CIENVVAR      OPENPALETTE_REDIS_PASSWORD=7336FB784AB98B17581BC1C5572C53FA
    2CIENVVAR      MIN_DUMP_SIZE=2304m
    2CIENVVAR      OPENPALETTE_PG_DBNAME=db_076937aa67744efeb485191b90ce95f1
    2CIENVVAR      OPENPALETTE_LOGSTASH_COS_TCP_PORT=4525
    2CIENVVAR      OPENPALETTE_MSB_IP=193.168.0.7
    2CIENVVAR      redis_host=ha-redis-ad92e9f1-28c6-488b-9ec1-765705b7484e-opcs
    2CIENVVAR      MAX_DUMP_SIZE=9216m
    2CIENVVAR      dwApp_elasticsearchBackupConfig_password=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_ADDRESS=commsrves-bd3fd32a-cc24-4d11-b068-2016ff56b578-opcs
    2CIENVVAR      dwApp_cacheConfig_cacheServiceVersion=v1
    2CIENVVAR      dwApp_cacheConfig_userName=uSsE9zaNktxyc2
    2CIENVVAR      OPENPALETTE_LOGSTASH_UMF_TCP_PORT=4523
    2CIENVVAR      dwApp_database_user=GbVgULrB41qW5l
    2CIENVVAR      dwApp_esclientconfig_servers_node1=[commsrves-bd3fd32a-cc24-4d11-b068-2016ff56b578-opcs]:9300
    2CIENVVAR      OPENPALETTE_LOGSTASH_UMF_UDP_PORT=4524
    2CIENVVAR      KUBERNETES_PORT_443_TCP_ADDR=10.254.0.1
    2CIENVVAR      OPENPALETTE_PGCACHE_ADDRESS=pgcache-77202065-5022-4a99-b5d3-5bc90f5f3bfa-opcs
    2CIENVVAR      HTTP_PORT=13601
    2CIENVVAR      PATH=/openj9jdk/bin:/openj9jdk/bin:/openjdk/bin:/openj9jdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    2CIENVVAR      OPENPALETTE_REDIS_ADDRESS=ha-redis-ad92e9f1-28c6-488b-9ec1-765705b7484e-opcs
    2CIENVVAR      OPENPALETTE_REDIS_SENTINEL_MASTERNAME=mymaster
    2CIENVVAR      redis_port=6379
    2CIENVVAR      OPENPALETTE_PGCACHE_PORT=36789
    2CIENVVAR      KUBERNETES_PORT_443_TCP_PORT=443
    2CIENVVAR      MEM_LIMIT=10240
    2CIENVVAR      dwApp_kafkaClientConf_zkServers=cskafka-22259439-d7ec-418e-bf17-31a7e68857de-zk-opcs:2181
    2CIENVVAR      redis_password=7336FB784AB98B17581BC1C5572C53FA
    2CIENVVAR      OPENPALETTE_REDIS_PORT=6379
    2CIENVVAR      OPENPALETTE_LOGSTASH_SHIPPER_HTTP_PORT=9600
    2CIENVVAR      OPENPALETTE_LOGSTASH_INDEXER_ADDRESS=logstash-6aa255e5-69c8-4892-998a-fb7d3ffa9fbb-opcs
    2CIENVVAR      OPENPALETTE_PG_PASSWORD=FAF05ACBBCC3A6B615D574DE99F1A3E9047687412D42FDB6BB715F13EFAA2348
    2CIENVVAR      KUBERNETES_PORT_443_TCP_PROTO=tcp
    2CIENVVAR      dwApp_elasticsearchConfig_tcpPort=9300
    2CIENVVAR      logstash_ip=logstash-6aa255e5-69c8-4892-998a-fb7d3ffa9fbb-opcs
    2CIENVVAR      OPENPALETTE_REDIS_SENTINEL_ADDRESS=ha-redis-ad92e9f1-28c6-488b-9ec1-765705b7484e-sentinel-opcs
    2CIENVVAR      openpalette_ms_bpname=rm-service
    2CIENVVAR      LANG=C.UTF-8
    2CIENVVAR      dwApp_cacheConfig_type=postgresql
    2CIENVVAR      dwApp_database_password=FAF05ACBBCC3A6B615D574DE99F1A3E9047687412D42FDB6BB715F13EFAA2348
    2CIENVVAR      dwApp_esclientconfig_clusterName=elasticsearch5
    2CIENVVAR      OPENPALETTE_LOGSTASH_INDEXER_PORT=4521
    2CIENVVAR      logstash_port=4522
    2CIENVVAR      dwApp_esclientconfig_password=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      OPENPALETTE_REDIS_SENTINEL_PORT=26379
    2CIENVVAR      dwApp_cacheConfig_autoDiscover=false
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_USERNAME=c5ELmd
    2CIENVVAR      dwApp_cacheConfig_password=A1C8E13EE923482D12E902067D0CF18B767D872BB24D36F52A58D91529D78005
    2CIENVVAR      dwApp_elasticsearchConfig_userName=c5ELmd
    2CIENVVAR      OPENPALETTE_MSB_PORT=10081
    2CIENVVAR      rdb_ip=commsrvpg-0d467525-8562-4fac-ab50-55dc7aeb1f51-opcs
    2CIENVVAR      dwApp_esclientconfig_security=1
    2CIENVVAR      OPENPALETTE_PG_ADDRESS=commsrvpg-0d467525-8562-4fac-ab50-55dc7aeb1f51-opcs
    2CIENVVAR      openpalette_ms_uuid=09f726d7-bb1e-45f5-9602-cb64530e580b
    2CIENVVAR      KUBERNETES_SERVICE_PORT_HTTPS=443
    2CIENVVAR      KUBERNETES_PORT_443_TCP=tcp://10.254.0.1:443
    2CIENVVAR      OPENPALETTE_PGCACHE_USERNAME=uSsE9zaNktxyc2
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_TCP_PORT=9300
    2CIENVVAR      dwApp_cacheConfig_cacheServerIp=pgcache-77202065-5022-4a99-b5d3-5bc90f5f3bfa-opcs
    2CIENVVAR      dwApp_cacheConfig_asNamespace=db_fa3ccb8f66544d75b191be4e26338b27
    2CIENVVAR      rdb_port=36789
    2CIENVVAR      OPENPALETTE_PG_PORT=36789
    2CIENVVAR      KUBERNETES_SERVICE_HOST=10.254.0.1
    2CIENVVAR      JAVA_HOME=/openj9jdk
    2CIENVVAR      PWD=/home/zenap
    2CIENVVAR      DBCONF=ip=194.167.0.6:3000
    2CIENVVAR      DEBUG_PORT=8777
    2CIENVVAR      dwApp_cacheConfig_cacheServerPort=36789
    2CIENVVAR      OPENPALETTE_LOGSTASH_SHIPPER_ADDRESS=logstash-6aa255e5-69c8-4892-998a-fb7d3ffa9fbb-opcs
    2CIENVVAR      OPENPALETTE_KAFKA_ADDRESS=cskafka-22259439-d7ec-418e-bf17-31a7e68857de-opcs
    2CIENVVAR      dwApp_elasticsearchConfig_securityReinforce=1
    2CIENVVAR      TZ=Asia/Shanghai
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_SECURITY_REINFORCE=1
    2CIENVVAR      OPENPALETTE_NAMESPACE=ranoss
    2CIENVVAR      openpalette_srv_uuid=c21f9df1-96a8-4a89-8b23-9728420985a2
    2CIENVVAR      OPENPALETTE_KAFKA_PORT=9092
    2CIENVVAR      OPENPALETTE_LOGSTASH_SHIPPER_PORT=4522
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_CLUSTER_NAME=elasticsearch5
    2CIENVVAR      dwApp_elasticsearchConfig_ip=commsrves-bd3fd32a-cc24-4d11-b068-2016ff56b578-opcs
    2CIENVVAR      rdb_dbname=db_076937aa67744efeb485191b90ce95f1
    2CIENVVAR      redis_sentinel_mastername=mymaster
    
观察CPU核数：物理核数40，分配给应用的3个核

    2CIPHYSCPU     Physical CPUs: 40
    2CIONLNCPU     Online CPUs: 40
    2CIBOUNDCPU    Bound CPUs: 3
    2CITARGETCPU   Target CPUs: 3
    
观察堆内存的使用情况：堆内存总计9G(=Xmx)，使用了7.2G，使用率80%，按理说堆内存并没有使用完

    1STHEAPTOTAL   Total memory:                  9663676416 (0x0000000240000000)
    1STHEAPINUSE   Total memory in use:           7758403992 (0x00000001CE6FD998)
    1STHEAPFREE    Total memory free:             1905272424 (0x0000000071902668)
    
观察堆内存的分配情况：老年代占用6.75G，两个新生代分别占用1.98G和275.9M，三个区域合计9G(=Xmx,=Total memory)，与上面的堆内存完全一致。


    1STHEAPTYPE    Object Memory
    NULL           id                 start              end                size               space/region
    1STHEAPSPACE   0x00007F39B4082C00         --                 --                 --         Generational 
    1STHEAPREGION  0x00007F39B40830F0 0x00000005C0000000 0x0000000770000000 0x00000001B0000000 Generational/Tenured Region 6.75G  7247757312
    1STHEAPREGION  0x00007F39B4082DD0 0x0000000770000000 0x00000007813E0000 0x00000000113E0000 Generational/Nursery Region 275.875M  289275904
    1STHEAPREGION  0x00007F39B4082CE0 0x00000007813E0000 0x0000000800000000 0x000000007EC20000 Generational/Nursery Region 1.98G  2126643200
    
    J9AllocateIndexableObject() returning NULL! 329392 bytes requested for object of class 000000000067FC00 from memory space 'Generational' id=00007F39B4082C00 

结合后面的GC信息来看，虚拟机一直从00007F39B4082C00这个空间申请内存，由于三个区域内存总和已经达到最大堆内存，因此申请不到。
但是最大的问题是，为什么只申请322k这么小的空间，不能到前面两个新生代的碎片中寻找？如果两个新生代没有任何的碎片，为什么Total memory free有1.8G之多？
很可能说明Total memory free的值不准确

观察GC情况：频繁从00007F39B4082C00这个空间申请内存，短短5秒进行了7次Global GC，可以看到申请的空间都不大，但仍申请不到。
最后Forcing J9AllocateIndexableObject() to fail due to excessive GC 
说明GC过度使用,很可能本例中是由于GC使用频繁达到上限后，触发的OOM

    1STGCHTYPE     GC History  
    3STHSTTYPE     08:48:06:800805000 GMT j9mm.101 -   J9AllocateIndexableObject() returning NULL! 329392 bytes requested for object of class 000000000067FC00 from memory space 'Generational' id=00007F39B4082C00 
    3STHSTTYPE     08:48:06:800804000 GMT j9mm.84 -   Forcing J9AllocateIndexableObject() to fail due to excessive GC 
    3STHSTTYPE     08:48:06:800174000 GMT j9mm.134 -   Allocation failure end: newspace=1832822760/2415919104 oldspace=72120272/7247757312 loa=72120272/72476672 
    3STHSTTYPE     08:48:06:799911000 GMT j9mm.470 -   Allocation failure cycle end: newspace=1832822760/2415919104 oldspace=72120272/7247757312 loa=72120272/72476672 
    3STHSTTYPE     08:48:06:799529000 GMT j9mm.560 -   LocalGC end: rememberedsetoverflow=0 causedrememberedsetoverflow=0 scancacheoverflow=0 failedflipcount=1 failedflipbytes=131080 failedtenurecount=6832777 failedtenurebytes=289040616 flipcount=6832775 flipbytes=288895056 newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 tenureage=0 
    3STHSTTYPE     08:48:06:798895000 GMT j9mm.140 -   Tilt ratio: 88 
    3STHSTTYPE     08:48:05:640728000 GMT j9mm.63 -   Set scavenger backout flag=true 
    3STHSTTYPE     08:48:04:916866000 GMT j9mm.64 -   LocalGC start: globalcount=140 scavengecount=1890 weakrefs=0 soft=0 phantom=0 finalizers=0 
    3STHSTTYPE     08:48:04:916699000 GMT j9mm.63 -   Set scavenger backout flag=false 
    3STHSTTYPE     08:48:04:911150000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=0.151 meanexclusiveaccessms=0.122 threads=1 lastthreadtid=0x0000000000DCC060 beatenbyotherthread=1 
    3STHSTTYPE     08:48:04:911149000 GMT j9mm.469 -   Allocation failure cycle start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=329392 
    3STHSTTYPE     08:48:04:911135000 GMT j9mm.133 -   Allocation failure start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=329392 
    3STHSTTYPE     08:48:04:910535000 GMT j9mm.101 -   J9AllocateIndexableObject() returning NULL! 72 bytes requested for object of class 0000000000670200 from memory space 'Generational' id=00007F39B4082C00 
    3STHSTTYPE     08:48:04:908312000 GMT j9mm.134 -   Allocation failure end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    3STHSTTYPE     08:48:04:908301000 GMT j9mm.470 -   Allocation failure cycle end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    3STHSTTYPE     08:48:04:907905000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=1905272424/9663676416 
    3STHSTTYPE     08:48:04:907434000 GMT j9mm.90 -   GlobalGC collect complete 
    3STHSTTYPE     08:48:04:906663000 GMT j9mm.137 -   Compact end: bytesmoved=0 
    3STHSTTYPE     08:47:30:793585000 GMT j9mm.136 -   Compact start: reason=low free space (less than 4%) 
    3STHSTTYPE     08:47:30:793485000 GMT j9mm.57 -   Sweep end 
    3STHSTTYPE     08:47:30:286766000 GMT j9mm.56 -   Sweep start 
    3STHSTTYPE     08:47:30:286738000 GMT j9mm.94 -   Class unloading end: classloadersunloaded=0 classesunloaded=0 
    3STHSTTYPE     08:47:30:286601000 GMT j9mm.60 -   Class unloading start 
    3STHSTTYPE     08:47:30:286535000 GMT j9mm.55 -   Mark end 
    3STHSTTYPE     08:47:21:352798000 GMT j9mm.54 -   Mark start 
    3STHSTTYPE     08:47:21:352650000 GMT j9mm.474 -   GlobalGC start: globalcount=139 
    3STHSTTYPE     08:47:21:350631000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=1905272424/9663676416 
    3STHSTTYPE     08:47:21:350174000 GMT j9mm.90 -   GlobalGC collect complete 
    3STHSTTYPE     08:47:21:349336000 GMT j9mm.137 -   Compact end: bytesmoved=0 
    3STHSTTYPE     08:46:46:987908000 GMT j9mm.136 -   Compact start: reason=low free space (less than 4%) 
    3STHSTTYPE     08:46:46:987783000 GMT j9mm.57 -   Sweep end 
    3STHSTTYPE     08:46:46:358371000 GMT j9mm.56 -   Sweep start 
    3STHSTTYPE     08:46:46:358357000 GMT j9mm.94 -   Class unloading end: classloadersunloaded=0 classesunloaded=0 
    3STHSTTYPE     08:46:46:358272000 GMT j9mm.60 -   Class unloading start 
    3STHSTTYPE     08:46:46:358216000 GMT j9mm.55 -   Mark end 
    3STHSTTYPE     08:46:37:239844000 GMT j9mm.54 -   Mark start 
    3STHSTTYPE     08:46:37:239692000 GMT j9mm.474 -   GlobalGC start: globalcount=138 
    3STHSTTYPE     08:46:37:239575000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=4.811 meanexclusiveaccessms=4.811 threads=0 lastthreadtid=0x0000000000DCC060 beatenbyotherthread=0 
    3STHSTTYPE     08:46:37:239574000 GMT j9mm.469 -   Allocation failure cycle start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=72 
    3STHSTTYPE     08:46:37:239247000 GMT j9mm.470 -   Allocation failure cycle end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    3STHSTTYPE     08:46:37:238715000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=1905272424/9663676416 
    3STHSTTYPE     08:46:37:238218000 GMT j9mm.90 -   GlobalGC collect complete 
    3STHSTTYPE     08:46:37:237166000 GMT j9mm.137 -   Compact end: bytesmoved=0 
    3STHSTTYPE     08:46:00:507750000 GMT j9mm.136 -   Compact start: reason=low free space (less than 4%) 
    3STHSTTYPE     08:46:00:507607000 GMT j9mm.57 -   Sweep end 
    3STHSTTYPE     08:45:59:920348000 GMT j9mm.56 -   Sweep start 
    3STHSTTYPE     08:45:59:920334000 GMT j9mm.94 -   Class unloading end: classloadersunloaded=0 classesunloaded=0 
    3STHSTTYPE     08:45:59:920240000 GMT j9mm.60 -   Class unloading start 
    3STHSTTYPE     08:45:59:920173000 GMT j9mm.55 -   Mark end 
    3STHSTTYPE     08:45:50:828059000 GMT j9mm.54 -   Mark start 
    3STHSTTYPE     08:45:50:827912000 GMT j9mm.474 -   GlobalGC start: globalcount=137 
    3STHSTTYPE     08:45:50:827714000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=4.811 meanexclusiveaccessms=4.811 threads=0 lastthreadtid=0x0000000000DCC060 beatenbyotherthread=0 
    3STHSTTYPE     08:45:50:827712000 GMT j9mm.469 -   Allocation failure cycle start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=72 
    3STHSTTYPE     08:45:50:827509000 GMT j9mm.470 -   Allocation failure cycle end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    3STHSTTYPE     08:45:50:826930000 GMT j9mm.560 -   LocalGC end: rememberedsetoverflow=0 causedrememberedsetoverflow=0 scancacheoverflow=0 failedflipcount=1 failedflipbytes=160 failedtenurecount=6733084 failedtenurebytes=289103120 flipcount=6733082 flipbytes=289088424 newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 tenureage=0 
    3STHSTTYPE     08:45:50:821070000 GMT j9mm.140 -   Tilt ratio: 88 
    3STHSTTYPE     08:45:49:720989000 GMT j9mm.63 -   Set scavenger backout flag=true 
    3STHSTTYPE     08:45:48:987417000 GMT j9mm.64 -   LocalGC start: globalcount=137 scavengecount=1889 weakrefs=0 soft=0 phantom=0 finalizers=0 
    3STHSTTYPE     08:45:48:987403000 GMT j9mm.63 -   Set scavenger backout flag=false 
    3STHSTTYPE     08:45:48:987106000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=4.811 meanexclusiveaccessms=4.811 threads=0 lastthreadtid=0x0000000000DCC060 beatenbyotherthread=1 
    3STHSTTYPE     08:45:48:987105000 GMT j9mm.469 -   Allocation failure cycle start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=72 
    3STHSTTYPE     08:45:48:987090000 GMT j9mm.133 -   Allocation failure start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=72 
    3STHSTTYPE     08:45:48:981118000 GMT j9mm.100 -   J9AllocateObject() returning NULL! 48 bytes requested for object of class 0000000000DF7A00 from memory space 'Generational' id=00007F39B4082C00 
    3STHSTTYPE     08:45:48:979681000 GMT j9mm.134 -   Allocation failure end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    3STHSTTYPE     08:45:48:979669000 GMT j9mm.470 -   Allocation failure cycle end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    3STHSTTYPE     08:45:48:979205000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=1905272424/9663676416 
    3STHSTTYPE     08:45:48:978713000 GMT j9mm.90 -   GlobalGC collect complete 
    3STHSTTYPE     08:45:48:977627000 GMT j9mm.137 -   Compact end: bytesmoved=0 
    3STHSTTYPE     08:45:13:158827000 GMT j9mm.136 -   Compact start: reason=low free space (less than 4%) 
    3STHSTTYPE     08:45:13:158740000 GMT j9mm.57 -   Sweep end 
    3STHSTTYPE     08:45:12:550519000 GMT j9mm.56 -   Sweep start 
    3STHSTTYPE     08:45:12:550504000 GMT j9mm.94 -   Class unloading end: classloadersunloaded=0 classesunloaded=0 
    3STHSTTYPE     08:45:12:550111000 GMT j9mm.60 -   Class unloading start 
    3STHSTTYPE     08:45:12:550047000 GMT j9mm.55 -   Mark end 
    3STHSTTYPE     08:45:03:199409000 GMT j9mm.54 -   Mark start 
    3STHSTTYPE     08:45:03:199246000 GMT j9mm.474 -   GlobalGC start: globalcount=136 
    3STHSTTYPE     08:45:03:198464000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=1905272424/9663676416 
    3STHSTTYPE     08:45:03:198002000 GMT j9mm.90 -   GlobalGC collect complete 
    3STHSTTYPE     08:45:03:197064000 GMT j9mm.137 -   Compact end: bytesmoved=0 
    3STHSTTYPE     08:44:25:968933000 GMT j9mm.136 -   Compact start: reason=low free space (less than 4%) 
    3STHSTTYPE     08:44:25:968798000 GMT j9mm.57 -   Sweep end 
    3STHSTTYPE     08:44:25:325953000 GMT j9mm.56 -   Sweep start 
    3STHSTTYPE     08:44:25:325927000 GMT j9mm.94 -   Class unloading end: classloadersunloaded=0 classesunloaded=0 
    3STHSTTYPE     08:44:25:323631000 GMT j9mm.60 -   Class unloading start 
    3STHSTTYPE     08:44:25:321137000 GMT j9mm.55 -   Mark end 
    3STHSTTYPE     08:44:16:487603000 GMT j9mm.54 -   Mark start 
    3STHSTTYPE     08:44:16:487458000 GMT j9mm.474 -   GlobalGC start: globalcount=135 
    3STHSTTYPE     08:44:16:487343000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=0.264 meanexclusiveaccessms=0.264 threads=0 lastthreadtid=0x0000000000E00C60 beatenbyotherthread=0 
    3STHSTTYPE     08:44:16:487341000 GMT j9mm.469 -   Allocation failure cycle start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=48 
    3STHSTTYPE     08:44:16:483895000 GMT j9mm.470 -   Allocation failure cycle end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    3STHSTTYPE     08:44:16:481152000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=1905272424/9663676416 
    3STHSTTYPE     08:44:16:480663000 GMT j9mm.90 -   GlobalGC collect complete 
    3STHSTTYPE     08:44:16:479800000 GMT j9mm.137 -   Compact end: bytesmoved=0 
    3STHSTTYPE     08:43:39:595774000 GMT j9mm.136 -   Compact start: reason=low free space (less than 4%) 
    3STHSTTYPE     08:43:39:595655000 GMT j9mm.57 -   Sweep end 
    3STHSTTYPE     08:43:39:017164000 GMT j9mm.56 -   Sweep start 
    3STHSTTYPE     08:43:39:017149000 GMT j9mm.94 -   Class unloading end: classloadersunloaded=0 classesunloaded=0 
    3STHSTTYPE     08:43:39:017051000 GMT j9mm.60 -   Class unloading start 
    3STHSTTYPE     08:43:39:016987000 GMT j9mm.55 -   Mark end 
    3STHSTTYPE     08:43:29:639856000 GMT j9mm.54 -   Mark start 
    3STHSTTYPE     08:43:29:639708000 GMT j9mm.474 -   GlobalGC start: globalcount=134 
    3STHSTTYPE     08:43:29:639520000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=0.264 meanexclusiveaccessms=0.264 threads=0 lastthreadtid=0x0000000000E00C60 beatenbyotherthread=0 
    3STHSTTYPE     08:43:29:639519000 GMT j9mm.469 -   Allocation failure cycle start: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 requestedbytes=48 
    3STHSTTYPE     08:43:29:639325000 GMT j9mm.470 -   Allocation failure cycle end: newspace=1832822760/2415919104 oldspace=72449664/7247757312 loa=72449664/72476672 
    NULL 
    
观察死锁：无
    
观察等待的锁：使用3LKWAITER关键字搜索

    3LKMONOBJECT       org/apache/kafka/clients/consumer/internals/ConsumerNetworkClient@0x00000005C0713318: owner "BroadcastReceiver" (J9VMThread:0x0000000000DE5300), entry count 1
    3LKWAITERQ            Waiting to enter:
    3LKWAITER                "kafka-coordinator-heartbeat-thread | BroadCastState_zenap_b2ad1446-d5c2-43b3-98a4-e68e1b1e060fa6a8321c-1552-4217-a0dd-5729700e8124" (J9VMThread:0x00000000013A3A00)

观察阻塞的线程：使用3XMTHREADBLOCK关键字搜索，大量的请求处理线程都Parked，原因是dw-274线程持有的一个锁(0x00000005FB64DBF0)，但这个锁(0x00000005FB64DBF0)在LOCKS部分没有列出来    

    3XMTHREADBLOCK     Blocked on: org/apache/kafka/clients/consumer/internals/ConsumerNetworkClient@0x00000005C0713090 Owned by: "QueryReceiver" (J9VMThread:0x0000000001273E00, java/lang/Thread:0x00000005C0712EB0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Waiting on: org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091AA00 Owned by: <unowned>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Waiting on: org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091AC30 Owned by: <unowned>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005C07128B0 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005C0711E88 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005C0206620 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FFE878C8 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005C02134F0 Owned by: <unknown>
    3XMTHREADBLOCK     Waiting on: org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091A7D0 Owned by: <unowned>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005C02064D8 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FFE878C8 Owned by: <unknown>
    3XMTHREADBLOCK     Blocked on: org/apache/kafka/clients/consumer/internals/ConsumerNetworkClient@0x00000005FB623350 Owned by: "configclient-consumer-thread" (J9VMThread:0x0000000001701A00, java/lang/Thread:0x00000005FB623160)
    3XMTHREADBLOCK     Waiting on: org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091A5A0 Owned by: <unowned>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Waiting on: org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091AE60 Owned by: <unowned>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FFE878C8 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x0000000639A02DB8 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Blocked on: org/apache/kafka/clients/consumer/internals/ConsumerNetworkClient@0x00000005C0713318 Owned by: "BroadcastReceiver" (J9VMThread:0x0000000000DE5300, java/lang/Thread:0x00000005C07135C8)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FB623600 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FFE878C8 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FFE878C8 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005C0712B58 Owned by: <unknown>
    3XMTHREADBLOCK     Waiting on: org/apache/kafka/clients/consumer/internals/ConsumerCoordinator@0x00000005C091A370 Owned by: <unowned>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FA34E228 Owned by: <unknown>
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00000005FB64DBF0 Owned by: "dw-274" (J9VMThread:0x00000000018E2100, java/lang/Thread:0x0000000765D5D3B0)
    3XMTHREADBLOCK     Parked on: java/util/concurrent/locks/AbstractQueuedSynchronizer$ConditionObject@0x00000005FFE878C8 Owned by: <unknown>
    3XMTHREADBLOCK     Blocked on: org/apache/kafka/clients/consumer/internals/ConsumerNetworkClient@0x00000005FB63ABC8 Owned by: "AuthLogoutListener" (J9VMThread:0x0000000001705500, java/lang/Thread:0x00000005FB63A9D8)
     
再搜索dw-274，发现这个dw线程没有处理任何请求。其他dw线程都由于GC而等待被唤醒，感觉THREADS部分和LOCKS部分对不上
    
    2LKREGMON          GCExtensions::gcExclusiveAccessMutex lock (0x00007F39B4004EE8): <unowned>
    3LKNOTIFYQ            Waiting to be notified:
    3LKWAITNOTIFY            "MemoryMXBean notification dispatcher" (J9VMThread:0x0000000000BEDB00)
    3LKWAITNOTIFY            "logback-1" (J9VMThread:0x0000000000E00300)
    3LKWAITNOTIFY            "logback-2" (J9VMThread:0x0000000000DE2300)
    3LKWAITNOTIFY            "metrics-console-reporter-1-thread-1" (J9VMThread:0x00000000010A0000)
    3LKWAITNOTIFY            "Timer-0" (J9VMThread:0x00000000010DA500)
    3LKWAITNOTIFY            "kafka-producer-network-thread | zenap_kafka_100.100.5.41_rm-service-producer1" (J9VMThread:0x00000000011FE100)
    3LKWAITNOTIFY            "dw-273 - GET /api/res/v1/subnetworks/06670f87-1e4f-49df-9910-858028199616/fddv3.eutrancellfdds?includeAttr=userLabel,nbiIdDn&queryDn=true" (J9VMThread:0x0000000001A60D00)
    3LKWAITNOTIFY            "dw-274" (J9VMThread:0x00000000018E2100)
    3LKWAITNOTIFY            "dw-275 - POST /api/res/v1/toolkits/id2name" (J9VMThread:0x0000000001A68400)
    
    ...


    
    