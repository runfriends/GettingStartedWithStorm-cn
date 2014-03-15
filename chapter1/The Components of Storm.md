**Storm组件**

对于一个Storm集群，若干节点组成一个连续运行的主节点。

在Storm集群中，有两类节点：主节点*master node*和工作节点*worker nodes*。主节点运行着一个叫做*Nimbus*的守护进程。这个守护进程负责在集群中分发代码，为工作节点分配任务，并监控故障。Supervisor守护进程作为拓扑的一部分运行在工作节点上。一个Storm拓扑结构在不同的机器上运行着众多的工作节点。

因为Storm在Zookeeper或本地磁盘上维持所有的集群状态，守护进程就可以是无状态的而且可能失效或重启而不会影响整个系统的健康（见图1-2）
![图1-2 Storm集群的组件][1]


  [1]: https://github.com/runfriends/GettingStartedWithStorm-cn/blob/master/chapter1/Figure%201-2.%20Components%20of%20a%20Storm%20cluster.png
