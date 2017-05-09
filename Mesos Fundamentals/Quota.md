
### Quota

当在Mesos上运行多个frameworks时，有一个问题是，默认的分配策略[wDRF allocator](https://people.eecs.berkeley.edu/~alig/papers/drf.pdf)可能即使超过了frameworks的公平共享份额，也会继续提供资源给这些frameworks(比如当没有其他的frameworks接受这些资源时)。目前没有一种机制可以保存这些资源用于集群未来消费利用。即使[dynamic reservations](http://mesos.apache.org/documentation/latest/reservation/)允许operators和frameworks在特殊的agent上动态的预留资源，但是还是没有解决上面的问题，因为单个agent有可能宕掉。

Mesos0.27引入了配额(**quotas**)的支持，配额的机制是保证一个role可以接收到最少的资源。配置指定了一个role可以接收到的最少的资源(除非集群的全部资源少于配置的配额资源，那么这就是一个配置错误)。当然了，更加Mesos的DRF资源分配规则，role可以接收到比配额更多的资源。

Quotas在某些方面和[reserved resources](http://mesos.apache.org/documentation/latest/reservation/)相似，但是也有一些不同点。最重要的一点就是，配额资源不会绑定到某个特殊的agent，也就是说，配额保证了role可以接收到集群中的任何机器上的一定的资源。但是预留就是用来在特殊的agent上分配具体的资源给指定的role。Quotas只能用operators配置，通过HTTP endpoint。动态预留可以通过frameworks配置。

注意，预留资源是为了满足一个role的配额。比如说，一个role已经分配了4 CPUs的配额，当然还有某个特殊agent上的2 CPUs的预留。那么这个role就能保证最少4 CPUs，而不是6 CPUs。

### Terminology

对于本文档来说，Operator指代一个人，一个工具，或者管理Mesos集群的脚本。

在计算机科学中，配额经常指代如下的某种情况：
* 最少保证
* 最大限制
* 上面两者都有

在Mesos中，配额是一个role可以依赖的资源分配保证，换句话说，是一个role能够接收到的最小份额。

### Motivation and Limitations

为了更好的理解，考虑在Mesos中如下的场景，这些场景有些支持配额，有些不是。

#### Scenario 1: Greedy Framework

集群中有两个frameworks，每个Framework运行在各自具体一定权重的role上，Framework fA运行在roleA上，Framework fB运行在roleB上。然后集群中就只有一种资源：100 CPUs。fA消耗了10 CPUs，然后空闲下来(拒绝了资源邀约)，同时，fB是贪婪的，接受了它能收到的所有的资源邀约，拿到了剩下的90 CPUs。如果没有配额，虽然fA的公平份额的50 CPUs，但是直到fB的任务终止，否则不会利用额外的40 CPUs。

#### Scenario 2: Resources for a new Framework

Framework fB是集群中唯一一个Framework，并且是贪婪的，那么它就会使用所有的可用资源：100 CPUs。如果roleA新加入一个Framework fA，那么这个fA并不会接受到它的公平份额，即50 CPUs。除非fB的任务终止。
为了处理这样的场景，配额并不是一个很好的解决方案，因为这里配额是在fB运行后并且使用了所有的资源才设置的。
这样的场景需要保持一个资源池，这个资源池不会再资源邀约中，或者抢占(preemption)运行中的任务
