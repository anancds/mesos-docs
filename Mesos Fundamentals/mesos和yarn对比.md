
### 编程语言

* yarn是java
* Mesos是c++
在语言上Mesos可能会有一定的优势。

### 架构

* Mesos是二级调度架构，也就是说master和agent管理资源上报和向Framework发出资源邀约，然后Framework负责资源的调度和任务的执行。
* yarn是master/slave的架构，是一个monolithic scheduling而不是二级调度框架。
这样再扩展性上mesos具有一定的优势。

### 管理资源类型

* yarn现在只管理Memory和CPUs。
* Mesos是：CPUs，GPU，disk，memory，ports。

### 资源分配

* yarn和Mesos的资源分配模块都是可插拔，算法都支持Dominant Resource Fairnes（DRF)，两者差不多。

* 在资源分配上Mesos有role，权重和资源预留，yarn有对应的概念比如队列，并可以为队列分配一定的比例，两者差不了多少。

### 资源分配粒度

* yarn是粗粒度的，分配的单位是container，资源一般是固定的，预先分配好，所以会有一定的预留，造成很大的浪费。
* Mesos是细粒度的，也就是说Mesos的master向Framework发出资源邀约后，Framework的scheduler可以选择接受全部的资源，也可以接受部分的资源，或者拒绝资源。

在资源利用率上Mesos因为是细粒度的，所以会好一些，但是yarn3.0后开始支持container resizing，目前特性比较新，还不清楚到底怎么样。

### 资源保证

* YARN采用的是增量资源分配机制，优先为应用程序预留一个节点上的资源，优点是不会发生饿死现象，但有一定的浪费。

* Mesos早期版本使用的是All or nothing的模式，可能会造成作业饿死（大资源的作业长时间无法得到满足)，但是目前已经支持资源预留和配额，所以也不会出现资源饿死现象。

两者差不多。

### 资源抢占

* yarn目前还不支持抢占。
* Mesos生态中的Aurora支持任务优先级，任务抢占和调度周期性任务，对长服务支持也很好。

这里Aurora的抢占是默认配置的，抢占是直接把运行的任务kill掉，机制是这样的，如果有两个任务，一个正在运行，然后新来一个任务，但是没有足够的资源运行，那么就有两种情况会抢占。第一是两个任务都在同一个role中，并且新来任务优先级高，还有一种情况就是运行的任务是一个可被抢占的任务，或者是一个可被撤销的任务，并且新来的任务是一个优先任务。

### 安全认证

* yarn有安全认证以及加密rpc连接。
* Mesos只有认证授权，并且支持SLA，目前没有传输加密。

### 资源隔离

* mesos支持linux container groups和docker隔离。
* yarn支持linux container groups，通过进程监控的方式控制内存，性能更高一点。

CPU资源比较特殊，是软隔离，如果一个应用分配的cpu少了，其实就是任务运行的慢一点，但是当应用超过分配给它的资源后，主要就是内存资源，Mesos和yarn一样，都是把应用的任务直接杀死。所以内存和磁盘空间是硬隔离，也就是说应用超过了分配给它的资源后，就会被杀死不能继续运行。

当一个任务在运行时，申请的内存超过了给它分配的内存，但是又小于机器内存总量，那么虽然可以申请到内存，但是运行一段时间后，还是会被kill掉。

### cgroup的资源隔离

通过代码验证了这样一种场景：假设应用程序每隔一秒钟申请1M内存，并且对申请内存有异常捕获，jvm退出时增加hook函数，cgroup的控制组分配的内存的20M，那么分如下几种情况。

* 如果是堆内存，采用默认的jvm参数，那么大概会在27M左右时被强制kill掉，并且捕获不到异常，也执行不了hook函数。
* 如果是堆内存，采用-Xmx15M -Xms15M的jvm参数，那么在超过15M时，会捕获到OOM的异常：Java heap space，并且执行了hook函数。
* 如果是堆外内存，采用默认的jvm参数，那么大概也是在27M左右会被kill掉，并且捕获不到异常，也执行不了hook函数。
* 如果是堆外内存，采用-XX:MaxDirectMemorySize=15M的jvm参数，那么在超过15M时，会捕获到OOM的异常： Direct buffer memory，并且执行了hook函数。


### 扩展性

* yarn是针对hadoop生态系统的，目前绑定的比较死。
* Mesos是数据中心的内核，只要去实现它提供的一些Framework的接口就可以了，扩展性比较好。

### 社区

* 目前yarn的社区人数稍多，但是Mesos正在赶上来，所以说yarn更成熟，但是Mesos前景更好一点，周边生态也更好一点。

### 目前支持的框架

这里只列出有可能跟我们相关的框架

Mesos支持如下：

* [Apache Spark](https://spark.apache.org/docs/latest/running-on-mesos.html)
* [Apache Hadoop](https://github.com/mesos/hadoop)
* [Flink](https://github.com/apache/flink/tree/master/flink-mesos)
* [Apache Storm](https://github.com/mesos/storm)
* [Apache Cassandra](https://github.com/mesosphere/ca)
* [ElasticSearch](https://github.com/mesos/elasticsearch)
* [Tachyon](https://github.com/mesosphere/tachyon-mesos)
* [HDFS](https://github.com/mesosphere/hdfs-deprecated)
* [Apache Kafka](https://github.com/mesos/kafka)
* [MongoDB](https://github.com/massenz/mongo_fw)
* [TensorFlow](https://github.com/douban/tfmesos)
* [Apache Aurora](http://aurora.apache.org/)
* [ZooKeeper](https://github.com/CiscoCloud/exhibitor-mesos-framework)
* [Kibana](https://github.com/mesos/kibana)
* [logstash](https://github.com/mesos/logstash)
* [Kubernetes](https://github.com/mesosphere/kubernetes-mesos)
* [redis](https://github.com/mesos/mr-redis)

yarn支持如下：

* hadoop
* Spark
* [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/hadoop/current/es-yarn.html)


### 应用场景

Mesos更加通用，是数据中心的内核，主要就是用来资源管理分配，因为粒度比较细，资源利用率高，所以比较适合有非常多的批处理应用同时跑，yarn更多的是和hadoop生态绑定在一起。
