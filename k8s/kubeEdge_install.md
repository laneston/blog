## kubernetes 与 kubeEdge 对应关系

|                        | Kubernetes 1.16 | Kubernetes 1.17 | Kubernetes 1.18 | Kubernetes 1.19 | Kubernetes 1.20 | Kubernetes 1.21 | Kubernetes 1.22 |
| ---------------------- | --------------- | --------------- | --------------- | --------------- | --------------- | --------------- | --------------- |
| KubeEdge 1.8           | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               |
| KubeEdge 1.9           | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               |
| KubeEdge 1.10          | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               |
| KubeEdge HEAD (master) | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               | ✓               |




## 为什么要部署kubeEdge

1. 边缘侧设备没有足够的资源运行一个完整的 Kubelet，Kubelet主要作用是获取最新的规范，确保各节点的 Pod 和容器在规范下运行；
2. 某些边缘侧设备是ARM架构，然而kubernetes不支持ARM架构。


1. KubeEdge 保留了 Kubernetes 的管理面，重新开发了节点 agent，大幅度优化让边缘组件资源占用更低很多；
2. KubeEdge 可以完美支持 ARM 架构和 x86 架构；
3. KubeEdge 有离线自治功能；
4. KubeEdge 丰富了应用和协议支持，目前已经支持和计划支持的有：MQTT、BlueTooth、OPC UA、Modbus等；
5. KubeEdge 通过底层优化的多路复用消息通道优化了云边的通信的性能。





