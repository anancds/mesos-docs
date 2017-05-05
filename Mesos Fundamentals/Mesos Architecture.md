### Mesos Architecture

![](http://mesos.apache.org/assets/img/documentation/architecture3.jpg)

上图描述了Mesos的主要组件：一个master daemon，用来管理集群中运行在每台机器上的agent daemon，还有Mesos Frameworks用来运行agent上的task。

Mesos的master通过Frameworks的resource offers可以支持细粒度的资源共享(CPU,RAM,...)。每个resouce offer包含如下列表：<agent ID, resource1: amount1, resource2: amount2, ...>(注意：这里关键字slave已经用agent代替了)。

Mesos master根据现有的策略，比如说fair sharing或者strict priority来决定给每个Framework分配多少资源。为了支持多样化的策略，master引入了模块化的架构，这样就可以通过插件的机制增加新的资源分配模块。

运行在mesos上的一个Framework由两个组件组成：scheduler和executor。scheduler启动后注册到master上，接收master发送的Resource Offer消息，来决定是否接受。executor进程在agent节点上启动，用来运行Framework的任务。

master给每个Framework分配好资源，Framework的scheduler拿到master提供给它的资源用来运行任务。当一个Framework接受了分配给它的资源，会传递给Mesos一个任务描述信息，然后Mesos就会在相应的agent运行这些任务。

### Example of resource offer

下图描述了如何调度一个Framework来运行任务。

![](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

* Agent1向master报告自己有4个cpu和4GB内存空闲，master就会调用分配策略的模块，得到的反馈就是需要提供所有的可用的资源给Framework1。
* Master发送一个 Resource Offer给Framework1，并告诉它agent1有多少可用资源。
* FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 agent1，一个 Task 需要<2个CPU，1 GB内存>，另外一个Task需要<1个CPU，2 GB内存>
* 最后master就把这些task发送给agent1，并且分配足够的资源给Framework executor，这是用来运行Framework的任务的。
接下来executor就会启动这两个任务。

因为还有1个cpu还有1GB内存没有分配，所以master的allocation module也许会把这些资源分配给FrameWork2。

此外，当任务运行完成或者有新的资源空闲，那么resource offer这个进程会重复运行。

Mesos提供了简单的接口，可供扩展，并且允许Framework独自演进，但是问题来了：在Mesos不知道Framework对资源有哪些限制的时候，怎么去满足Framework的资源限制。举例来说，当mesos不知道Framework需要的数据存储在哪些节点上，那么怎么让Framework获取到本地的数据呢。Mesos解决这个问题就是通过提供Framework拒绝资源的功能(reject offers)。如果mesos无法满足Framewor的资源限制，那么Framewor就会拒绝提供给他的资源。特别的，我们发现一个简单的调度策略称之为delay scheduling，就是Framework会等待一个有限的时间来获取存储数据的节点，最终产生最优的数据本地化。
