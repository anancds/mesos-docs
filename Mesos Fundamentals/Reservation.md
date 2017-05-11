### Reservation

Mesos提供了一种在具体slave上预留资源的机制。这个概念最先是在Mesos0.14版本中的静态预留资源中引入的，并且允许operators在slave服务起来时指定预留资源。这个在Mesos0.23版本后扩展成动态预留，这样可以使operators和授权的frameworks动态预留集群中的资源。

这两种预留方式，都是针对同一个role上的。

### Static Reservation

一个operator可以配置一个role上的资源预留，通过指定**--resources**标志。比如说，假设我们再一台slave上有12 CPUs和6144 MB内存，然后我们想给role **ads** 预留8 CPUs和4096 MB内存，那么我们在起slave服务时应该这样：

    $ mesos-slave \
          --master=<ip>:<port> \
          --resources="cpus:4;mem:2048;cpus(ads):8;mem(ads):4096"

这样我们就在这个slave上为role **ads** 预留了8 CPUs和4096 MB内存。

**警告**： 为了可以修改静态的预留方式，operator必须指定**--resource**标识并且重启slave服务。

**注意**：这个特性是向后兼容的。推荐的使用方式是通过**--resources**标识来指定一台slave上不是预留的所有的可用资源，然后通过master HTTP endpoints来管理资源动态预留。

### Dynamic Reservation

就像上面的静态资源预留提到的，通过**--resources**标识来指定预留资源，并且是静态的。也就是说，静态预留的资源就不能给别的role了，也不能把资源改成非预留。动态资源预留就可以让operators和授权的frameworks在slave服务起来后预留资源，或者改成非预留。

默认的，frameworks和operator可以给任何的role都预留资源，也可以解除预留的资源。[Authorization](http://mesos.apache.org/documentation/latest/authorization/)可以限制哪些role可以预留资源，哪些role可以解除预留资源。因为这些操作需要授权，所以frameworks或者operator需要提供一个**principal**来标识它。为了使用授权了的资源预留和解除资源预留，Mesos master必须配置合适的ACLs，更多信息，参考[authorization documentation](http://mesos.apache.org/documentation/latest/authorization/)。

* 当有资源邀约的时候，frameworks通过**acceptOffers** API 来反馈这个请求，主要反馈的是**Offer::Operation::Reserve** 和 **Offer::Operation::Unreserve** 消息体。

*  **/reserve** 和 **/unreserve** HTTP endpoints允许operators通过master动态资源分配。
