### Mesos是什麽
Mesos诞生于UC Berkeley的一个研究项目，现已成为Apache中的项目，当前有一些公司使用Mesos管理集群资源，比如Twitter。

总体上看，Mesos是一个master/slave结构，其中，master是非常轻量级的，仅保存了framework（各种计算框架称为framework）和mesos slave的一些状态，而这些状态很容易通过framework和slave重新注册而重构，因而很容易使用了zookeeper解决mesos master的单点故障问题。
Mesos master实际上是一个全局资源调度器，采用某种策略将某个slave上的空闲资源分配给某一个framework，各种framework通过自己的调度器向Mesos master注册，以接入到Mesos中；而Mesos slave主要功能是汇报任务的状态和启动各个framework的executor（比如Hadoop的excutor就是TaskTracker）。

### Mesos中的基本术语解释
- Mesos agent nodes （slave nodes）： 接受master指令，执行task的节点。
- Master nodes：发送task给slave node
- ZooKeeper 在多个maser nodes中选举active的那一个。
- Framework： 例如Marathon 可以类比hadoop2中Yarn是资源管理平台 MapReduce是一个计算框架。

### 参考
- http://dongxicheng.org/apache-mesos/meso-architecture/
