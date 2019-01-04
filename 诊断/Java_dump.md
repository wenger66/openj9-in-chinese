# Java转储

wenger66开头部分为作者经验，建议关注

## 目录  
- [转储格式](#转储格式) 
  - [文件格式](#文件格式) 
  - [段落格式](#段落格式) 
  - [行格式](#行格式) 
- [转储内容](#转储内容)  
  - [TITLE](#TITLE) 
  - [GPINFO](#GPINFO) 
  - [ENVINFO](#ENVINFO)  
  - [NATIVEMEMINFO](#NATIVEMEMINFO)
  - [MEMINFO](#MEMINFO)    
  - [LOCKS](#LOCKS) 
  - [THREADS](#THREADS)   
  - [HOOK](#HOOK)  
  - [SHARED CLASSES](#SHARED_CLASSES)  
  - [CLASSES](#CLASSES)  
- [实战](#实战) 
  - [GPF错误](#GPF错误) 
  - [OOM错误](#OOM错误) 
  - [本地OOM错误](#本地OOM错误)  
  - [死锁](#死锁)
  - [挂死](#挂死)  

## 转储格式
### 文件格式
Javadump 通常是文本格式(.txt)，缺省文件名为 javacore.\<date>.\<time>.\<pid>.\<sequence number>.txt，因此可以通过一般的文本编辑器进行阅读，阅读时需要注意段与行的格式
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

等等

## 转储内容
Java转储文件汇总了事件发生时虚拟机的状态，包括虚拟机组件的大部分信息。转储文件由多个部分组成，
每个部分提供了不同的信息

感觉段落用英文更加专业，无歧义
### TITLE 
TITLE部分提供了转储时的事件信息。下面的例子中，你可以看到是vmstop事件在2018/08/30 21:55:47这个时间
触发了这次转储

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
GPINFO部分提供了运行虚拟机的操作系统信息。下面的例子，说明虚拟机是运行在Linux系统上的

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
    
GPINFO部分包含的内容可能因为转储原因不同而多种多样。例如如果是因为gpf事件触发的dump，
会用**VM Flags**记录导致崩溃的库，通过VM Flags这个值
可以追溯到具体虚拟机组件。比如下面这段

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

0000000000000000 (0x00000) 表示崩溃是虚拟机外部造成的

### ENVINFO
ENVINFO部分提供了应用程序运行环境的很多有用信息，例如
* Java版本（**1CIJAVAVERSION**）
* OpenJ9虚拟机版本及其组件版本（**1CIVMVERSION**, **1CIJ9VMVERSION**, 
**1CIJITVERSION**, **1CIOMRVERSION**, **1CIJCLVERSION**）
* 虚拟机启动时间（**1CISTARTTIME**）
* 进程ID（**1CIPROCESSID**）
* Jave home文件夹（**1CIJAVAHOMEDIR**）
* Java dll文件夹（**1CIJAVADLLDIR**）
* 用户设置的Java参数（**1CIUSERARG**）
* 用户进程受到的资源限制信息（**1CIUSERLIMITS**）
* 环境变量（**1CIENVVARS**）
* 系统信息（**1CISYSINFO**）
* CPU信息（**1CICPUINFO**）

==============================        

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


资源限制(**1CIUSERLIMITS**)部分，除了NOFILE和NPROC类，其他都是字节为单位。(好像不对，例如RLIMIT_CPU)。
在Linux系统中，Resouce Limit指在一个进程的执行过程中，它受到的资源的限制
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

==================================================


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
    

wenger66:`环境变量(**1CIENVVARS**)部分，会打印出所有的环境变量，对于容器内的应用程序非常有用`

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
* Online CPUs：不明
* Bound CPUs：从容器环境看，这就是容器拥有的CPU核数
* Target CPUs：不明

=====================================

    1CICPUINFO     CPU Information
    NULL           ------------------------------------------------------------------------
    2CIPHYSCPU     Physical CPUs: 8
    2CIONLNCPU     Online CPUs: 8
    2CIBOUNDCPU    Bound CPUs: 1
    2CITARGETCPU   Target CPUs: 1
    
    
### NATIVEMEMINFO

NATIVEMEMINFO部分提供了所有通过库函数例如malloc()函数和mmap()函数分配的本机内存大小，
内存大小按照组件做了细分，每个内存类别都包含该类别和所有子类别的内存大小总和。
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


可以在VM Class libraries部分找到堆外内存(Direct Byte Buffers)的内存类别

如果设置Dcom.ibm.dbgmalloc=true之后，会在Class Libraries 部分中记录明细的内存分类信息

这部分不会记录应用分配的或者 JNI code 分配的内存

这部分统计的内存总量始终略低于通过操作系统工具统计的本机地址空间总使用量

### MEMINFO
MEMINFO部分与内存管理有关，提供了虚拟机对内存使用的分类明细的10进制和16进制格式，包括对象堆，内部内存，类使用的内存，
JIT代码缓存和JIT数据缓存。你可以根据这部分判断出当前使用的垃圾回收策略

堆内存部分(**1STHEAPTYPE**)记录对象内存使用的堆区域信息，包括堆区域的开始、结束的地址，以及堆区域的大小
段内存部分(**1STSEGMENT**)记录了内部内存，类占用内存，JIT代码高速缓存和JIT数据高速缓存的分段信息，
包括控制分段的数据结构的地址，分段开始和分段结束的地址，以及分段大小

堆内存部分 (**HEAPTYPE**)：
* id - 空间或区域的标识
* start - 堆区域的启动地址
* end - 堆区域的结束地址
* size - 堆区域的大小(单位：字节)
* space/region - 对于仅包含id和名称的行，该列显示内存空间的名称。否则，该列显示内存空间名称，
后跟该内存空间中包含的特定区域的名称。

段内存部分 (**SEGTYPE**)：
* segment - 段控制数据结构的地址
* start - 本机内存段的开始地址
* alloc - 本机内存段的当前分配地址
* end - 本机内存段的结束地址
* type - 内部位字段，用于描述本机内存段的特征
* size - 本机内存段的大小(单位：字节)

关于堆/段/内部内存/类占用内存的解释：参考[这里](https://github.com/wenger66/openj9-in-chinese/blob/master/垃圾回收/Memory_Manager.md)

为了清晰，下面的例子略去了部分内容，用...表示

    NULL           ------------------------------------------------------------------------
    0SECTION       MEMINFO subcomponent dump routine
    NULL           =================================
    NULL           
    1STHEAPTYPE    Object Memory
    NULL           id                 start              end                size               space/region
    1STHEAPSPACE   0x00007FF4F00744A0         --                 --                 --         Generational
    1STHEAPREGION  0x00007FF4F0074CE0 0x0000000087F40000 0x0000000088540000 0x0000000000600000 Generational/Tenured Region
    1STHEAPREGION  0x00007FF4F0074930 0x00000000FFE00000 0x00000000FFF00000 0x0000000000100000 Generational/Nursery Region
    1STHEAPREGION  0x00007FF4F0074580 0x00000000FFF00000 0x0000000100000000 0x0000000000100000 Generational/Nursery Region
    NULL
    1STHEAPTOTAL   Total memory:                     8388608 (0x0000000000800000)
    1STHEAPINUSE   Total memory in use:              2030408 (0x00000000001EFB48)
    1STHEAPFREE    Total memory free:                6358200 (0x00000000006104B8)
    NULL
    1STSEGTYPE     Internal Memory
    NULL           segment            start              alloc              end                type       size
    1STSEGMENT     0x00007FF4F004CBC8 0x00007FF4CD33C000 0x00007FF4CD33C000 0x00007FF4CE33C000 0x01000440 0x0000000001000000
    1STSEGMENT     0x00007FF4F004CB08 0x00007FF4DE43D030 0x00007FF4DE517770 0x00007FF4DE53D030 0x00800040 0x0000000000100000
    NULL
    1STSEGTOTAL    Total memory:                    17825792 (0x0000000001100000)
    1STSEGINUSE    Total memory in use:               894784 (0x00000000000DA740)
    1STSEGFREE     Total memory free:               16931008 (0x00000000010258C0)
    NULL           
    1STSEGTYPE     Class Memory
    NULL           segment            start              alloc              end                type       size
    1STSEGMENT     0x00007FF4F03B5638 0x0000000001053D98 0x000000000105BD98 0x000000000105BD98 0x00010040 0x0000000000008000
    1STSEGMENT     0x00007FF4F03B5578 0x0000000001048188 0x0000000001050188 0x0000000001050188 0x00010040 0x0000000000008000
    ...
    NULL
    1STSEGTOTAL    Total memory:                     3512520 (0x00000000003598C8)
    1STSEGINUSE    Total memory in use:              3433944 (0x00000000003465D8)
    1STSEGFREE     Total memory free:                  78576 (0x00000000000132F0)
    NULL           
    1STSEGTYPE     JIT Code Cache
    NULL           segment            start              alloc              end                type       size
    1STSEGMENT     0x00007FF4F00961F8 0x00007FF4CE43D000 0x00007FF4CE445790 0x00007FF4DE43D000 0x00000068 0x0000000010000000
    NULL
    1STSEGTOTAL    Total memory:                   268435456 (0x0000000010000000)
    1STSEGINUSE    Total memory in use:                34704 (0x0000000000008790)
    1STSEGFREE     Total memory free:              268400752 (0x000000000FFF7870)
    1STSEGLIMIT    Allocation limit:               268435456 (0x0000000010000000)
    NULL           
    1STSEGTYPE     JIT Data Cache
    NULL           segment            start              alloc              end                type       size
    1STSEGMENT     0x00007FF4F0096668 0x00007FF4CC553030 0x00007FF4CC753030 0x00007FF4CC753030 0x00000048 0x0000000000200000
    NULL
    1STSEGTOTAL    Total memory:                     2097152 (0x0000000000200000)
    1STSEGINUSE    Total memory in use:              2097152 (0x0000000000200000)
    1STSEGFREE     Total memory free:                      0 (0x0000000000000000)
    1STSEGLIMIT    Allocation limit:               402653184 (0x0000000018000000)
    NULL           
    1STGCHTYPE     GC History  
    NULL 
    
上个例子中，GC History部分是空的。只要虚拟机进行过一次GC，这部分就会丰富起来，比如下面的例子

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

### LOCKS
LOCKS部分提供了锁的信息，锁是用来保护同一时间被多个实体访问的共享对象。LOCKS这部分信息在应用程序
出现死锁时尤为关键，死锁是指两个或多个线程互相持有对方需要的锁。LOCKS部分中有导致死锁的线程的
详细信息，可以帮助你定位死锁的根源。

在进行Java转储时，JVM会尝试检测死锁循环：
JVM 可以检测由通过同步获取的锁和/或扩展了 java.util.concurrent.locks.AbstractOwnableSynchronizer 类的锁组成的死锁循环

Java 语言中的每个对象都有关联的锁定，也称为监控器（Monitor），所以LOCKS部分中大量出现monitor字样。

**3LKWAITER**这个关键字值得注意，会搜索到一些需要你探究的信息

下面的例子展示了典型的没有出现死锁的LOCKS部分信息。为了更加清晰，
下面的例子略去了部分内容，用...表示

    NULL           ------------------------------------------------------------------------
    0SECTION       LOCKS subcomponent dump routine
    NULL           ===============================
    NULL           
    1LKPOOLINFO    Monitor pool info:
    2LKPOOLTOTAL     Current total number of monitors: 3
    NULL           
    1LKMONPOOLDUMP Monitor Pool Dump (flat & inflated object-monitors):
    2LKMONINUSE      sys_mon_t:0x00007FF4B0001D78 infl_mon_t: 0x00007FF4B0001DF8:
    3LKMONOBJECT       java/lang/ref/ReferenceQueue@0x00000000FFE26A10: <unowned>
    3LKNOTIFYQ            Waiting to be notified:
    3LKWAITNOTIFY            "Common-Cleaner" (J9VMThread:0x0000000000FD0100)
    NULL           
    1LKREGMONDUMP  JVM System Monitor Dump (registered monitors):
    2LKREGMON          Thread global lock (0x00007FF4F0004FE8): <unowned>
    2LKREGMON          &(PPG_mem_mem32_subAllocHeapMem32.monitor) lock (0x00007FF4F0005098): <unowned>
    2LKREGMON          NLS hash table lock (0x00007FF4F0005148): <unowned>
    ...
    NULL           
    
    
下面的例子来自死锁测试程序。请看Deadlock detected !!!部分，Deadlock Thread 0 等待着
Deadlock Thread 1持有锁的String对象，而Deadlock Thead 1等待着Deadlock Thead 0持有的
ReentrantLock实例

    NULL           ------------------------------------------------------------------------
    0SECTION       LOCKS subcomponent dump routine
    NULL           ===============================
    NULL            
    1LKPOOLINFO    Monitor pool info:
    2LKPOOLTOTAL     Current total number of monitors: 2
    NULL            
    1LKMONPOOLDUMP Monitor Pool Dump (flat & inflated object-monitors):
    2LKMONINUSE      sys_mon_t:0x00007F5E24013F10 infl_mon_t: 0x00007F5E24013F88:
    3LKMONOBJECT      java/lang/String@0x00007F5E5E18E3D8: Flat locked by "Deadlock Thread 1" (0x00007F5E84362100), entry count 1
    3LKWAITERQ            Waiting to enter:
    3LKWAITER                "Deadlock Thread 0" (0x00007F5E8435BD00)
    NULL            
    1LKREGMONDUMP  JVM System Monitor Dump (registered monitors):
    2LKREGMON          Thread global lock (0x00007F5E84004F58): <unowned>
    2LKREGMON          &(PPG_mem_mem32_subAllocHeapMem32.monitor) lock (0x00007F5E84005000): <unowned>
    2LKREGMON          NLS hash table lock (0x00007F5E840050A8): <unowned>
                < lines removed for brevity >
    
    1LKDEADLOCK    Deadlock detected !!!
    NULL           ---------------------
    NULL            
    2LKDEADLOCKTHR  Thread "Deadlock Thread 0" (0x00007F5E8435BD00)
    3LKDEADLOCKWTR    is waiting for:
    4LKDEADLOCKMON      sys_mon_t:0x00007F5E24013F10 infl_mon_t: 0x00007F5E24013F88:
    4LKDEADLOCKOBJ      java/lang/String@0x00007F5E5E18E3D8
    3LKDEADLOCKOWN    which is owned by:
    2LKDEADLOCKTHR  Thread "Deadlock Thread 1" (0x00007F5E84362100)
    3LKDEADLOCKWTR    which is waiting for:
    4LKDEADLOCKOBJ      java/util/concurrent/locks/ReentrantLock$NonfairSync@0x00007F5E7E1464F0
    3LKDEADLOCKOWN    which is owned by:
    2LKDEADLOCKTHR  Thread "Deadlock Thread 0" (0x00007F5E8435BD00)


### THREADS

THREADS部分提供了虚拟机线程池的概要信息，Java线程、本地线程的详细信息，以及线程调用栈。
对于应用程序员而言，本部分是Javadump最有用、最常观察的部分之一，可以帮助你定位阻塞、等待的线程。

* **3XMTHREADINFO**：线程名称，虚拟机线程结构和Java线程对象的地址，Java线程状态和Java线程优先级
* **3XMJAVALTHREAD**：Java线程ID和daemon状态
* **3XMTHREADINFO1**：本地操作系统的线程ID，优先级，调度策略，虚拟机线程状态，虚拟机线程标记
* **3XMTHREADINFO2**：本地栈的地址范围
* **3XMTHREADINFO3**：Java调用栈信息或本地调用栈信息
* **5XESTACKTRACE**：调用堆栈，这部分可以表明是否有方法持有了某个锁

wenger66:`THREADS部分的Current Thread非常有用，可以显示自动转储时，当前的线程。经过测试，手工转储时，没有 Current thread 部分；只有自动转储时，才有 Current thread 部分`


关于Java的Daemon线程的解释：参考[这里](https://www.cnblogs.com/ChrisWang/archive/2009/11/28/1612815.html)

Java线程优先级会根据平台映射至操作系统优先级值。较大的Java线程优先级值表明该线程具有较高的优先级。换言之，该线程会比较低优先级的线程更频繁地运行。

Java线程状态和虚拟机线程状态的值可以是以下
* R - 可运行 - 条件满足时，该线程将运行。
* CW - 等待条件 - 该线程正在等待。例如，由于I/O已阻止了该线程调用了 wait() 方法，
以等待通知监视器该线程通过 join() 调用正在与另一个线程同步
* S – 已挂起 – 该线程已被另一个线程刮起
* Z – 僵死 – 该线程已被结束
* P – 已停放 – 该线程已因并发 API (java.util.concurrent) 而被停放。如线程池中的线程被使用后再次放回线程池，状态即为Parked
* B – 已阻塞 – 该线程正在等待获取其他对象当前拥有的锁

**3XMTHREADBLOCK** - 如果线程状态为已停放(P)、已阻塞(B)、正在等待条件(CW)，那么输出信息中会包含以**3XMTHREADBLOCK**开头的一行，紧跟着会有
Blocked on，Parked on，Waiting on，
会列出该线程正在等待的资源，以及当前拥有该资源的线程。要特别关注这3类线程

**3XMCPUTIME** - 对于 Java 线程和本机线程，输出信息中会包含以 **3XMCPUTIME** 开头的行。该行显示自启动线程
以来线程所耗用的 CPU 时间（以秒计），即该线程所使用的 CPU 总时间。
注意：如果从线程池中复用某个 Java 线程，那么不会重置此线程的 CPU 时间，而会继续累计。

**3XMHEAPALLOC** - 对于 Java 线程，输出信息中会包含以**3XMHEAPALLOC** 开头的行。
自上次垃圾回收以来该线程所分配的堆字节数。

**1XECTHTYPE** - 对于由异常 throw、catch、uncaught 和 systhrow 事件
或由 com.ibm.jvm.Dump API 触发的 Javadump，会在 THREADS 部分的结尾处输出包含**1XECTHTYPE**的行。
这些行显示dump时的触发点。

为了更加清晰，下面的例略去了部分内容，使用...表示


    NULL           ------------------------------------------------------------------------
    0SECTION       THREADS subcomponent dump routine
    NULL           =================================
    NULL
    1XMPOOLINFO    JVM Thread pool info:
    2XMPOOLTOTAL       Current total number of pooled threads: 18
    2XMPOOLLIVE        Current total number of live threads: 16
    2XMPOOLDAEMON      Current total number of live daemon threads: 15
    NULL           
    1XMTHDINFO     Thread Details
    NULL           
    3XMTHREADINFO      "JIT Diagnostic Compilation Thread-7 Suspended" J9VMThread:0x0000000000EFC500, omrthread_t:0x00007FF4F00A77E8, java/lang/Thread:0x00000000FFE97480, state:R, prio=10
    3XMJAVALTHREAD            (java/lang/Thread getId:0xA, isDaemon:true)
    3XMTHREADINFO1            (native thread ID:0x7657, native priority:0xB, native policy:UNKNOWN, vmstate:CW, vm thread flags:0x00000081)
    3XMTHREADINFO2            (native stack address range from:0x00007FF4CCC36000, to:0x00007FF4CCD36000, size:0x100000)
    3XMCPUTIME               CPU usage total: 0.000037663 secs, current category="JIT"
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           No Java callstack associated with this thread
    3XMTHREADINFO3           No native callstack available for this thread
    NULL
    ...
    3XMTHREADINFO      "Common-Cleaner" J9VMThread:0x0000000000FD0100, omrthread_t:0x00007FF4F022A520, java/lang/Thread:0x00000000FFE26F40, state:CW, prio=8
    3XMJAVALTHREAD            (java/lang/Thread getId:0x2, isDaemon:true)
    3XMTHREADINFO1            (native thread ID:0x765A, native priority:0x8, native policy:UNKNOWN, vmstate:CW, vm thread flags:0x00080181)
    3XMTHREADINFO2            (native stack address range from:0x00007FF4CC0B8000, to:0x00007FF4CC0F8000, size:0x40000)
    3XMCPUTIME               CPU usage total: 0.000150926 secs, current category="Application"
    3XMTHREADBLOCK     Waiting on: java/lang/ref/ReferenceQueue@0x00000000FFE26A10 Owned by: <unowned>
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at java/lang/Object.wait(Native Method)
    4XESTACKTRACE                at java/lang/Object.wait(Object.java:221)
    4XESTACKTRACE                at java/lang/ref/ReferenceQueue.remove(ReferenceQueue.java:138)
    5XESTACKTRACE                   (entered lock: java/lang/ref/ReferenceQueue@0x00000000FFE26A10, entry count: 1)
    4XESTACKTRACE                at jdk/internal/ref/CleanerImpl.run(CleanerImpl.java:148)
    4XESTACKTRACE                at java/lang/Thread.run(Thread.java:835)
    4XESTACKTRACE                at jdk/internal/misc/InnocuousThread.run(InnocuousThread.java:122)
    3XMTHREADINFO3           No native callstack available for this thread
    NULL
    NULL
    3XMTHREADINFO      "IProfiler" J9VMThread:0x0000000000F03D00, omrthread_t:0x00007FF4F00B06F8, java/lang/Thread:0x00000000FFE97B60, state:R, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0xC, isDaemon:true)
    3XMTHREADINFO1            (native thread ID:0x7659, native priority:0x5, native policy:UNKNOWN, vmstate:CW, vm thread flags:0x00000081)
    3XMTHREADINFO2            (native stack address range from:0x00007FF4F8940000, to:0x00007FF4F8960000, size:0x20000)
    3XMCPUTIME               CPU usage total: 0.004753103 secs, current category="JIT"
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           No Java callstack associated with this thread
    3XMTHREADINFO3           No native callstack available for this thread
    NULL
    ...
    1XMWLKTHDERR   The following was reported while collecting native stacks:
    2XMWLKTHDERR             unable to count threads(3, -2)
    NULL
    1XMTHDSUMMARY  Threads CPU Usage Summary
    NULL           =========================
    NULL
    1XMTHDCATINFO  Warning: to get more accurate CPU times for the GC, the option -XX:-ReduceCPUMonitorOverhead can be used. See the user guide for more information.
    NULL
    1XMTHDCATEGORY All JVM attached threads: 0.280083000 secs
    1XMTHDCATEGORY |
    2XMTHDCATEGORY +--System-JVM: 0.270814000 secs
    2XMTHDCATEGORY |  |
    3XMTHDCATEGORY |  +--GC: 0.000599000 secs
    2XMTHDCATEGORY |  |
    3XMTHDCATEGORY |  +--JIT: 0.071904000 secs
    1XMTHDCATEGORY |
    2XMTHDCATEGORY +--Application: 0.009269000 secs
    NULL
    
    
在发生Java转储时，所有运行Java代码的线程状态可能是可运行（R）状态或者正在等待条件（CW）状态

wenger66:
`要特别关注已停放(P)、已阻塞(B)、正在等待条件(CW)这三类状态的线程
要了解哪些资源拥有处于停放、等待或阻塞状态的线程，
可以查找以 **3XMTHREADBLOCK** 开始的行。 该行还可能指示哪个线程拥有该资源。`


#### Owned by: <unknown>
可以扩展并使用 AbstractOwnableSynchronizer 类以在 Javadump中提供信息，以便帮助诊断锁的问题。
否则很可能会显示为unknown

     
#### 阻塞线程

以下是 Javadump 的 THREADS 部分中的输出样例，
显示处于阻塞状态的线程 Thread-5，该线程状态是已阻塞（B）。 
该线程等待资源 java/lang/String@0x4D8C90F8，这个资源目前由线程 main 拥有

    3XMTHREADINFO      "Thread-5" J9VMThread:0x4F6E4100, j9thread_t:0x501C0A28, java/lang/Thread:0x4D8C9520, state:B, prio=5
    3XMTHREADINFO1            (native thread ID:0x664, native priority:0x5, native policy:UNKNOWN)
    3XMTHREADBLOCK     Blocked on: java/lang/String@0x4D8C90F8 Owned by: "main" (J9VMThread:0x00129100, java/lang/Thread:0x00DD4798)
    
Javadump 的 LOCKS 部分显示有关该块的以下对应输出：

    1LKMONPOOLDUMP Monitor Pool Dump (flat & inflated object-monitors):
    2LKMONINUSE      sys_mon_t:0x501C18A8 infl_mon_t: 0x501C18E4:
    3LKMONOBJECT       java/lang/String@0x4D8C90F8: Flat locked by "main" (0x00129100), entry count 1
    3LKWAITERQ            Waiting to enter:
    3LKWAITER                "Thread-5" (0x4F6E4100)
    
你可以在 Javadump 的 THREADS 部分中的其他内容中查找有关阻塞线程 main 信息，
以了解 main 线程正在执行的操作

#### 等待线程
以下是 Javadump 的 THREADS 部分中的输出样例，
显示处于等待状态线程 Thread-5，该线程状态是正在等待（CW）。 
该线程等待获得有关资源 java/lang/String@0x4D8C90F8 的通知，这个资源目前由线程 main 拥有


    3XMTHREADINFO      "Thread-5" J9VMThread:0x00503D00, j9thread_t:0x00AE45C8, java/lang/Thread:0x68E04F90, state:CW, prio=5
    3XMTHREADINFO1            (native thread ID:0xC0C, native priority:0x5, native policy:UNKNOWN)
    3XMTHREADBLOCK     Waiting on: java/lang/String@0x68E63E60 Owned by: "main" (J9VMThread:0x6B3F9A00, java/lang/Thread:0x68E64178)

Javadump 的 LOCKS 部分显示有关正在等待的监视器的对应输出

    1LKMONPOOLDUMP Monitor Pool Dump (flat & inflated object-monitors):
    2LKMONINUSE      sys_mon_t:0x00A0ADB8 infl_mon_t: 0x00A0ADF4:
    3LKMONOBJECT       java/lang/String@0x68E63E60: owner "main" (0x6B3F9A00), entry count 1
    3LKNOTIFYQ            Waiting to be notified:
    3LKWAITNOTIFY            "Thread-5" (0x00503D00)
    

### HOOK
HOOK 部分提供了用于内部性能诊断的 JVM 内部事件回调的详细信息。此部分列出了多个回调接口，每个接口都包含了独立的
回调事件。

下面的例子中显示了J9VMHookInterface接口的数据，包括最近一次调用和耗时最长的调用的信息，包括：
调用代码位置和代码行，开始时间，调用耗时。

    NULL           ------------------------------------------------------------------------
    0SECTION       HOOK subcomponent dump routine
    NULL           ==============================
    1HKINTERFACE   MM_OMRHookInterface
    NULL           ------------------------------------------------------------------------
    1HKINTERFACE   MM_PrivateHookInterface
    NULL           ------------------------------------------------------------------------
    1HKINTERFACE   MM_HookInterface
    NULL           ------------------------------------------------------------------------
    1HKINTERFACE   J9VMHookInterface
    NULL           ------------------------------------------------------------------------
    2HKEVENTID     1
    3HKCALLCOUNT       18
    3HKLAST            Last Callback
    4HKCALLSITE           trcengine.c:392
    4HKSTARTTIME          Start Time: 2018-08-30T21:55:47.601
    4HKDURATION           DurationMs: 0
    3HKLONGST          Longest Callback
    4HKCALLSITE           trcengine.c:392
    4HKSTARTTIME          Start Time: 2018-08-30T21:55:47.460
    4HKDURATION           DurationMs: 1
    NULL
    ...
    1HKINTERFACE   J9VMZipCachePoolHookInterface
    NULL           ------------------------------------------------------------------------
    1HKINTERFACE   J9JITHookInterface
    NULL           ------------------------------------------------------------------------
    2HKEVENTID     3
    3HKCALLCOUNT       65
    3HKLAST            Last Callback
    4HKCALLSITE           ../common/mgmtinit.c:191
    4HKSTARTTIME          Start Time: 2018-08-30T21:55:47.601
    4HKDURATION           DurationMs: 0
    3HKLONGST          Longest Callback
    4HKCALLSITE           ../common/mgmtinit.c:191
    4HKSTARTTIME          Start Time: 2018-08-30T21:55:47.486
    4HKDURATION           DurationMs: 0
    ...
    NULL
    
### SHARED_CLASSES

OpenJ9 中的共享类特性可以用来减少内存占用并改进 JVM 启动时间，参考[这里](https://www.ibm.com/developerworks/cn/java/j-class-sharing-openj9/index.html)

如果共享类特性在运行期启用，SHARED CLASSES部分将提供创建共享类缓存的配置信息，共享类缓存的大小和内容的概要信息

下面的例子中，创建共享类缓存时的配置包括：
* 类调试区域(Class Debug Area) - -Xnolinenumbers建议虚拟机不要加载任何类的调试信息，-Xnolinenumbers = false说明加载了类调试信息
* 字节码工具(Byte code instrumentation) - 默认启用BCI，共享类特性支持与运行时字节码修改进行集成
* 保存类路径 - 默认启用保存类路径
    
    
    NULL
    1SCLTEXTCRTW   Cache Created With
    NULL           ------------------
    NULL
    2SCLTEXTXNL        -Xnolinenumbers       = false
    2SCLTEXTBCI        BCI Enabled           = true
    2SCLTEXTBCI        Restrict Classpaths   = false
    NULL


缓存概要信息包括了类缓存大小 (**2SCLTEXTCSZ**) 是16776608个字节, 
弹性？最大缓存大小 (**2SCLTEXTSMB**) 也是16776608个字节, 
空闲空间大小(**2SCLTEXTFRB**)是12691668个字节，
类调试区域(**2SCLTEXTDAS**)是1331200个字节，11%的空间已使用。

    NULL
    1SCLTEXTCSUM   Cache Summary
    NULL           ------------------
    NULL
    2SCLTEXTNLC        No line number content                    = false
    2SCLTEXTLNC        Line number content                       = true
    NULL
    2SCLTEXTRCS        ROMClass start address                    = 0x00007F423061C000
    2SCLTEXTRCE        ROMClass end address                      = 0x00007F42307B9A28
    2SCLTEXTMSA        Metadata start address                    = 0x00007F42313D42FC
    2SCLTEXTCEA        Cache end address                         = 0x00007F4231600000
    2SCLTEXTRTF        Runtime flags                             = 0x00102001ECA6028B
    2SCLTEXTCGN        Cache generation                          = 35
    NULL
    2SCLTEXTCSZ        Cache size                                = 16776608
    2SCLTEXTSMB        Softmx bytes                              = 16776608
    2SCLTEXTFRB        Free bytes                                = 12691668
    2SCLTEXTRCB        ROMClass bytes                            = 1694248
    2SCLTEXTAOB        AOT code bytes                            = 0
    2SCLTEXTADB        AOT data bytes                            = 0
    2SCLTEXTAHB        AOT class hierarchy bytes                 = 32
    2SCLTEXTATB        AOT thunk bytes                           = 0
    2SCLTEXTARB        Reserved space for AOT bytes              = -1
    2SCLTEXTAMB        Maximum space for AOT bytes               = -1
    2SCLTEXTJHB        JIT hint bytes                            = 308
    2SCLTEXTJPB        JIT profile bytes                         = 2296
    2SCLTEXTJRB        Reserved space for JIT data bytes         = -1
    2SCLTEXTJMB        Maximum space for JIT data bytes          = -1
    2SCLTEXTNOB        Java Object bytes                         = 0
    2SCLTEXTZCB        Zip cache bytes                           = 919328
    2SCLTEXTRWB        ReadWrite bytes                           = 114080
    2SCLTEXTJCB        JCL data bytes                            = 0
    2SCLTEXTBDA        Byte data bytes                           = 0
    2SCLTEXTMDA        Metadata bytes                            = 23448
    2SCLTEXTDAS        Class debug area size                     = 1331200
    2SCLTEXTDAU        Class debug area % used                   = 11%
    2SCLTEXTDAN        Class LineNumberTable bytes               = 156240
    2SCLTEXTDAV        Class LocalVariableTable bytes            = 0
    NULL
    2SCLTEXTNRC        Number ROMClasses                         = 595
    2SCLTEXTNAM        Number AOT Methods                        = 0
    2SCLTEXTNAD        Number AOT Data Entries                   = 0
    2SCLTEXTNAH        Number AOT Class Hierarchy                = 1
    2SCLTEXTNAT        Number AOT Thunks                         = 0
    2SCLTEXTNJH        Number JIT Hints                          = 14
    2SCLTEXTNJP        Number JIT Profiles                       = 20
    2SCLTEXTNCP        Number Classpaths                         = 1
    2SCLTEXTNUR        Number URLs                               = 0
    2SCLTEXTNTK        Number Tokens                             = 0
    2SCLTEXTNOJ        Number Java Objects                       = 0
    2SCLTEXTNZC        Number Zip Caches                         = 5
    2SCLTEXTNJC        Number JCL Entries                        = 0
    2SCLTEXTNST        Number Stale classes                      = 0
    2SCLTEXTPST        Percent Stale classes                     = 0%
    NULL
    2SCLTEXTCPF        Cache is 24% full
    NULL
    
    
**2SCLTEXTCMDT**开头的行显示了共享类缓存的名称和地址,CR表明缓存是64位压缩的引用缓存  

    NULL
    1SCLTEXTCMST   Cache Memory Status
    NULL           ------------------
    1SCLTEXTCNTD       Cache Name                    Feature                  Memory type              Cache path
    NULL
    2SCLTEXTCMDT       sharedcc_doc-javacore         CR                       Memory mapped file       /tmp/javasharedresources/C290M4F1A64P_sharedcc_doc-javacore_G35
    NULL
    1SCLTEXTCMST   Cache Lock Status
    NULL           ------------------
    1SCLTEXTCNTD       Lock Name                     Lock type                TID owning lock
    NULL
    2SCLTEXTCWRL       Cache write lock              File lock                Unowned
    2SCLTEXTCRWL       Cache read/write lock         File lock                Unowned
    NULL

### CLASSES
CLASSES部分提供了ClassLoader的信息。第一部分提供了每个ClassLoader的概要信息(**2CLTEXTCLLOADER**)，包括
加载的库数量和类数量。接下来有更多的关于加载库的详细信息(**1CLTEXTCLLIB**)和加载类的详细信息(**1CLTEXTCLLO**)

下面的例子中，你可以看见java/lang/InternalAnonymousClassLoader加载了2个类，分别是
jdk/internal/loader/BuiltinClassLoader$$Lambda$2/00000000F03876A0(0x0000000001030F00)
和jdk/internal/loader/BuiltinClassLoader$$Lambda$1/00000000F00D2460(0x0000000001018A00)

    NULL           ------------------------------------------------------------------------
    0SECTION       CLASSES subcomponent dump routine
    NULL           =================================
    1CLTEXTCLLOS    Classloader summaries
    1CLTEXTCLLSS        12345678: 1=primordial,2=extension,3=shareable,4=middleware,5=system,6=trusted,7=application,8=delegating
    2CLTEXTCLLOADER     p---st-- Loader *System*(0x00000000FFE1D258)
    3CLNMBRLOADEDLIB        Number of loaded libraries 5
    3CLNMBRLOADEDCL         Number of loaded classes 638
    2CLTEXTCLLOADER     -x--st-- Loader jdk/internal/loader/ClassLoaders$PlatformClassLoader(0x00000000FFE1D4F0), Parent *none*(0x0000000000000000)
    3CLNMBRLOADEDLIB        Number of loaded libraries 0
    3CLNMBRLOADEDCL         Number of loaded classes 0
    2CLTEXTCLLOADER     ----st-- Loader java/lang/InternalAnonymousClassLoader(0x00000000FFE1DFD0), Parent *none*(0x0000000000000000)
    3CLNMBRLOADEDLIB        Number of loaded libraries 0
    3CLNMBRLOADEDCL         Number of loaded classes 2
    2CLTEXTCLLOADER     -----ta- Loader jdk/internal/loader/ClassLoaders$AppClassLoader(0x00000000FFE1DAD0), Parent jdk/internal/loader/ClassLoaders$PlatformClassLoader(0x00000000FFE1D4F0)
    3CLNMBRLOADEDLIB        Number of loaded libraries 0
    3CLNMBRLOADEDCL         Number of loaded classes 0
    1CLTEXTCLLIB    ClassLoader loaded libraries
    2CLTEXTCLLIB        Loader *System*(0x00000000FFE1D258)
    3CLTEXTLIB              /home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk/lib/compressedrefs/jclse9_29
    3CLTEXTLIB              /home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk/lib/java
    3CLTEXTLIB              /home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk/lib/compressedrefs/j9jit29
    3CLTEXTLIB              /home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk/lib/zip
    3CLTEXTLIB              /home/me/openj9-openjdk-jdk9/build/linux-x86_64-normal-server-release/images/jdk/lib/nio
    1CLTEXTCLLOD    ClassLoader loaded classes
    2CLTEXTCLLOAD       Loader *System*(0x00000000FFE1D258)
    3CLTEXTCLASS            [Ljava/lang/Thread$State;(0x0000000001056400)
    ...
    2CLTEXTCLLOAD       Loader jdk/internal/loader/ClassLoaders$PlatformClassLoader(0x00000000FFE1D4F0)
    2CLTEXTCLLOAD       Loader java/lang/InternalAnonymousClassLoader(0x00000000FFE1DFD0)
    3CLTEXTCLASS            jdk/internal/loader/BuiltinClassLoader$$Lambda$2/00000000F03876A0(0x0000000001030F00)
    3CLTEXTCLASS            jdk/internal/loader/BuiltinClassLoader$$Lambda$1/00000000F00D2460(0x0000000001018A00)
    2CLTEXTCLLOAD       Loader jdk/internal/loader/ClassLoaders$AppClassLoader(0x00000000FFE1DAD0)

## 实战
后续会根据团队遇到的问题更新，这是学习使用Javadump文件非常好的素材

### GPF错误
这个案例中，Java应用程序由于GPF（General Protection Fault）错误崩溃，自动进行了Java转储

首先从TITLE部分观察到转储的原因是GPF


    0SECTION       TITLE subcomponent dump routine
    NULL           ===============================
    1TICHARSET     UTF-8
    1TISIGINFO     Dump Event "gpf" (00002000) received
    1TIDATETIME    Date: 2018/09/24 at 15:18:03:115
    1TINANOTIME    System nanotime: 4498949283020796
    1TIFILENAME    Javacore filename:    /home/test/JNICrasher/javacore.20180924.151801.29399.0002.txt
    1TIREQFLAGS    Request Flags: 0x81 (exclusive+preempt)
    1TIPREPSTATE   Prep State: 0x100 (trace_disabled)
    1TIPREPINFO    Exclusive VM access not taken: data may not be consistent across javacore sections
    
要定位这个问题，需要找到是哪个线程引起的GPF错误。在THREADS部分，  **Current thread**关键字指出了在应用崩溃时
正在运行的线程。下面是THREADS部分的详细信息
  
    NULL           ------------------------------------------------------------------------
    0SECTION       THREADS subcomponent dump routine
    NULL           =================================
    NULL
    1XMPOOLINFO    JVM Thread pool info:
    2XMPOOLTOTAL       Current total number of pooled threads: 16
    2XMPOOLLIVE        Current total number of live threads: 15
    2XMPOOLDAEMON      Current total number of live daemon threads: 14
    NULL            
    1XMCURTHDINFO  Current thread
    3XMTHREADINFO      "main" J9VMThread:0xB6B60E00, omrthread_t:0xB6B049D8, java/lang/Thread:0xB55444D0, state:R, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0x1, isDaemon:false)
    3XMTHREADINFO1            (native thread ID:0x72D8, native priority:0x5, native policy:UNKNOWN, vmstate:R, vm thread flags:0x00000000)
    3XMTHREADINFO2            (native stack address range from:0xB6CE3000, to:0xB74E4000, size:0x801000)
    3XMCPUTIME               CPU usage total: 0.319865924 secs, current category="Application"
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=778008 (0xBDF18)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at JNICrasher.doSomethingThatCrashes(Native Method)
    4XESTACKTRACE                at JNICrasher.main(JNICrasher.java:7)
    3XMTHREADINFO3           Native callstack:
    4XENATIVESTACK               (0xB6C6F663 [libj9prt29.so+0x3b663])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB6C6F1CE [libj9prt29.so+0x3b1ce])
    4XENATIVESTACK               (0xB6C6F2C6 [libj9prt29.so+0x3b2c6])
    4XENATIVESTACK               (0xB6C6ED93 [libj9prt29.so+0x3ad93])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB6C6ED07 [libj9prt29.so+0x3ad07])
    4XENATIVESTACK               (0xB6C6AA3D [libj9prt29.so+0x36a3d])
    4XENATIVESTACK               (0xB6C6C3A4 [libj9prt29.so+0x383a4])
    4XENATIVESTACK               (0xB667FA19 [libj9dmp29.so+0xfa19])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB66878CF [libj9dmp29.so+0x178cf])
    4XENATIVESTACK               (0xB6688083 [libj9dmp29.so+0x18083])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB6680C0D [libj9dmp29.so+0x10c0d])
    4XENATIVESTACK               (0xB667F9D7 [libj9dmp29.so+0xf9d7])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB668B02F [libj9dmp29.so+0x1b02f])
    4XENATIVESTACK               (0xB668B4D3 [libj9dmp29.so+0x1b4d3])
    4XENATIVESTACK               (0xB66740F1 [libj9dmp29.so+0x40f1])
    4XENATIVESTACK               (0xB66726FA [libj9dmp29.so+0x26fa])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB66726A9 [libj9dmp29.so+0x26a9])
    4XENATIVESTACK               (0xB6676AE4 [libj9dmp29.so+0x6ae4])
    4XENATIVESTACK               (0xB668D75A [libj9dmp29.so+0x1d75a])
    4XENATIVESTACK               (0xB6A28DD4 [libj9vm29.so+0x81dd4])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB6A289EE [libj9vm29.so+0x819ee])
    4XENATIVESTACK               (0xB6A29A40 [libj9vm29.so+0x82a40])
    4XENATIVESTACK               (0xB6C52B6A [libj9prt29.so+0x1eb6a])
    4XENATIVESTACK               __kernel_rt_sigreturn+0x0 (0xB7747410)
    4XENATIVESTACK               (0xB75330B6 [libffi29.so+0x50b6])
    4XENATIVESTACK               ffi_raw_call+0xad (0xB7531C53 [libffi29.so+0x3c53])
    4XENATIVESTACK               (0xB69BE4AB [libj9vm29.so+0x174ab])
    4XENATIVESTACK               (0xB6A665BC [libj9vm29.so+0xbf5bc])
    4XENATIVESTACK               (0xB6A15552 [libj9vm29.so+0x6e552])
    4XENATIVESTACK               (0xB6A30894 [libj9vm29.so+0x89894])
    4XENATIVESTACK               (0xB6A6F169 [libj9vm29.so+0xc8169])
    4XENATIVESTACK               (0xB6C52F6E [libj9prt29.so+0x1ef6e])
    4XENATIVESTACK               (0xB6A6F1FA [libj9vm29.so+0xc81fa])
    4XENATIVESTACK               (0xB6A30994 [libj9vm29.so+0x89994])
    4XENATIVESTACK               (0xB6A2CE4C [libj9vm29.so+0x85e4c])
    4XENATIVESTACK               (0xB770487D [libjli.so+0x787d])
    4XENATIVESTACK               (0xB7719F72 [libpthread.so.0+0x6f72])
    4XENATIVESTACK               clone+0x5e (0xB763543E [libc.so.6+0xee43e])
    

从以上详细信息可以看出，崩溃时正在运行的线程是java/lang/Thread，可以看到Java调用栈和本地调用栈。
为了在应用程序中模拟崩溃，这个例子调用了一个JNI的方法JNIcrasher，本地调用栈显示了失败时的部分信息，
但没有包含任何可以帮助你定位错误的方法名，因此你可以继续使用Systemdump，使用
 Dump viewer工具打开Systemdump文件，使用info thread命令打印这个线程的Java调用栈和本地调用栈。
 所以一般Systemdump可以用来辅助Javadump定位问题。
 
 
### OOM错误

这个案例中，Java应用程序耗尽了内存，导致OutOfMemoryError，自动进行了Java转储

首先从TITLE部分观察到Javadump的原因是systhrow，systhrow的详细原因是java/lang/OutOfMemoryError

    0SECTION       TITLE subcomponent dump routine
    NULL           ===============================
    1TICHARSET     UTF-8
    1TISIGINFO     Dump Event "systhrow" (00040000) Detail "java/lang/OutOfMemoryError" "Java heap space" received
    1TIDATETIME    Date: 2018/09/14 at 15:29:42:709
    1TINANOTIME    System nanotime: 3635648876608448
    1TIFILENAME    Javacore filename:    /home/cheesemp/test/javacore.20180914.152929.18885.0003.txt
    1TIREQFLAGS    Request Flags: 0x81 (exclusive+preempt)
    1TIPREPSTATE   Prep State: 0x104 (exclusive_vm_access+trace_disabled)
    
接下来可以观察MEMINFO部分，这部分可以观察到分配了多少堆内存用于存储对象（**1STHEAPTYPE**），已经使用了多少，
仍空闲多少。最简单的解决问题的方法就是给你的应用程序设置一个更大的堆内存

如果你不清楚设置多大的堆内存才合适，你可以观察ENVINFO部分，这部分提供了应用程序的启动参数。
查找**1CIUSERARGS**关键字，Java堆大小通过-Xmx参数设置，如果这个参数还没有设置，会使用默认值。

这个案例中，并不是通过调大堆内存来解决问题的。我们继续观察MEMINFO部分

 
    0SECTION       MEMINFO subcomponent dump routine
    NULL           =================================
    NULL
    1STHEAPTYPE    Object Memory
    NULL           id         start      end        size       space/region
    1STHEAPSPACE   0xB6B49D20     --         --         --     Generational
    1STHEAPREGION  0xB6B4A078 0x95750000 0xB5470000 0x1FD20000 Generational/Tenured Region
    1STHEAPREGION  0xB6B49F10 0xB5470000 0xB54C0000 0x00050000 Generational/Nursery Region
    1STHEAPREGION  0xB6B49DA8 0xB54C0000 0xB5750000 0x00290000 Generational/Nursery Region
    NULL
    1STHEAPTOTAL   Total memory:         536870912 (0x20000000)
    1STHEAPINUSE   Total memory in use:  302603160 (0x12095B98)
    1STHEAPFREE    Total memory free:    234267752 (0x0DF6A468)
    
这部分显示只有56%的堆内存被使用，因此建议应用程序进行调优，所以需要观察是哪个线程导致的OOM，
这个线程正在试图干什么。和上一个案例类似，你可以观察THREADS部分的当前线程，下面是**Current Thread**的输出  
 
    0SECTION       THREADS subcomponent dump routine
    NULL           =================================
    NULL
    1XMPOOLINFO    JVM Thread pool info:
    2XMPOOLTOTAL       Current total number of pooled threads: 16
    2XMPOOLLIVE        Current total number of live threads: 16
    2XMPOOLDAEMON      Current total number of live daemon threads: 15
    NULL
    1XMCURTHDINFO  Current thread
    3XMTHREADINFO      "main" J9VMThread:0xB6B60C00, omrthread_t:0xB6B049D8, java/lang/Thread:0x95764520, state:R, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0x1, isDaemon:false)
    3XMTHREADINFO1            (native thread ID:0x49C6, native priority:0x5, native policy:UNKNOWN, vmstate:R, vm thread flags:0x00001020)
    3XMTHREADINFO2            (native stack address range from:0xB6CB5000, to:0xB74B6000, size:0x801000)
    3XMCPUTIME               CPU usage total: 8.537823831 secs, current category="Application"
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at java/lang/StringBuffer.ensureCapacityImpl(StringBuffer.java:696)
    4XESTACKTRACE                at java/lang/StringBuffer.append(StringBuffer.java:486(Compiled Code))
    5XESTACKTRACE                   (entered lock: java/lang/StringBuffer@0x957645B8, entry count: 1)
    4XESTACKTRACE                at java/lang/StringBuffer.append(StringBuffer.java:428(Compiled Code))
    4XESTACKTRACE                at HeapBreaker.main(HeapBreaker.java:34(Compiled Code))
    3XMTHREADINFO3           Native callstack:
    4XENATIVESTACK               (0xB6C535B3 [libj9prt29.so+0x3b5b3])
    4XENATIVESTACK               (0xB6C36F3E [libj9prt29.so+0x1ef3e])
    4XENATIVESTACK               (0xB6C5311E [libj9prt29.so+0x3b11e])
    4XENATIVESTACK               (0xB6C53216 [libj9prt29.so+0x3b216])
    4XENATIVESTACK               (0xB6C52CE3 [libj9prt29.so+0x3ace3])
    4XENATIVESTACK               (0xB6C36F3E [libj9prt29.so+0x1ef3e])
    4XENATIVESTACK               (0xB6C52C57 [libj9prt29.so+0x3ac57])
    4XENATIVESTACK               (0xB6C4E9CD [libj9prt29.so+0x369cd])
    4XENATIVESTACK               (0xB6C502FA [libj9prt29.so+0x382fa])
    
为了模拟Java的OOM，这个案例的应用程序使用一个死循环不断的在StringBuffer对象上追加字符串。
Java调用堆显示出HeapBreaker的main程序在不断通过java/lang/StringBuffer.append方法追加字符串，
某次追加调用java/lang/StringBuffer.ensureCapacityImpl()这个方法时抛出了OOM错误。

StringBuffer对象底层是char数组，当达到数组的最大容量时，当前数组中的内容就要拷贝到一个新的、容量更大的数组中。
新建一个更大的数组就是在StringBuffer.ensureCapacity()方法中实现的，方法运行过程中，难免要需要旧数组2倍的内存空间，
也就是在这个点，内存占用超出了Java堆所有的剩余空间，导致了OOM。

MEMINFO部分还能观察到一些导致OOM的大的内存分配请求。我们来观察GC History部分(**1STGCHTYPE**)，
这部分会显示触发GC活动的请求。下面的例子中，你可以观察到一次大的内存分配请求(关键字requestedbytes，请求大小=576M)直接导致了全局GC。
如果GC后仍不能释放足够多的空间以响应这个请求，那么也会导致OOM。


    1STGCHTYPE     GC History  
    3STHSTTYPE     14:29:29:580239000 GMT j9mm.101 -   J9AllocateIndexableObject() returning NULL! 0 bytes requested for object of class B6BBC300 from memory space 'Generational' id=B6B49D20
    3STHSTTYPE     14:29:29:579916000 GMT j9mm.134 -   Allocation failure end: newspace=2686912/3014656 oldspace=231597224/533856256 loa=5338112/5338112
    3STHSTTYPE     14:29:29:579905000 GMT j9mm.470 -   Allocation failure cycle end: newspace=2686912/3014656 oldspace=231597224/533856256 loa=5338112/5338112
    3STHSTTYPE     14:29:29:579859000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=234284136/536870912
    3STHSTTYPE     14:29:29:579807000 GMT j9mm.90 -   GlobalGC collect complete
    3STHSTTYPE     14:29:29:579776000 GMT j9mm.137 -   Compact end: bytesmoved=301989896
    3STHSTTYPE     14:29:29:313899000 GMT j9mm.136 -   Compact start: reason=compact to meet allocation
    3STHSTTYPE     14:29:29:313555000 GMT j9mm.57 -   Sweep end
    3STHSTTYPE     14:29:29:310772000 GMT j9mm.56 -   Sweep start
    3STHSTTYPE     14:29:29:310765000 GMT j9mm.94 -   Class unloading end: classloadersunloaded=0 classesunloaded=0
    3STHSTTYPE     14:29:29:310753000 GMT j9mm.60 -   Class unloading start
    3STHSTTYPE     14:29:29:310750000 GMT j9mm.55 -   Mark end
    3STHSTTYPE     14:29:29:306013000 GMT j9mm.54 -   Mark start
    3STHSTTYPE     14:29:29:305957000 GMT j9mm.474 -   GlobalGC start: globalcount=9
    3STHSTTYPE     14:29:29:305888000 GMT j9mm.475 -   GlobalGC end: workstackoverflow=0 overflowcount=0 memory=234284136/536870912
    3STHSTTYPE     14:29:29:305837000 GMT j9mm.90 -   GlobalGC collect complete
    3STHSTTYPE     14:29:29:305808000 GMT j9mm.137 -   Compact end: bytesmoved=189784
    3STHSTTYPE     14:29:29:298042000 GMT j9mm.136 -   Compact start: reason=compact to meet allocation
    3STHSTTYPE     14:29:29:297695000 GMT j9mm.57 -   Sweep end
    3STHSTTYPE     14:29:29:291696000 GMT j9mm.56 -   Sweep start
    3STHSTTYPE     14:29:29:291692000 GMT j9mm.55 -   Mark end
    3STHSTTYPE     14:29:29:284994000 GMT j9mm.54 -   Mark start
    3STHSTTYPE     14:29:29:284941000 GMT j9mm.474 -   GlobalGC start: globalcount=8
    3STHSTTYPE     14:29:29:284916000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=0.016 meanexclusiveaccessms=0.016 threads=0 lastthreadtid=0xB6B61100 beatenbyotherthread=0
    3STHSTTYPE     14:29:29:284914000 GMT j9mm.469 -   Allocation failure cycle start: newspace=2678784/3014656 oldspace=80601248/533856256 loa=5338112/5338112 requestedbytes=603979784
    3STHSTTYPE     14:29:29:284893000 GMT j9mm.470 -   Allocation failure cycle end: newspace=2678784/3014656 oldspace=80601248/533856256 loa=5338112/5338112
    3STHSTTYPE     14:29:29:284858000 GMT j9mm.560 -   LocalGC end: rememberedsetoverflow=0 causedrememberedsetoverflow=0 scancacheoverflow=0 failedflipcount=0 failedflipbytes=0 failedtenurecount=0 failedtenurebytes=0 flipcount=2 flipbytes=64 newspace=2678784/3014656 oldspace=80601248/533856256 loa=5338112/5338112 tenureage=0
    3STHSTTYPE     14:29:29:284140000 GMT j9mm.140 -   Tilt ratio: 89
    3STHSTTYPE     14:29:29:283160000 GMT j9mm.64 -   LocalGC start: globalcount=8 scavengecount=335 weakrefs=0 soft=0 phantom=0 finalizers=0
    3STHSTTYPE     14:29:29:283123000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=0.016 meanexclusiveaccessms=0.016 threads=0 lastthreadtid=0xB6B61100 beatenbyotherthread=0
    3STHSTTYPE     14:29:29:283120000 GMT j9mm.469 -   Allocation failure cycle start: newspace=753616/3014656 oldspace=80601248/533856256 loa=5338112/5338112 requestedbytes=603979784
    3STHSTTYPE     14:29:29:283117000 GMT j9mm.133 -   Allocation failure start: newspace=753616/3014656 oldspace=80601248/533856256 loa=5338112/5338112 requestedbytes=603979784
    3STHSTTYPE     14:29:29:269762000 GMT j9mm.134 -   Allocation failure end: newspace=2686928/3014656 oldspace=80601248/533856256 loa=5338112/5338112
    3STHSTTYPE     14:29:29:269751000 GMT j9mm.470 -   Allocation failure cycle end: newspace=2686976/3014656 oldspace=80601248/533856256 loa=5338112/5338112
    3STHSTTYPE     14:29:29:269718000 GMT j9mm.560 -   LocalGC end: rememberedsetoverflow=0 causedrememberedsetoverflow=0 scancacheoverflow=0 failedflipcount=0 failedflipbytes=0 failedtenurecount=0 failedtenurebytes=0 flipcount=0 flipbytes=0 newspace=2686976/3014656 oldspace=80601248/533856256 loa=5338112/5338112 tenureage=0
    3STHSTTYPE     14:29:29:268981000 GMT j9mm.140 -   Tilt ratio: 89
    3STHSTTYPE     14:29:29:268007000 GMT j9mm.64 -   LocalGC start: globalcount=8 scavengecount=334 weakrefs=0 soft=0 phantom=0 finalizers=0
    3STHSTTYPE     14:29:29:267969000 GMT j9mm.135 -   Exclusive access: exclusiveaccessms=0.016 meanexclusiveaccessms=0.016 threads=0 lastthreadtid=0xB6B61100 beatenbyotherthread=0
    3STHSTTYPE     14:29:29:267966000 GMT j9mm.469 -   Allocation failure cycle start: newspace=0/3014656 oldspace=80601248/533856256 loa=5338112/5338112 requestedbytes=48
    3STHSTTYPE     14:29:29:267963000 GMT j9mm.133 -   Allocation failure start: newspace=0/3014656 oldspace=80601248/533856256 loa=5338112/5338112 requestedbytes=48
    3STHSTTYPE     14:29:29:249015000 GMT j9mm.134 -   Allocation failure end: newspace=2686928/3014656 oldspace=80601248/533856256 loa=5338112/5338112
    3STHSTTYPE     14:29:29:249003000 GMT j9mm.470 -   Allocation failure cycle end: newspace=2686976/3014656 oldspace=80601248/533856256 loa=5338112/5338112
    3STHSTTYPE     14:29:29:248971000 GMT j9mm.560 -   LocalGC end: rememberedsetoverflow=0 causedrememberedsetoverflow=0 scancacheoverflow=0 failedflipcount=0 failedflipbytes=0 failedtenurecount=0 failedtenurebytes=0 flipcount=0 flipbytes=0 newspace=2686976/3014656 oldspace=80601248/533856256 loa=5338112/5338112 tenureage=0
    

尽管这个例子中是Java代码故意造成的OOM，但实际应用中，类似的内存分配问题常有发生，例如在处理包含大量数据的XML文件时。

定位OOM问题接下来的步骤是打开Systemdump文件。可以使用MAT(Eclipse Memory Analyzer tool)
打开dump文件，查找StringBuffer对象，可以找到更多导致OOM的线索。最普遍的场景是，可以观察到相同的字符串
一遍又一遍的创建，那可能会让你联想到代码存在死循环。

注意：如果你使用MAT去分析Systemdump文件，你必须在Eclipse中
安装DTFJ(Diagnostic Tool Framework for Java)插件，可以通过下面的菜单安装DTFJ
 
    Help > Install New Software > Work with "IBM Diagnostic Tool Framework for Java" >  
  
与之前的案例不同，如果你的应用发生了OOM，并且从MEMIFO部分观察到只有很少的堆内存空闲，那么，当前是哪个线程
就不重要了，任何线程都可能由于被调度到并分配内存而导致应用的OOM。这种情况下，你可能需要增大堆内存
或者通过调优应用来解决。


### 本地OOM错误
这个案例中，虚拟机占用的内存溢出。虚拟机占用的内存包括虚拟机存储用来进行操作的资源和数据。
虚拟机进程可以使用的本地内存受限于操作系统，例如Unix的ulimits

当本地OOM错误发生时，会自动进行Java转储。可以从转储文件的TITLE部分观察到Java转储的原因是systhrow，
systhrow的详细原因是java/lang/OutOfMemoryError，本地内存耗尽。

    0SECTION       TITLE subcomponent dump routine
    NULL           ===============================
    1TICHARSET     UTF-8
    1TISIGINFO     Dump Event "systhrow" (00040000) Detail "java/lang/OutOfMemoryError" "native memory exhausted" received
    1TIDATETIME    Date: 2018/09/14 at 15:49:55:887
    1TINANOTIME    System nanotime: 3636862054495675
    1TIFILENAME    Javacore filename:    /home/cheesemp/test/javacore.20180914.154814.19708.0003.txt
    1TIREQFLAGS    Request Flags: 0x81 (exclusive+preempt)
    1TIPREPSTATE   Prep State: 0x104 (exclusive_vm_access+trace_disabled)
    
有时，是当前线程导致的本地OOM，可以通过观察THREADS部分的Current Thread来继续诊断

    0SECTION       THREADS subcomponent dump routine
    NULL           =================================
    NULL
    1XMPOOLINFO    JVM Thread pool info:
    2XMPOOLTOTAL       Current total number of pooled threads: 16
    2XMPOOLLIVE        Current total number of live threads: 16
    2XMPOOLDAEMON      Current total number of live daemon threads: 15
    NULL            
    1XMCURTHDINFO  Current thread
    3XMTHREADINFO      "main" J9VMThread:0xB6C60C00, omrthread_t:0xB6C049D8, java/lang/Thread:0xB55E3C10, state:R, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0x1, isDaemon:false)
    3XMTHREADINFO1            (native thread ID:0x4CFD, native priority:0x5, native policy:UNKNOWN, vmstate:R, vm thread flags:0x00001020)
    3XMTHREADINFO2            (native stack address range from:0xB6D4E000, to:0xB754F000, size:0x801000)
    3XMCPUTIME               CPU usage total: 3.654896026 secs, current category="Application"
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at sun/misc/Unsafe.allocateDBBMemory(Native Method)
    4XESTACKTRACE                at java/nio/DirectByteBuffer.<init>(DirectByteBuffer.java:127(Compiled Code))
    4XESTACKTRACE                at java/nio/ByteBuffer.allocateDirect(ByteBuffer.java:311)
    4XESTACKTRACE                at NativeHeapBreaker.main(NativeHeapBreaker.java:9)
    3XMTHREADINFO3           Native callstack:
    4XENATIVESTACK               (0xB6A9F5B3 [libj9prt29.so+0x3b5b3])
    ...
    4XENATIVESTACK               (0xB582CC9C [libjclse7b_29.so+0x40c9c])
    4XENATIVESTACK               Java_sun_misc_Unsafe_allocateDBBMemory+0x88 (0xB5827F6B [libjclse7b_29.so+0x3bf6b])
    4XENATIVESTACK               (0x94A2084A [<unknown>+0x0])
    4XENATIVESTACK               (0xB6B2538B [libj9vm29.so+0x6c38b])
    4XENATIVESTACK               (0xB6B4074C [libj9vm29.so+0x8774c])
    4XENATIVESTACK               (0xB6B7F299 [libj9vm29.so+0xc6299])
    4XENATIVESTACK               (0xB6A82F3E [libj9prt29.so+0x1ef3e])
    4XENATIVESTACK               (0xB6B7F32A [libj9vm29.so+0xc632a])
    4XENATIVESTACK               (0xB6B4084C [libj9vm29.so+0x8784c])
    4XENATIVESTACK               (0xB6B3CD0C [libj9vm29.so+0x83d0c])
    4XENATIVESTACK               (0xB776F87D [libjli.so+0x787d])
    4XENATIVESTACK               (0xB7784F72 [libpthread.so.0+0x6f72])
    4XENATIVESTACK               clone+0x5e (0xB76A043E [libc.so.6+0xee43e])

为了更加清晰，略去了部分本地调用堆栈的输出，用...表示

Java调用堆栈显示了从Java代码(sun/misc/Unsafe.allocateDBBMemory(Native Method))到本地代码的调用关系
表明应用程序在请求堆外内存，Java堆外内存功能的底层正是本地内存，Java堆只是维护一个引用指向本地堆缓存。
这个案例中，堆外内存可能正是本地OOM的罪魁祸首。

定位本地OOM问题的下一步是观察Javadump文件的NATIVEMEMINFO部分，可以看到JRE进程整体占用的内存，还可以按组件观察每个组件占用的内存

    0SECTION       NATIVEMEMINFO subcomponent dump routine
    NULL           =================================
    0MEMUSER
    1MEMUSER       JRE: 3,166,386,688 bytes / 4388 allocations
    1MEMUSER       |
    2MEMUSER       +--VM: 563,176,824 bytes / 1518 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Classes: 3,104,416 bytes / 120 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Memory Manager (GC): 548,181,888 bytes / 398 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Java Heap: 536,932,352 bytes / 1 allocation
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Other: 11,249,536 bytes / 397 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Threads: 10,817,120 bytes / 147 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Java Stack: 115,584 bytes / 16 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Native Stack: 10,616,832 bytes / 17 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Other: 84,704 bytes / 114 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Trace: 163,688 bytes / 268 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--JVMTI: 17,320 bytes / 13 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--JNI: 23,296 bytes / 55 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Port Library: 8,576 bytes / 74 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Other: 860,520 bytes / 443 allocations
    1MEMUSER       |
    2MEMUSER       +--JIT: 3,744,728 bytes / 122 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--JIT Code Cache: 2,097,152 bytes / 1 allocation
    2MEMUSER       |  |
    3MEMUSER       |  +--JIT Data Cache: 524,336 bytes / 1 allocation
    2MEMUSER       |  |
    3MEMUSER       |  +--Other: 1,123,240 bytes / 120 allocations
    1MEMUSER       |
    2MEMUSER       +--Class Libraries: 2,599,463,024 bytes / 2732 allocations
    2MEMUSER       |  |
    3MEMUSER       |  +--Harmony Class Libraries: 1,024 bytes / 1 allocation
    2MEMUSER       |  |
    3MEMUSER       |  +--VM Class Libraries: 2,599,462,000 bytes / 2731 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--sun.misc.Unsafe: 2,598,510,480 bytes / 2484 allocations
    4MEMUSER       |  |  |  |
    5MEMUSER       |  |  |  +--Direct Byte Buffers: 2,598,510,480 bytes / 2484 allocations
    3MEMUSER       |  |  |
    4MEMUSER       |  |  +--Other: 951,520 bytes / 247 allocations
    1MEMUSER       |
    2MEMUSER       +--Unknown: 2,112 bytes / 16 allocations
    NULL           

观察VM Class Libraries部分，可以看见这部分占用堆外内存大小。由于本案例是在32位系统上发生的本地OOM，那么
2,598,510,480 字节基本可以认为是Unix操作系统的物理内存耗尽，进程耗尽操作系统的内存一般是由于ulimit设置，
增加ulimit的设置可能能避免这个错误，可以在当前会话通过ulimit -f 命令来临时设置阈值。

理论上32位虚拟机最大可占用的内存空间是32位地址空间，即4G。大多数操作系统中，每个进程中有部分地址会被内核占用，
因此实际可以利用的内存空间明显小于4G。结果是，在32位虚拟机上耗尽内存的现象非常普遍。

4G限制同样影响到64位虚拟机。在引用压缩模式下，为了提升性能，所有对象、类、线程、锁的引用都是32位值，
so these structures can be allocated only at 32-bit addresses. However,
 the operating system might place other allocations within this 4 GB of address space,
 当这个区域被充满或者有大量碎片时，虚拟机就会抛出NativeOutOfMemoryError错误。这个错误时常发生在虚拟机试图
 创建一个新线程或加载一个类时。在本地OOM发生时，Current Thread History部分可能会包含更多虚拟机层面的信息。
 
你可以通过设置-Xmcrs参数避免此问题，关于-Xmcrs，参考[这里](https://www.eclipse.org/openj9/docs/xmcrs/)

另一个导致本地OOM错误的常见原因是类重复加载，很可能类在堆外也被加载了一遍。如果NATIVEMEMINFO部分
显示的类分配值特别大，那么很可能就是类重复加载的问题。通过MAT的Class Loader Explorer特性可以观察到类重复
加载的问题。

### 死锁

死锁是指两个或多个线程互相持有对方需要的锁。死锁发生时，你的应用程序将停止响应并且挂住。此时生成一个Javadump文件
会很快帮助你定位到发生死锁的线程。这种情况下，你可以发送SIGQUIT信号(kill -3)给虚拟机，触发转储。

虚拟机会检测到最常见的死锁场景。如果检测到死锁，LOCKS部分就会相关的信息。一些更加复杂的死锁，比如本地互斥锁和Java
锁的死锁，就可能无法检测出来了。

下面的例子是比较普通的死锁的Javadump的输出
    
    NULL           
    1LKDEADLOCK    Deadlock detected !!!
    NULL           ---------------------
    NULL           
    2LKDEADLOCKTHR  Thread "Worker Thread 2" (0x94501D00)
    3LKDEADLOCKWTR    is waiting for:
    4LKDEADLOCKMON      sys_mon_t:0x08C2B344 infl_mon_t: 0x08C2B384:
    4LKDEADLOCKOBJ      java/lang/Object@0xB5666698
    3LKDEADLOCKOWN    which is owned by:
    2LKDEADLOCKTHR  Thread "Worker Thread 3" (0x94507500)
    3LKDEADLOCKWTR    which is waiting for:
    4LKDEADLOCKMON      sys_mon_t:0x08C2B3A0 infl_mon_t: 0x08C2B3E0:
    4LKDEADLOCKOBJ      java/lang/Object@0xB5666678
    3LKDEADLOCKOWN    which is owned by:
    2LKDEADLOCKTHR  Thread "Worker Thread 1" (0x92A3EC00)
    3LKDEADLOCKWTR    which is waiting for:
    4LKDEADLOCKMON      sys_mon_t:0x08C2B2E8 infl_mon_t: 0x08C2B328:
    4LKDEADLOCKOBJ      java/lang/Object@0xB5666688
    3LKDEADLOCKOWN    which is owned by:
    2LKDEADLOCKTHR  Thread "Worker Thread 2" (0x94501D00)
    
这段信息告诉你Worker Thread 2等待Worker Thread 3，而Worker Thread 3在等待Worker Thread 1，
Worker Thread 1在等待Worker Thread 2，这就形成了死锁。还可以在THREADS部分的Java方法栈和本地方法栈观察到
死锁的现象。通过观察每个Worker Thread的方法栈，也能跟踪到造成此死锁的具体代码行。

这个案例中，你可以观察到所有工作线程的输出(**4XESTACKTRACE/5XESTACKTRACE**) ，在DeadLockTest.java的35行导致了死锁的问题

    3XMTHREADINFO      "Worker Thread 1" J9VMThread:0x92A3EC00, omrthread_t:0x92A3C2B0, java/lang/Thread:0xB5666778, state:B, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0x13, isDaemon:false)
    3XMTHREADINFO1            (native thread ID:0x52CF, native priority:0x5, native policy:UNKNOWN, vmstate:B, vm thread flags:0x00000201)
    3XMTHREADINFO2            (native stack address range from:0x9297E000, to:0x929BF000, size:0x41000)
    3XMCPUTIME               CPU usage total: 0.004365543 secs, current category="Application"
    3XMTHREADBLOCK     Blocked on: java/lang/Object@0xB5666688 Owned by: "Worker Thread 2" (J9VMThread:0x94501D00, java/lang/Thread:0xB56668D0)
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at WorkerThread.run(DeadLockTest.java:35)
    5XESTACKTRACE                   (entered lock: java/lang/Object@0xB5666678, entry count: 1)
    ...
    3XMTHREADINFO      "Worker Thread 2" J9VMThread:0x94501D00, omrthread_t:0x92A3C8F0, java/lang/Thread:0xB56668D0, state:B, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0x14, isDaemon:false)
    3XMTHREADINFO1            (native thread ID:0x52D0, native priority:0x5, native policy:UNKNOWN, vmstate:B, vm thread flags:0x00000201)
    3XMTHREADINFO2            (native stack address range from:0x946BF000, to:0x94700000, size:0x41000)
    3XMCPUTIME               CPU usage total: 0.004555580 secs, current category="Application"
    3XMTHREADBLOCK     Blocked on: java/lang/Object@0xB5666698 Owned by: "Worker Thread 3" (J9VMThread:0x94507500, java/lang/Thread:0xB5666A18)
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at WorkerThread.run(DeadLockTest.java:35)
    5XESTACKTRACE                   (entered lock: java/lang/Object@0xB5666688, entry count: 1)
    ...
    3XMTHREADINFO      "Worker Thread 3" J9VMThread:0x94507500, omrthread_t:0x92A3CC10, java/lang/Thread:0xB5666A18, state:B, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0x15, isDaemon:false)
    3XMTHREADINFO1            (native thread ID:0x52D1, native priority:0x5, native policy:UNKNOWN, vmstate:B, vm thread flags:0x00000201)
    3XMTHREADINFO2            (native stack address range from:0x9467E000, to:0x946BF000, size:0x41000)
    3XMCPUTIME               CPU usage total: 0.003657010 secs, current category="Application"
    3XMTHREADBLOCK     Blocked on: java/lang/Object@0xB5666678 Owned by: "Worker Thread 1" (J9VMThread:0x92A3EC00, java/lang/Thread:0xB5666778)
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at WorkerThread.run(DeadLockTest.java:35)
    5XESTACKTRACE                   (entered lock: java/lang/Object@0xB5666698, entry count: 1)
    
    
### 挂死

应用程序可能由于很多原因而挂死，但是最常见的原因是GC活动导致的，在GC期间，应用程序会暂停。你可以通过观察GC日志来定位
挂死的问题，参数是-verbose:gc option

死锁问题对外表现也是应用程序挂死。通过Javadump的deadlock部分可以定位死锁问题。

如果你排除了GC活动和死锁，另一种常见的挂死场景是由于相关的线程在等待Java对象锁，这类问题也可以通过观察Java转储文件来定位。
最简单的锁等待挂死场景就是线程需要另一个线程持有的锁，但是另一个线程因为某种原因无法释放这个锁。

出现挂死问题，首先得观察LOCKS部分，这部分列出了所有的锁、持有锁的线程、等待锁的线程。你可以通过观察正在等待的线程
来定位挂死的问题。

这个案例中，Javadump的LOCKS部分显示Worker Thread 0(*3LKMONOBJECT*)持有了一把锁，有19个Worker Thread
正在等待他释放。

    NULL           ------------------------------------------------------------------------
    0SECTION       LOCKS subcomponent dump routine
    NULL           ===============================
    NULL           
    1LKPOOLINFO    Monitor pool info:
    2LKPOOLTOTAL     Current total number of monitors: 1
    NULL           
    1LKMONPOOLDUMP Monitor Pool Dump (flat & inflated object-monitors):
    2LKMONINUSE      sys_mon_t:0x92711200 infl_mon_t: 0x92711240:
    3LKMONOBJECT       java/lang/Object@0xB56658D8: Flat locked by "Worker Thread 0" (J9VMThread:0x92A3EC00), entry count 1
    3LKWAITERQ            Waiting to enter:
    3LKWAITER                "Worker Thread 1" (J9VMThread:0x92703F00)
    3LKWAITER                "Worker Thread 2" (J9VMThread:0x92709C00)
    3LKWAITER                "Worker Thread 3" (J9VMThread:0x92710A00)
    3LKWAITER                "Worker Thread 4" (J9VMThread:0x92717F00)
    3LKWAITER                "Worker Thread 5" (J9VMThread:0x9271DC00)
    3LKWAITER                "Worker Thread 6" (J9VMThread:0x92723A00)
    3LKWAITER                "Worker Thread 7" (J9VMThread:0x92729800)
    3LKWAITER                "Worker Thread 8" (J9VMThread:0x92733700)
    3LKWAITER                "Worker Thread 9" (J9VMThread:0x92739400)
    3LKWAITER                "Worker Thread 10" (J9VMThread:0x92740200)
    3LKWAITER                "Worker Thread 11" (J9VMThread:0x92748100)
    3LKWAITER                "Worker Thread 12" (J9VMThread:0x9274DF00)
    3LKWAITER                "Worker Thread 13" (J9VMThread:0x92754D00)
    3LKWAITER                "Worker Thread 14" (J9VMThread:0x9275AA00)
    3LKWAITER                "Worker Thread 15" (J9VMThread:0x92760800)
    3LKWAITER                "Worker Thread 16" (J9VMThread:0x92766600)
    3LKWAITER                "Worker Thread 17" (J9VMThread:0x9276C300)
    3LKWAITER                "Worker Thread 18" (J9VMThread:0x92773100)
    3LKWAITER                "Worker Thread 19" (J9VMThread:0x92778F00)
    NULL      
    
下一步是定位为什么 Worker Thread 0没有释放这个锁，最好的定位方法是通过线程名称或者J9VMThread ID来搜索，
看看这个线程当前在干什么

下面的部分显示了线程Worker Thread 0的详细信息

    NULL
    3XMTHREADINFO      "Worker Thread 0" J9VMThread:0x92A3EC00, omrthread_t:0x92A3C280, java/lang/Thread:0xB56668B8, state:CW, prio=5
    3XMJAVALTHREAD            (java/lang/Thread getId:0x13, isDaemon:false)
    3XMTHREADINFO1            (native thread ID:0x511F, native priority:0x5, native policy:UNKNOWN, vmstate:CW, vm thread flags:0x00000401)
    3XMTHREADINFO2            (native stack address range from:0x9297E000, to:0x929BF000, size:0x41000)
    3XMCPUTIME               CPU usage total: 0.000211878 secs, current category="Application"
    3XMHEAPALLOC             Heap bytes allocated since last GC cycle=0 (0x0)
    3XMTHREADINFO3           Java callstack:
    4XESTACKTRACE                at java/lang/Thread.sleep(Native Method)
    4XESTACKTRACE                at java/lang/Thread.sleep(Thread.java:941)
    4XESTACKTRACE                at WorkerThread.doWork(HangTest.java:37)
    4XESTACKTRACE                at WorkerThread.run(HangTest.java:31)
    5XESTACKTRACE                   (entered lock: java/lang/Object@0xB56658D8, entry count: 1)
    
在这段输出的最后一行，你可以看到Work Thread 0申请到一把锁(java/lang/Object@0xB56658D8),
申请到之后，线程的run方法开始运行，内部调用doWork方法。从方法栈可以看出这个线程在HangTest.java的37行
调用了java/lang/Thread.sleep方法，从而导致线程无法完成工作并且无法释放锁。这个例子中，sleep方法的调用
促使了应用程序的挂死，但在实际生产环境中，导致应用程序挂死的，可能是一个阻塞操作，
例如通过输入流或者套接字读取数据；也可能是一个线程在等待另一个线程持有的另一个锁。

有一点很重要，每一个Java转储文件只是一个时间点的快照。定位一个问题，你至少要每隔一段时间(例如30秒)转储1次，共转储3次，
然后比较每次转储的输出。通过比较你可以观察到哪些线程一直在运行，哪些线程已经运行结束。

这个例子中，即使多次转储，线程状态也没有发生改变。因此你需要聚焦到WorkerThread.doWork的实现逻辑。

另一种常见的场景是Javadump显示了大量的线程在等待一个由另一个线程持有的锁，但等待的线程和持有锁的线程在不断变化。
那么这个场景的瓶颈就可能是线程竞争问题，多个线程持续不断的竞争一把锁。在严重的情况下，线程持有锁的时间也非常短暂，
应用程序处理锁、调度锁的耗时超过了执行应用程序代码的耗时，性能会急剧下降，对外表现也是应用程序挂死。线程竞争问题通常是
应用程序设计问题引起的，你可以使用本场景类似的方法，观察是哪些代码行在竞争锁。
   
