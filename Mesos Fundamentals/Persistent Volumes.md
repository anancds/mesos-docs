## Persistent Volumes

Mesos提供了一个从磁盘资源创建持久化卷的资源。当起一个任务时，可以在任务的sandbox外创建一个卷，然后持久化在这个节点上，甚至任务消亡或者结束也不会消失。当任务退出后，它的资源-包括持久化卷，可以提供给Framework。因此这个Framework可以又起同一个task，或者起恢复的task，或者一个新的task消费前面一个task的输出。持久化卷允许有状态的服务，比如HDFS和cassandra存储它们的数据在Mesos上，而不需要采取别的解决方法(比如把任务状态写入分布式文件系统，这个文件系统是挂载在任务sandbox外的外部都知道的一个位置)。

### Usage

持久化卷只能通过预留(**reserved**)的磁盘资源创建，不管是静态预留还是动态预留。一个动态预留的持久化卷如果不是显示的销毁持久化卷，那么就不能解除动态预留。这些规矩就是为了限制意外的错误，比如一个包含敏感数据的持久化卷提供给了集群中的别的Framework。相类似的，如果一个正在运行中的任务正在使用这个持久化卷，那么就不能销毁。

关于Mesos提供的预留机制，参考[Reservation](http://mesos.apache.org/documentation/latest/reservation/)文档。

通过预留[multiple disk resources](http://mesos.apache.org/documentation/latest/multiple-disk/)，持久化卷也可以创建在隔离的昂贵的磁盘上。

默认情况下，不同executors管理的运行中的task不同共享同一个持久化卷。一旦通过一个持久化卷起了一个任务后，那么这个持久化卷就不会出现在资源邀约中，除非任务结束运行。共享卷是这么一种持久化卷，就是在同一台agent上，多个任务可以同时访问这个共享卷。更多信息，查看[shared volumes](http://mesos.apache.org/documentation/latest/shared-resources/)。

持久化卷可以同
