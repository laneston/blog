# 云端部署

kubernetes 的 [安装过程](https://blog.csdn.net/weixin_39177986/article/details/124807924) 就不在这里细说了，想要了解的可以查看连接中的博客。


## 查看 kuberetes 工具是否安装

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

## 默认镜像拉取方式

默认情况下, kubeadm 会从 k8s.gcr.io 仓库拉取镜像。如果请求的 Kubernetes 版本是 CI 标签 （例如 ci/latest），则使用 gcr.io/k8s-staging-ci-images。k8s.gcr.io 仓库需要使用外网，建议使用内网支持的镜像库。使用 dockerhub 下的 k8simage，这个域名下同步了不少谷歌镜像：

```
docker pull docker.io/k8simage/kube-apiserver:v1.22.10
docker pull docker.io/k8simage/kube-controller-manager:v1.22.10
docker pull docker.io/k8simage/kube-scheduler:v1.22.10
docker pull docker.io/k8simage/kube-proxy:v1.22.10
docker pull docker.io/k8simage/pause:3.5
docker pull docker.io/k8simage/etcd:3.5.0-0
docker pull docker.io/k8simage/coredns/coredns:v1.8.4
```

下载之后对镜像从新打标签:

```
docker tag docker.io/k8simage/kube-proxy-amd64:v1.11.3 k8s.gcr.io/kube-proxy-amd64:v1.11.3
```

当然，在配置文件中也可以通过修改 kubeadm 初始化配置进行指定下载镜像库，这是最简便的方式。

## 修改配置文件

获取默认配置文件并保存至本地

```
kubeadm config print init-defaults > kubeadm-defaults.yaml
```

可以通过修改配置文件 (kubeadm-defaults.yaml) 指定镜像下载链接，以下是本设备使用的一个配置文件内容，可用作与参考：

```
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
bootstrapTokens:
  - groups:
      - system:bootstrappers:kubeadm:default-node-token
    token: abcdef.0123456789abcdef
    ttl: 24h0m0s
    usages:
      - signing
      - authentication
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  taints: null
localAPIEndpoint:
  advertiseAddress: 172.16.16.6 (如果使用服务器部署，此处应是内网IP)
  bindPort: 6443
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
etcd:
  local:
    dataDir: /var/lib/etcd
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
apiServer:
  timeoutForControlPlane: 4m0s

certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}

imageRepository: docker.io/k8simage

#修改为当前使用版本号
kubernetesVersion: 1.22.9

scheduler: {}
---
#在 v1.22 中，如果用户没有设置cgroupDriver下的字段KubeletConfiguration， kubeadm将默认为systemd
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta3
#修改kubelet驱动
cgroupDriver: systemd
```
可以引用：
[kubeadm-defaults.yaml 配置文件样例](https://github.com/laneston/blog/blob/main/k8s/kubeadm-defaults.yaml)
并执行:

```
kubeadm init --config ./kubeadm-defaults.yaml
```


# 错误处理

## 问题01 ip_forward不为1

```
[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
```
查看ip_forward：
```
cat /proc/sys/net/ipv4/ip_forward
```
确实不为1

设置：
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

## 问题02 初始化提示The kubelet is not running

```
   Unfortunately, an error has occurred:
                timed out waiting for the condition

        This error is likely caused by:
                - The kubelet is not running
                - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

        If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                - 'systemctl status kubelet'
                - 'journalctl -xeu kubelet'

        Additionally, a control plane component may have crashed or exited when started by the container runtime.
        To troubleshoot, list all containers using your preferred container runtimes CLI.

        Here is one example how you may list all Kubernetes containers running in docker:
                - 'docker ps -a | grep kube | grep -v pause'
                Once you have found the failing container, you can inspect its logs with:
                - 'docker logs CONTAINERID'

error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher
```

配置文件中带有字段：
```
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
```

## 问题03 Unable to update cni config" err="no networks found in /etc/cni/net.d

```
Jul 02 10:48:45 VM-0-4-centos kubelet[178540]: I0702 10:48:45.239162  178540 cni.go:239] "Unable to update cni config" err="no networks found in /etc/cni/net.d"
Jul 02 10:48:47 VM-0-4-centos kubelet[178540]: E0702 10:48:47.270449  178540 kubelet.go:2376] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized"
```

解决方案是 [安装 flannel 组件](#flannel)





## 重置 kubeadm 配置

如果需要将 kubeadm 配置恢复到初始状态，可执行以下命令：

```
kubeadm reset

systemctl daemon-reload
systemctl restart kubelet.service
```

如果配置文件符合部署环境，则会看到如下打印，则说明初始化成功：

```
[init] Using Kubernetes version: v1.22.9
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local vm-16-6-centos] and IPs [10.96.0.1 172.16.16.6]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost vm-16-6-centos] and IPs [172.16.16.6 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost vm-16-6-centos] and IPs [172.16.16.6 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 7.502086 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node vm-16-6-centos as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node vm-16-6-centos as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.16.6:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:346df63942aaaae451e35c51990562ac9a543111c277b0a6ed6daef24a02f571 
```

# 安装 flannel 组件
<a id='flannel'></a>

部署文件路径如下：https://github.com/flannel-io/flannel/blob/master/Documentation/kube-flannel.yml

将文件下载至本地后可以通过以下命令进行安装：
```
kubectl apply -f kube-flannel.yml
```

此时可能会弹出以下错误警告信息：

```
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

原因：kubernetes master没有与本机绑定，集群初始化的时候没有绑定，此时设置在本机的环境变量即可解决问题。

解决办法：
```
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile
```

再执行：
```
kubectl apply -f kube-flannel.yml
```