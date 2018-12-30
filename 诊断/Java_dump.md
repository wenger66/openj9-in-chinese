# Java dump (https://www.eclipse.org/openj9/docs/dump_javadump/)
## 格式
### 文件格式
Java dump 通常是文本格式(.txt)，因此可以通过一般的文本编辑器进行阅读，阅读时需要注意段与行的格式
### 段落格式
* 每一段的开头，都会用“-----”与上一段明显的区分开来
* 每一段的标题也会用“=====”作为标识
### 行格式
每一行都包含一个标签，这个标签最多由 15 个字符组成。

其中第一位数字代表信息的详细级别（0，1，2，3，4），级别越高代表信息越详细

接下来的两个字符是段标题的缩写，比如
* CI - Command-line interpreter
* CL - Class loader
* LK - Locking
* ST - Storage
* TI - Title
* XE - Execution engine

其余部分为信息的概述

## 内容
Java dump汇总了事件发生时虚拟机的状态，包括大部分虚拟机组件的信息，dump文件由多个部分组成，每个部分提供了不同的信息

感觉段落用英文更加专业，无歧义
### TITLE 
TITLE段落包含了触发dump的事件信息。下面的例子中，你可以看到是vmstop事件在2018/08/30 21:55:47这个时间触发了这个dump

    0SECTION       TITLE subcomponent dump routine
    NULL           ===============================
    1TICHARSET     UTF-8
    1TISIGINFO     Dump Event "vmstop" (00000002) Detail "#0000000000000000" received
    1TIDATETIME    Date: 2018/08/30 at 21:55:47:607
    1TINANOTIME    System nanotime: 22012355276134
    1TIFILENAME    Javacore filename:    /home/doc-javacore/javacore.20180830.215547.30285.0001.txt
    1TIREQFLAGS    Request Flags: 0x81 (exclusive+preempt)
    1TIPREPSTATE   Prep State: 0x106 (vm_access+exclusive_vm_access+trace_disabled)  

