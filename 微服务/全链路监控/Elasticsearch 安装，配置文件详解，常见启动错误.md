Elasticsearch 安装，配置文件详解，常见启动错误
=====================================


# ElasticSearch简介

* Elasticsearch 是一个分布式可扩展的实时搜索和分析引擎,一个建立在全文搜索引擎 Apache Lucene(TM) 基础上的搜索引擎.当然 Elasticsearch 并不仅仅是 Lucene 那么简单，它不仅包括了全文搜索功能，还可以进行以下工作:

      分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。
      实时分析的分布式搜索引擎。
      可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。

# ElasticSearch安装与气动

* https://www.elastic.co/cn/downloads/past-releases 下载es安装包
* 解压安装包
* 在bin目录下直接运行 elasticsearch.sh （Windows系统为：/elasticsearchbat）则可气动


# ElasticSearch配置详解

* cluster.name： 配置的集群名称，默认是elasticsearch，es服务会通过广播方式自动连接在同一网段下的es服务，通过多播方式进行通信，同一网段下可以有多个集群，通过集群名称这个属性来区分不同的集群
* node.name: 当前配置所在机器的节点名，你不设置就默认随机指定一个name列表中名字，该name列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字
* node.master： 指定该节点是否有资格被选举成为node（注意这里只是设置成有资格， 不代表该node一定就是master），默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master
* node.data: 指定该节点是否存储索引数据，默认为true
* index.number_of_shards: 设置默认索引分片个数，默认为5片
* index.number_of_replicas: 设置默认索引副本个数，默认为1个副本。如果采用默认设置，而你集群只配置了一台机器，那么集群的健康度为yellow，也就是所有的数据都是可用的，但是某些复制没有被分配（健康度可用  分为绿色、黄色或红色。绿色代表一切正常，集群功能齐全，黄色意味着所有的数据都是可用的，但是某些复制没有被分配，红色则代表因为某些原因，某些数据不可用）。
* path.conf： 设置配置文件的存储路径，默认是es根目录下的config文件夹
* path.data: 设置数据存储位置，默认值是es根目录下的data文件夹
* path.work: 设置临时文件的存储路径，默认是es根目录下的work文件夹
* path.logs：设置日志文件的存储路径，默认是es根目录下的logs文件夹
* path.plugins：设置插件的存放路径，默认是es根目录下的plugins文件夹, 插件在es里面普遍使用，用来增强原系统核心功能
* bootstrap.mlockall：设置为true来锁住内存不进行swapping。因为当jvm开始swapping时es的效率 会降低，所以要保证它不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。 同时也要允许elasticsearch的进程可以锁住内存，linux下启动es之前可以通过`ulimit -l unlimited`命令设置
* network.bind_host：设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0，绑定这台机器的任何一个ip
* network.publish_host：设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。
* network.host: 这个参数是用来同时设置bind_host和publish_host上面两个参数。
* transport.tcp.port: 设置节点之间交互的tcp端口，默认是9300。
* transport.tcp.compress: 设置是否压缩tcp传输时的数据，默认为false，不压缩。
* http.port: 设置对外服务的http端口，默认为9200。
* http.max_content_length: 设置内容的最大容量，默认100mb
* http.enabled: 是否使用http协议对外提供服务，默认为true，开启。
* gateway.type: gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器等。
* gateway.recover_after_nodes: 设置集群中N个节点启动时进行数据恢复，默认为1。
* gateway.recover_after_time: 设置初始化数据恢复进程的超时时间，默认是5分钟。
* gateway.expected_nodes: 设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。
* cluster.routing.allocation.node_initial_primaries_recoveries: 初始化数据恢复时，并发恢复线程的个数，默认为4。
* cluster.routing.allocation.node_concurrent_recoveries: 添加删除节点或负载均衡时并发恢复线程的个数，默认为4。
* indices.recovery.max_size_per_sec: 设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。
* indices.recovery.concurrent_streams: 设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。
* discovery.zen.minimum_master_nodes: 设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）
* discovery.zen.ping.timeout: 设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。
* discovery.zen.ping.multicast.enabled: 设置是否打开多播发现节点，默认是true。
* discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"] 设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。


# ElasticSearch常见异常
* can not run elasticsearch as root
      es无法用root账户气动，切换到非root账户即可
* main ERROR Could not register mbeans java.security.AccessControlException: access denied ("javax.management.MBeanTrustPermission" "register")
      当前用户无权限操作文件夹，改变es文件夹所有者为当前用户
* max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
      最大虚拟内存不够，sysctl.conf配置文件
      sudo vi /etc/sysctl.conf
      添加下面配置：
      vm.max_map_count=655360
      并执行命令：
      sudo sysctl -p
* max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
      文件打开数不够，修改limits.conf配置文件
      sudo vi /etc/security/limits.conf
      添加如下内容:
      * soft nofile 65536
      * hard nofile 131072
      * soft nproc 2048
      * hard nproc 4096

      sudo vi /etc/pam.d/common-session
      添加 session required pam_limits.so
      sudo vi /etc/pam.d/common-session-noninteractive

添加 session required pam_limits.so
* max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
      用户[esadmin]的最大线程数[1024]太少，至少增加到[4096]
      vim /etc/security/limits.d/90-nproc.conf
      把* soft nproc 1024修改成* soft nproc 2048
*  max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
      最大虚拟内存区vm.max_map_count[65530]太低，至少增加到[262144]
      使用root用户vi /etc/sysctl.conf
      添加下面配置
      vm.max_map_count=655360
      然后执行 sysctl –p

* java.lang.UnsupportedOperationException: seccomp unavailable: CONFIG_SECCOMP not compiled into kernel, CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER are needed
      因为Centos6不支持SecComp,而ES默认bootstrap.system_call_filter为true进行检测,所以导致检测失败,失败后直接导致ES不能启动解决:
      修改elasticsearch.yml,添加：
      bootstrap.memory_lock: false
      bootstrap.system_call_filter: false
