## 什么是flannel

Flannel是CoreOS团队针对Kubernetes设计的一个网络规划服务，运行在一个网上的网（应用层网络），并不依靠ip地址来传递消息，而是采用一种映射机制，把ip地址和identifiers做映射来资源定位。 简单来说，它的功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址。

Flannel github：https://github.com/coreos/flannel




## 为什么使用flannel

在默认的Docker配置中，每个节点上的Docker服务会分别负责所在节点容器的IP分配。这样导致的一个问题是，不同节点上容器可能获得相同的内外IP地址。

Flannel的设计目的就是为集群中的所有节点重新规划IP地址的使用规则，从而使得不同节点上的容器能够获得“同属一个内网”且”不重复的”IP地址，并让属于不同节点上的容器能够直接通过内网IP通信。

## 如何实现flannel

Flannel实质上是一种“覆盖网络(overlay network)”，也就是将TCP数据包装在另一种网络包里面进行路由转发和通信，目前已经支持UDP、VxLAN、AWS VPC和GCE路由等数据转发方式，默认的节点间数据通信方式是UDP转发。

