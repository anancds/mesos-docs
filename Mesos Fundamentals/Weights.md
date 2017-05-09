## Weights

在Mesos中，Weights是用来控制不同的roles之间的集群资源的相对份额。

在Mesos0.28及以前版本中，只能通过Mesos master的命令行参数**--weights**来配置，如果一个role没有指定具体的权重，那么默认就是1.0。权重一旦设置不可就不能改变，除非重启Mesos masters。

Mesos1.0后，就包含了一个[/weights]的endpoint,可以在运行时改变权重，命令行设置权重就废弃了。