Dump Event参数可以参考[Dump Events](https://github.com/wenger66/openj9-in-chinese/blob/master/%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0/JVM%20-X%E5%8F%82%E6%95%B0/-Xdump.md#dump%E4%BA%8B%E4%BB%B6)

### GPINFO
GPINFO段落包含了运行虚拟机的操作系统的信息。下面的例子，说明虚拟机是运行在Linux系统上的

    NULL           ------------------------------------------------------------------------
    0SECTION       GPINFO subcomponent dump routine
    NULL           ================================
    2XHOSLEVEL     OS Level         : Linux 3.10.0-862.11.6.el7.x86_64
    2XHCPUS        Processors -
    3XHCPUARCH       Architecture   : amd64
    3XHNUMCPUS       How Many       : 4
    3XHNUMASUP       NUMA is either not supported or has been disabled by user
    NULL           
    1XHERROR2      Register dump section only produced for SIGSEGV, SIGILL or SIGFPE.
    NULL
    
这段包含的内容可能因为dump的原因不同而多种多样。例如如果是因为gpf事件触发的dump，会用VM Flags记录导致崩溃的库，通过VM Flags这个值
可以追溯到VM的组件。比如下面这段

    1XHFLAGS       VM flags:0000000000000000  
    
VM Flags的十六进制的数字以MSSSS格式结束。M表示虚拟机，SSSS码表示具体的组件，如下表格：

|   Component   |   Code value |
| --------   | -----:   | 
|INTERPRETER	|   0x10000    |
|GC	|0x20000|
|GROW_STACK	|0x30000|
|JNI	|0x40000|
|JIT_CODEGEN	|0x50000|
|BCVERIFY	|0x60000|
|RTVERIFY	|0x70000|
|SHAREDCLASSES	|0x80000|

0000000000000000 (0x00000) 表示崩溃不是由虚拟机造成的

### ENVINFO
ENVINFO段落包含了程序运行环境的很多有用信息
* Java版本（1CIJAVAVERSION）
* OpenJ9虚拟机版本及其组件版本（1CIVMVERSION, 1CIJ9VMVERSION, 1CIJITVERSION, 1CIOMRVERSION, 1CIJCLVERSION）
* 虚拟机启动时间（1CISTARTTIME）
* 进程ID（1CIPROCESSID）
* Jave home文件夹（1CIJAVAHOMEDIR）
* Java dll文件夹（1CIJAVADLLDIR）
* 用户设置的Java参数（1CIUSERARG）
* 用户进程受到的资源限制信息（1CIUSERLIMITS）
* 环境变量（1CIENVVARS）
* 系统信息（1CISYSINFO）
* CPU信息（1CICPUINFO）


    NULL           ------------------------------------------------------------------------
    0SECTION       ENVINFO subcomponent dump routine
    NULL           =================================
    1CIJAVAVERSION JRE 9 Linux amd64-64 (build 9.0.4-internal+0-adhoc..openj9-openjdk-jdk9)
    1CIVMVERSION   20180830_000000
    1CIJ9VMVERSION 8e7c6ec
    1CIJITVERSION  8e7c6ec
    1CIOMRVERSION  553811b_CMPRSS
    1CIJCLVERSION  ec1d223 based on jdk-9.0.4+12
    1CIJITMODES    JIT enabled, AOT enabled, FSD disabled, HCR enabled
    1CIRUNNINGAS   Running as a standalone JVM
    1CIVMIDLESTATE VM Idle State: ACTIVE
    1CISTARTTIME   JVM start time: 2018/08/30 at 21:55:47:387
    1CISTARTNANO   JVM start nanotime: 22012135233549
    1CIPROCESSID   Process ID: 30285 (0x764D)
    1CICMDLINE     [not available]
    1CIJAVAHOMEDIR Java Home Dir:   /home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk
    1CIJAVADLLDIR  Java DLL Dir:    /home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk/bin
    1CISYSCP       Sys Classpath:   
    1CIUSERARGS    UserArgs:
    2CIUSERARG               -Xoptionsfile=/home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk/lib/options.default
    ...
    NULL
    1CIUSERLIMITS  User Limits (in bytes except for NOFILE and NPROC)
    NULL           ------------------------------------------------------------------------
    NULL           type                            soft limit           hard limit
    2CIUSERLIMIT   RLIMIT_AS                        unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_CORE                              0            unlimited
    2CIUSERLIMIT   RLIMIT_CPU                       unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_DATA                      unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_FSIZE                     unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_LOCKS                     unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_MEMLOCK                       65536                65536
    2CIUSERLIMIT   RLIMIT_NOFILE                         4096                 4096
    2CIUSERLIMIT   RLIMIT_NPROC                          4096                30592
    2CIUSERLIMIT   RLIMIT_RSS                       unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_STACK                       8388608            unlimited
    2CIUSERLIMIT   RLIMIT_MSGQUEUE                     819200               819200
    2CIUSERLIMIT   RLIMIT_NICE                              0                    0
    2CIUSERLIMIT   RLIMIT_RTPRIO                            0                    0
    2CIUSERLIMIT   RLIMIT_SIGPENDING                    30592                30592
    NULL
    1CIENVVARS     Environment Variables
    NULL           ------------------------------------------------------------------------
    2CIENVVAR      XDG_VTNR=1
    2CIENVVAR      SSH_AGENT_PID=2653
    ...
    NULL           
    1CISYSINFO     System Information
    NULL           ------------------------------------------------------------------------
    2CISYSINFO     /proc/sys/kernel/core_pattern = core
    2CISYSINFO     /proc/sys/kernel/core_uses_pid = 1
    NULL          
    1CICPUINFO     CPU Information
    NULL           ------------------------------------------------------------------------
    2CIPHYSCPU     Physical CPUs: 4
    2CIONLNCPU     Online CPUs: 4
    2CIBOUNDCPU    Bound CPUs: 4
    2CIACTIVECPU   Active CPUs: 0
    2CITARGETCPU   Target CPUs: 4

资源限制（1CIUSERLIMITS）部分，除了NOFILE和NPROC类，其他都是字节为单位（好像不对，例如RLIMIT_CPU）。在Linux系统中，Resouce Limit指在一个进程的执行过程中，它受到的资源的限制
* RLIMIT_AS - 进程的最大虚内存空间，字节为单位。
* RLIMIT_CORE - 内核转存文件的最大长度。
* RLIMIT_CPU - 最大允许的CPU使用时间，秒为单位。当进程达到软限制，内核将给其发送SIGXCPU信号，这一信号的默认行为是终止进程的执行。然而，可以捕捉信号，处理句柄可将控制返回给主程序。如果进程继续耗费CPU时间，核心会以每秒一次的频率给其发送SIGXCPU信号，直到达到硬限制，那时将给进程发送 SIGKILL信号终止其执行。
* RLIMIT_DATA - 进程数据段的最大值。
* RLIMIT_FSIZE - 进程可建立的文件的最大长度。如果进程试图超出这一限制时，核心会给其发送SIGXFSZ信号，默认情况下将终止进程的执行。
* RLIMIT_LOCKS - 进程可建立的锁和租赁的最大值。
* RLIMIT_MEMLOCK - 进程可锁定在内存中的最大数据量，字节为单位。
* RLIMIT_MSGQUEUE - 进程可为POSIX消息队列分配的最大字节数。
* RLIMIT_NICE - 进程可通过setpriority() 或 nice()调用设置的最大完美值。
* RLIMIT_NOFILE - 指定比进程可打开的最大文件数值，超出此值，将会产生EMFILE错误。
* RLIMIT_NPROC - 用户可拥有的最大进程数。
* RLIMIT_RTPRIO - 进程可通过sched_setscheduler 和 sched_setparam设置的最大实时优先级。
* RLIMIT_SIGPENDING - 用户可拥有的最大挂起信号数。
* RLIMIT_STACK - 最大的进程堆栈，以字节为单位。


    1CIUSERLIMITS  User Limits (in bytes except for NOFILE and NPROC)
    NULL           ------------------------------------------------------------------------
    NULL           type                            soft limit           hard limit
    2CIUSERLIMIT   RLIMIT_AS                        unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_CORE                      unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_CPU                       unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_DATA                      unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_FSIZE                     unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_LOCKS                     unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_MEMLOCK                       65536                65536
    2CIUSERLIMIT   RLIMIT_NOFILE                        65536                65536
    2CIUSERLIMIT   RLIMIT_NPROC                     unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_RSS                       unlimited            unlimited
    2CIUSERLIMIT   RLIMIT_STACK                       8388608            unlimited
    2CIUSERLIMIT   RLIMIT_MSGQUEUE                     819200               819200
    2CIUSERLIMIT   RLIMIT_NICE                              0                    0
    2CIUSERLIMIT   RLIMIT_RTPRIO                            0                    0
    2CIUSERLIMIT   RLIMIT_SIGPENDING                    95712                95712
    
环境变量（1CIENVVARS）部分，会打印出所有的环境变量，对于docker环境非常有用

    1CIENVVARS     Environment Variables
    NULL           ------------------------------------------------------------------------
    2CIENVVAR      KUBERNETES_SERVICE_PORT=443
    2CIENVVAR      KUBERNETES_PORT=tcp://10.254.0.1:443
    2CIENVVAR      dwApp_elasticsearchConfig_autoDiscover=false
    2CIENVVAR      dwApp_elasticsearchConfig_port=9200
    2CIENVVAR      dwApp_elasticsearchConfig_clusterName=elasticsearch5
    2CIENVVAR      dwApp_elasticsearchConfig_password=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      OPENPALETTE_PGCACHE_DBNAME=db_4aa5664477b347518ae829590d0a29d3
    2CIENVVAR      OPENPALETTE_KAFKA_ZOOKEEPER_ADDRESS=cskafka-273de3a3-3772-4e83-990e-ad6f52b452e8-zk-opcs
    2CIENVVAR      CPU_CORE_NUM=1
    2CIENVVAR      dwApp_kafkaClientConf_bootstrapServers=cskafka-273de3a3-3772-4e83-990e-ad6f52b452e8-opcs:9092
    2CIENVVAR      dwApp_elasticsearchBackupConfig_backupPath=
    2CIENVVAR      HOSTNAME=oes-rm-rm-service-1-fwgtq
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_HTTP_PORT=9200
    2CIENVVAR      OPENPALETTE_PG_USERNAME=UW5KhXPBIaDMpO
    2CIENVVAR      openpalette_srv_deployname=oes-rm
    2CIENVVAR      OPENPALETTE_KAFKA_ZOOKEEPER_PORT=2181
    2CIENVVAR      LD_LIBRARY_PATH=/lib64
    2CIENVVAR      JVM_TYPE=openj9
    2CIENVVAR      SHLVL=2
    2CIENVVAR      JAVA_GLOBAL_OPTS= -Xdump:heap:events=systhrow+user,filter=java/lang/OutOfMemoryError,request=exclusive+prepwalk+compact,label=/home/zenap/dump/dump-2018-12-28-21-30-22.phd -Xdump:none -Xdump:system:events=gpf+abort+traceassert,range=1..0,priority=999,request=serial,label=/home/zenap/dump/core-dump-2018-12-28-21-30-22.%seq.dmp -Xdump:heap:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=500,request=exclusive+compact+prepwalk,label=/home/zenap/dump/dump-dump-2018-12-28-21-30-22.%seq.phd -Xdump:heap:events=user,priority=500,request=exclusive+compact+prepwalk,label=/home/zenap/dump/dump-dump-user-2018-12-28-21-30-22.%seq.phd -Xdump:java:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=400,request=exclusive+preempt,label=/home/zenap/dump/javacore-dump-2018-12-28-21-30-22.%seq.txt -Xdump:java:events=gpf+abort+traceassert+user,priority=400,request=exclusive+preempt,label=/home/zenap/dump/javacore-dump-2018-12-28-21-30-22.%seq.txt -Xdump:snap:events=systhrow,filter=java/lang/OutOfMemoryError,range=1..1,priority=300,request=serial,label=/home/zenap/dump/snap-dump-2018-12-28-21-30-22.%seq.trc -Xdump:snap:events=gpf+abort+traceassert,priority=300,request=serial,label=/home/zenap/dump/snap-dump-2018-12-28-21-30-22.%seq.trc -Xverbosegclog:/home/zenap/gclog/gc-2018-12-28-21-30-22.log,10,20000 -Xquickstart  -Dfile.encoding=UTF-8 -Xlp:objectheap:pagesize=4K  -Xlp:codecache:pagesize=4K -XX:+UseContainerSupport 
    2CIENVVAR      HOME=/root
    2CIENVVAR      dwApp_msbClientConfig_msbSvrIp=193.168.0.4
    2CIENVVAR      dwApp_esclientconfig_username=c5ELmd
    2CIENVVAR      openpalette_container_name=rm-service
    2CIENVVAR      dwApp_kafkaClientConf_autoDiscover=false
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_PASSWORD=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      JVM_GC_OPTS= -Xgcpolicy:gencon -Xgcthreads3 
    2CIENVVAR      dwApp_elasticsearchBackupConfig_user=
    2CIENVVAR      dwApp_msbClientConfig_msbSvrPort=10081
    2CIENVVAR      OPENPALETTE_LOGSTASH_OEM_TCP_PORT=4522
    2CIENVVAR      dwApp_cacheConfig_cacheServiceName=postgreCacheService
    2CIENVVAR      OPENPALETTE_LOGSTASH_INDEXER_HTTP_PORT=9601
    2CIENVVAR      OPENPALETTE_PGCACHE_PASSWORD=FA211C176CFA69AC0F87F6FACE530ADAEB6393236565BC7B83FCC77D2AD9FD12
    2CIENVVAR      OPENPALETTE_REDIS_PASSWORD=7336FB784AB98B17581BC1C5572C53FA
    2CIENVVAR      MIN_DUMP_SIZE=921m
    2CIENVVAR      OPENPALETTE_PG_DBNAME=db_24e3c661c5c7470ab286fab096b8a871
    2CIENVVAR      OPENPALETTE_LOGSTASH_COS_TCP_PORT=4525
    2CIENVVAR      OPENPALETTE_MSB_IP=193.168.0.4
    2CIENVVAR      redis_host=single-redis-78167794-7fd9-4111-b9d9-8a0920d85d64-opcs
    2CIENVVAR      MAX_DUMP_SIZE=3686m
    2CIENVVAR      dwApp_elasticsearchBackupConfig_password=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_ADDRESS=commsrves-feacf1f4-af4a-4943-be75-1de7e2be3f5e-opcs
    2CIENVVAR      dwApp_cacheConfig_cacheServiceVersion=v1
    2CIENVVAR      dwApp_cacheConfig_userName=NQ2q1wihoSDTYj
    2CIENVVAR      OPENPALETTE_LOGSTASH_UMF_TCP_PORT=4523
    2CIENVVAR      dwApp_database_user=UW5KhXPBIaDMpO
    2CIENVVAR      dwApp_esclientconfig_servers_node1=[commsrves-feacf1f4-af4a-4943-be75-1de7e2be3f5e-opcs]:9300
    2CIENVVAR      OPENPALETTE_LOGSTASH_UMF_UDP_PORT=4524
    2CIENVVAR      KUBERNETES_PORT_443_TCP_ADDR=10.254.0.1
    2CIENVVAR      OPENPALETTE_PGCACHE_ADDRESS=pgcache-edf5bfe5-3ad8-4662-b69a-9aa2938a1e89-opcs
    2CIENVVAR      HTTP_PORT=13601
    2CIENVVAR      PATH=/openj9jdk/bin:/openj9jdk/bin:/openjdk/bin:/openj9jdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    2CIENVVAR      OPENPALETTE_REDIS_ADDRESS=single-redis-78167794-7fd9-4111-b9d9-8a0920d85d64-opcs
    2CIENVVAR      redis_port=6379
    2CIENVVAR      OPENPALETTE_PGCACHE_PORT=36789
    2CIENVVAR      KUBERNETES_PORT_443_TCP_PORT=443
    2CIENVVAR      MEM_LIMIT=4096
    2CIENVVAR      dwApp_kafkaClientConf_zkServers=cskafka-273de3a3-3772-4e83-990e-ad6f52b452e8-zk-opcs:2181
    2CIENVVAR      redis_password=7336FB784AB98B17581BC1C5572C53FA
    2CIENVVAR      OPENPALETTE_LOGSTASH_INDEXER_ADDRESS=logstash-9e0887fd-cdd8-4797-920b-64d92acbde92-opcs
    2CIENVVAR      OPENPALETTE_LOGSTASH_SHIPPER_HTTP_PORT=9600
    2CIENVVAR      OPENPALETTE_PG_PASSWORD=221A10B1855A34F66C966781A4A7AC10264138FF5F3D5057DB861AA76CDB85C4
    2CIENVVAR      OPENPALETTE_REDIS_PORT=6379
    2CIENVVAR      KUBERNETES_PORT_443_TCP_PROTO=tcp
    2CIENVVAR      dwApp_elasticsearchConfig_tcpPort=9300
    2CIENVVAR      logstash_ip=logstash-9e0887fd-cdd8-4797-920b-64d92acbde92-opcs
    2CIENVVAR      openpalette_ms_bpname=rm-service
    2CIENVVAR      LANG=C.UTF-8
    2CIENVVAR      dwApp_cacheConfig_type=postgresql
    2CIENVVAR      dwApp_database_password=221A10B1855A34F66C966781A4A7AC10264138FF5F3D5057DB861AA76CDB85C4
    2CIENVVAR      dwApp_esclientconfig_clusterName=elasticsearch5
    2CIENVVAR      OPENPALETTE_LOGSTASH_INDEXER_PORT=4521
    2CIENVVAR      logstash_port=4522
    2CIENVVAR      dwApp_esclientconfig_password=31DC5E11EEFB028A798D97BD7C67BB0B
    2CIENVVAR      dwApp_cacheConfig_autoDiscover=false
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_USERNAME=c5ELmd
    2CIENVVAR      dwApp_cacheConfig_password=FA211C176CFA69AC0F87F6FACE530ADAEB6393236565BC7B83FCC77D2AD9FD12
    2CIENVVAR      dwApp_elasticsearchConfig_userName=c5ELmd
    2CIENVVAR      OPENPALETTE_MSB_PORT=10081
    2CIENVVAR      rdb_ip=commsrvpg-a933661b-efe4-446a-9d19-9708c11bd1cb-opcs
    2CIENVVAR      dwApp_esclientconfig_security=1
    2CIENVVAR      OPENPALETTE_PG_ADDRESS=commsrvpg-a933661b-efe4-446a-9d19-9708c11bd1cb-opcs
    2CIENVVAR      openpalette_ms_uuid=b5dcb440-fffc-43db-8db3-2f29c371b9a9
    2CIENVVAR      KUBERNETES_SERVICE_PORT_HTTPS=443
    2CIENVVAR      KUBERNETES_PORT_443_TCP=tcp://10.254.0.1:443
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_TCP_PORT=9300
    2CIENVVAR      OPENPALETTE_PGCACHE_USERNAME=NQ2q1wihoSDTYj
    2CIENVVAR      dwApp_cacheConfig_cacheServerIp=pgcache-edf5bfe5-3ad8-4662-b69a-9aa2938a1e89-opcs
    2CIENVVAR      dwApp_cacheConfig_asNamespace=db_4aa5664477b347518ae829590d0a29d3
    2CIENVVAR      rdb_port=36789
    2CIENVVAR      OPENPALETTE_PG_PORT=36789
    2CIENVVAR      KUBERNETES_SERVICE_HOST=10.254.0.1
    2CIENVVAR      JAVA_HOME=/openj9jdk
    2CIENVVAR      PWD=/home/zenap
    2CIENVVAR      DBCONF=ip=194.167.0.3:3000
    2CIENVVAR      DEBUG_PORT=8777
    2CIENVVAR      dwApp_cacheConfig_cacheServerPort=36789
    2CIENVVAR      OPENPALETTE_LOGSTASH_SHIPPER_ADDRESS=logstash-9e0887fd-cdd8-4797-920b-64d92acbde92-opcs
    2CIENVVAR      OPENPALETTE_KAFKA_ADDRESS=cskafka-273de3a3-3772-4e83-990e-ad6f52b452e8-opcs
    2CIENVVAR      dwApp_elasticsearchConfig_securityReinforce=1
    2CIENVVAR      TZ=Asia/Shanghai
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_SECURITY_REINFORCE=1
    2CIENVVAR      OPENPALETTE_LOGSTASH_SHIPPER_PORT=4522
    2CIENVVAR      openpalette_srv_uuid=c6341ca5-9414-45f9-a7c1-20c1710326e9
    2CIENVVAR      OPENPALETTE_KAFKA_PORT=9092
    2CIENVVAR      OPENPALETTE_NAMESPACE=ranoss
    2CIENVVAR      OPENPALETTE_ELASTICSEARCH_CLUSTER_NAME=elasticsearch5
    2CIENVVAR      dwApp_elasticsearchConfig_ip=commsrves-feacf1f4-af4a-4943-be75-1de7e2be3f5e-opcs
    2CIENVVAR      rdb_dbname=db_24e3c661c5c7470ab286fab096b8a871
    
CPU信息（1CICPUINFO）部分，也是非常有用的
* Physical CPUs：物理CPU核数
* Online CPUs：？
* Bound CPUs：从容器环境看，这就是容器拥有的CPU核数
* Target CPUs：？


    1CICPUINFO     CPU Information
    NULL           ------------------------------------------------------------------------
    2CIPHYSCPU     Physical CPUs: 8
    2CIONLNCPU     Online CPUs: 8
    2CIBOUNDCPU    Bound CPUs: 1
    2CITARGETCPU   Target CPUs: 1
    
    
### NATIVEMEMINFO

NATIVEMEMINFO部分提供了所有通过库函数例如malloc()函数和mmap()函数分配的本机内存大小，
内存大小按照组件做了细分，每个内存类别都包含该类别和所有子类别的内存大小综合。
在下面的例子中，4682840字节，141个分配单元？的本机内存分配给了虚拟机上的类


    NULL           ------------------------------------------------------------------------
    0SECTION       NATIVEMEMINFO subcomponent dump routine
    NULL           =================================
    0MEMUSER
    1MEMUSER       JRE: 2,569,088,312 bytes / 4653 allocations
    1MEMUSER       |
    2MEMUSER       +--VM: 2,280,088,336 bytes / 2423 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Classes: 4,682,840 bytes / 141 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Memory Manager (GC): 2,054,966,784 bytes / 433 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Java Heap: 2,014,113,792 bytes / 1 allocation
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Other: 40,852,992 bytes / 432 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Threads: 10,970,016 bytes / 156 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Java Stack: 197,760 bytes / 16 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Native Stack: 10,616,832 bytes / 17 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Other: 155,424 bytes / 123 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Trace: 180,056 bytes / 263 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--JVMTI: 17,776 bytes / 13 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--JNI: 36,184 bytes / 52 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Port Library: 208,179,632 bytes / 72 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Unused <32bit allocation regions: 208,168,752 bytes / 1 allocation
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Other: 10,880 bytes / 71 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Other: 1,055,048 bytes / 1293 allocations
    1MEMUSER       |
    2MEMUSER       +--JIT: 288,472,816 bytes / 140 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--JIT Code Cache: 268,435,456 bytes / 1 allocation
    2MEMUSER       |  |
    3MEMUSER       |  +--JIT Data Cache: 2,097,216 bytes / 1 allocation
    2MEMUSER       |  |
    3MEMUSER       |  +--Other: 17,940,144 bytes / 138 allocations
    1MEMUSER       |
    2MEMUSER       +--Class Libraries: 13,432 bytes / 25 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--VM Class Libraries: 13,432 bytes / 25 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--sun.misc.Unsafe: 3,184 bytes / 13 allocations
    4MEMUSER       |  |  |  |
    5MEMUSER       |  |  |  +--Direct Byte Buffers: 1,056 bytes / 12 allocations
    4MEMUSER       |  |  |  |
    5MEMUSER       |  |  |  +--Other: 2,128 bytes / 1 allocation
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Other: 10,248 bytes / 12 allocations
    1MEMUSER       |
    2MEMUSER       +--Unknown: 513,728 bytes / 2065 allocations
    NULL           


可以在VM Class libraries部分找到 Direct Byte Buffers的内存类别

如果设置Dcom.ibm.dbgmalloc=true之后，会在Class Libraries 部分中记录明细的内存分类信息

这部分不会记录应用分配的或者JNI code分配的内存

这部分统计的内存总量始终略低于通过操作系统工具统计的本机地址空间总使用量ß



### THREADS

* 3XMTHREADINFO：线程名称，虚拟机线程结构的地址信息，Java线程对象，线程状态和线程优先级
* 3XMTHREADINFO1：本地操作系统的线程ID，优先级，调度策略，虚拟机线程状态，虚拟机线程标记
* 3XMTHREADINFO2：本地栈的地址范围
* 3XMJAVALTHREAD：Java线程ID和daemon状态
* 5XESTACKTRACE：调用堆栈，这部分可以表明是否有方法锁住了当前的线程

关于Java的Daemon的解释：参考[这里](https://www.cnblogs.com/ChrisWang/archive/2009/11/28/1612815.html)


Thread state value	Status	Description
R	Runnable	The thread is able to run
CW	Condition Wait	The thread is waiting
S	Suspended	The thread is suspended by another thread
Z	Zombie	The thread is destroyed
P	Parked	The thread is parked by java.util.concurrent
B	Blocked	The thread is waiting to obtain a lock



