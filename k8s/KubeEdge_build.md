# 云端部署

kubeEdge 的 [安装过程](https://blog.csdn.net/weixin_39177986/article/details/124807924) 就不在这里细说了，想要了解的可以查看连接中的博客。

## 查看工具是否安装

```
yum list installed | grep kubelet
yum list installed | grep kubeadm
yum list installed | grep kubectl
```


## 容器运行时


容器不光是 Docker，还有其他容器，比如 CoreOS 的 rkt。为了保证容器生态的健康发展，保证不同容器之间能够兼容，包含 Docker、CoreOS、Google在内的若干公司共同成立了一个叫 Open Container Initiative（OCI） 的组织，其目是制定开放的容器规范。

runtime 是容器真正运行的地方，runtime 需要跟操作系统 kernel 紧密协作，为容器提供运行环境。

lxc、runc 和 rkt 是目前主流的三种容器 runtime。

- lxc 是 Linux 上老牌的容器 runtime，Docker 最初也是用 lxc 作为 runtime。
- runc 是 Docker 自己开发的容器 runtime，符合 oci 规范，也是现在 Docker 的默认 runtime。
- rkt 是 CoreOS 开发的容器 runtime，符合 oci 规范，因而能够运行 Docker 的容器。

# 镜像拉取

## 查看需拉取镜像列表

kubeadm config images list

```
I0607 14:51:32.871878  275642 version.go:255] remote version is much newer: v1.24.1; falling back to: stable-1.22
k8s.gcr.io/kube-apiserver:v1.22.10
k8s.gcr.io/kube-controller-manager:v1.22.10
k8s.gcr.io/kube-scheduler:v1.22.10
k8s.gcr.io/kube-proxy:v1.22.10
k8s.gcr.io/pause:3.5
k8s.gcr.io/etcd:3.5.0-0
k8s.gcr.io/coredns/coredns:v1.8.4
```

## 镜像拉取


默认情况下, kubeadm 会从 k8s.gcr.io 仓库拉取镜像。如果请求的 Kubernetes 版本是 CI 标签 （例如 ci/latest），则使用 gcr.io/k8s-staging-ci-images。k8s.gcr.io 仓库需要使用外网，建议使用内网支持的镜像库。

使用 dockerhub 下的 k8simage，这个域名下同步了不少谷歌镜像：


```
docker pull docker.io/k8simage/kube-apiserver:v1.22.10
docker pull docker.io/k8simage/kube-controller-manager:v1.22.10
docker pull docker.io/k8simage/kube-scheduler:v1.22.10
docker pull docker.io/k8simage/kube-proxy:v1.22.10
docker pull docker.io/k8simage/pause:3.5
docker pull docker.io/k8simage/etcd:3.5.0-0
docker pull docker.io/k8simage/coredns/coredns:v1.8.4
```



来进行下载,下载之后对镜像从新打标签:

```
docker tag docker.io/k8simage/kube-proxy-amd64:v1.11.3 k8s.gcr.io/kube-proxy-amd64:v1.11.3
```

## 通过配置文件进行修改


可以通过修改配置文件指定镜像下载链接


```
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 106.52.179.147
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: VM-0-4-centos 
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: docker.io/k8simage
kind: ClusterConfiguration
kubernetesVersion: 1.22.9
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```





# 边端部署



