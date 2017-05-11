### 共同点

*


### 不同点

* 语言：yarn是java，mesos是c++。
* 隔离：mesos用linux container groups，yarn用unix进程隔离，mesos隔离性更强一点，但是可能会有一些额外的开销。
* 粒度：yarn是粗粒度的，控制的container，都是预先分配好的

* yarn的调度是以数据为中心，而mesos的调度是以资源为中心。

### 编程语言

* yarn是java
* Mesos是c++
在语言上Mesos可能会有一定的优势。

### 架构

* Mesos是二级调度架构，也就是说master和agent

### 管理资源类型

* yarn现在只管理Memory和CPUs。
* Mesos是：CPUs，GPU，disk，memory，ports。

### 资源分配

### 资源分配粒度

* yarn是粗粒度的，分配的单位是container，资源一般是固定的，预先分配好，所以会有一定的预留，造成很大的浪费。
* Mesos是细粒度的，也就是说Mesos的master向Framework发出资源邀约后，Framework的scheduler可以选择接受全部的资源，也可以接受部分的资源，或者拒绝资源。

### 资源抢占

### 资源隔离

### 扩展性

* yarn是针对hadoop生态系统的。
* Mesos是数据中心的内核，只要去实现它提供的一些Framework的接口就可以了，扩展性比较好。

### 应用场景
