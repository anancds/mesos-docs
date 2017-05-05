### 共同点

*


### 不同点

* 语言：yarn是java，mesos是c++。
* 隔离：mesos用linux container groups，yarn用unix进程隔离，mesos隔离性更强一点。
* 粒度：yarn是粗粒度的，控制的container，都是预先分配好的

* yarn的调度是以数据为中心，而mesos的调度是以资源为中心。
