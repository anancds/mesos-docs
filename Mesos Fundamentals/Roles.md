## Roles

在Mesos中，roles可以用来指定某种资源是专门为一个或者多个frameworks预留的。当资源提供给frameworks时，通过Roles可以提供一些各种各样的限制。一些关于roles的用例有：

* 提供特定的agent上全部的资源给特定的frameworks。
* 把一个集群分割成两个组织：组织A上使用的预留资源只能提供给那些注册到组织A的role上的frameworks(查看[reservation documentation](http://mesos.apache.org/documentation/latest/reservation/))。
* 确保一个framework创建的[persistent volumes](http://mesos.apache.org/documentation/latest/persistent-volume/)不会提供给不同注册时不时同一个role的framework。
* 表明一组frameworks相比别的组拥有高优先级(提供更多的资源)。
* 为同一个role的一个或者多个frameworks设置一个有保证的资源分配方案。


### Roles and access control

有两种方式可以控制一个framework可以注册成哪个role。
第一，可以用ACLs指定某些frameworks可以注册到某些roles。更多的信息，查看[authorization](http://mesos.apache.org/documentation/latest/authorization/)。
第二， 当Mesos的master服务起来时，通过**--roles**可以配置一个role的白名单。这个配置项指定以roles的列表，用逗号隔开。如果一旦指定了这个白名单，那么只有在这个名单里的roles才可以使用。如果想改变这个白名单，那么Mesos的master服务必须重启。注意，在Mesos的高可用部署模式下，必须非常小心的保证所有的Mesos master都配置了同一个白名单。

在Mesos0.26及更早的版本，必须同时配置ACLs和白名单，因为在这些版本中，任何没有出现在白名单中的role都不可以使用。

在Mesos0.27版本后，改变了上述行为：如果没有指定**--roles**，白名单允许使用任何一个role。因此在Mesos0.27版本中推荐的最佳实践就是用ACLs来定义哪些roles可以使用，因为**--roles**已经弃用了。

### Associating frameworks with roles

当一个framework想要注册到Mesos的master时，可以指定一个role，当然也可以不指定，这是个可选项。

作为一个开发人员，可以为自己写的framework指定一个role，通过**FrameworkInfo**里的**role**字段。

作为一个使用人员，普遍的可以在起framework时指定这个framework是哪一个role，怎么使用取决于你具体使用的framework的用户接口，比如Marathon通过**--mesos_flag**命令行参数。

### Multiple frameworks in the same role

多个frameworks可以使用同一个role。这样做也是有用的，比如：一个framework可以创建一个持久化卷(persistent volume),并写入数据。一旦写入持久化卷数据的task结束了，那么这个持久化卷就会提供给别的在同一个role里的framework。这么做给了第二个framework(consumer)机会起任务读取第一个framework(producer)产生的数据。

不管怎么样，配置多个frameworks使用同一个role需要非常的小心，因为所有的frameworks都可以访问这个role中预留的所有的资源。比如：如果一个framework存储了一些敏感的数据在持久化卷中，那么这个持久化卷有可能被同一个role中的别的frameworks访问到。相类似的，如果一个framework创建了一个持久化卷，同个role中的另外一个framework有可能偷这个持久化卷，并且用它起自己的任务。更广泛的来说，共享同一个role的多个frameworks需要相互协作，来保证合理的使用同一个role中的资源。

### Associating resources with roles

一个资源通过reservation(预留)的方式分配给role。资源可以静态的预留(当拥有这个资源的agent服务已经起来)，也可以是动态的：frameworks和操作者可以指定给定的role中的某个资源需要顺序的预留，更多信息，参考：[reservation](http://mesos.apache.org/documentation/latest/reservation/)。

### The default role

命名为\*的role是很特殊的，资源被分配给名字是\*的role，那么就不是预留的(unreserved)。简单的来说，当一个framework注册时没有指定role，那么它就会被分配给名字是\*的role。默认情况下，同一个agen上的所有的资源初始都是分配给默认的\*role的(在起agent服务时，通过命令行的**--default-role**可以改变这个默认名字)。

默认是\*的role的功能和不是默认的role是不一样的。比如：一些动态预留的资源可以从默认的\*role到不是默认的role中，但是如果不是默认的\*role，那么就不能从一个role到另外一个role(除去第一次不是预留的资源，比如，用[ /unreserve](http://mesos.apache.org/documentation/latest/endpoints/master/unreserve/)操作符的http端点)。相类似的在一些不是预留的资源上不能创建持久化卷。

### Invalid role

一个role的名字必须是一个有效的目录名称，所以不能是一下的形式：
* 空字符串
* .或者..
* 以-开头
* 包括任何斜杠，回退符，空白符

### Roles and resource allocation

默认情况下，Mesos master使用Dominant Resource Fairness(DRF)算法来分配资源，特别的，DRF算法的实现首先确认在所有的roles的主资源中(dominant resource)，哪个role是最公平共享的。在这个role中的所有的frameworks那么就会轮流的提供额外的资源。

资源分配的进程可以通过给roles设置[weights](http://mesos.apache.org/documentation/latest/weights/)来定制：一个权重设置为2的role是权重设置为1的role的公平共享资源的两倍。默认的，所有role的权重都是1。权重可以通过[ /weights](http://mesos.apache.org/documentation/latest/endpoints/master/weights/)操作符来配置，或者通过已经被弃用的Mesos master的命令行参数**--weights**来设置。

### Role vs. Principal

一个Principal表示一个和Mesos交互的实体。Principals和用户名非常相近。比如，frameworks在注册到master上时提供了一个Principal，并且操作者在使用HTTP端点的时候也会提供一个Principal。一个实体可能需要用Principal[authenticate](http://mesos.apache.org/documentation/latest/authentication/)，为了证明同一性。并且Principal也有可能用来[authorize](http://mesos.apache.org/documentation/latest/authorization/)实体的执行动作，比如：[ resource reservation ](http://mesos.apache.org/documentation/latest/reservation/)，还有[persistent volume]()的创建和销毁。

另外一方面，就像上面描述的，Roles用来排他的关联资源和frameworks。
