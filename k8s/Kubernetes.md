# KubeEdge 保姆式部署攻略

## 环境配置

### 禁用开机启动防火墙

```
systemctl status firewalld.service //查看防火墙状态
systemctl disable firewalld.service //失能防火墙
```

### 禁用SELinux

/etc/selinux/config
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

### 关闭系统Swap（交换空间）

典型的计算机系统有两种类型的存储器。一种称为RAM（随机存取存储器）的易失性存储器，由程序主动使用，而第二种类型是称为HD（硬盘）的非易失性存储器，通常用于长期存储。

内核的内存管理单元（MMU）从硬盘中分割出一部分内存，并将其借给RAM，以防出现异常峰值。例如，如果一个系统已经有4GB的RAM，MMU可以从HD中分割出额外的2GB，并将其伪装成总共6GB。这种6GB内存也称为虚拟内存。MMU有内置的启发式算法来确定哪些内容是实际RAM的一部分，哪些内容应该是交换空间的一部分，并不断刷新它。应该注意的是，硬盘速度较慢，访问RAM的速度较快。因此，访问交换空间上的内容具有不确定性。

**查看交换分区**


```
# free -m
              total        used        free      shared  buff/cache   available
Mem:           1817         266         819           1         731        1404
Swap:             0           0           0
```

例子中系统所用的Swap为0，所以无需关闭。

**临时关闭**

```
swapoff -a
```

**永久关闭**


查看交换空间信息：

/etc/fstab 
```
UUID=078770b7-9017-4119-8863-5545673dc1f5 /                       ext4    defaults        1 1
UUID=f3d7d9a2-7e45-4165-a93c-a8e5a3286e49 /boot                   ext4    defaults        1 2
UUID=A5E9-3050          /boot/efi               vfat    umask=0077,shortname=winnt 0 2
UUID=07af0edf-b4ba-4592-bc02-299d156f173a /home                   ext4    defaults        1 2
#UUID=9a127488-c05d-4ce1-987e-d37e19dc4d8c none                    swap    defaults        0 0
```

注释响应的选项
/etc/default/grub
```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=false
GRUB_CMDLINE_LINUX_DEFAULT=' '
GRUB_TERMINAL_OUTPUT="console"
#GRUB_CMDLINE_LINUX="crashkernel=auto resume=UUID=9a127488-c05d-4ce1-987e-d37e19dc4d8c rhgb quiet"
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

linux 修改开机启动参数后，使用grub2-mkconfig 命令使得参数设置有效:

```
grub2-mkconfig  -o /boot/grub2/grub.cf
```




## 更换镜像源

[更换华为源 - github](https://github.com/laneston/blog/blob/main/linuxSystem/HUAWEI_repository.md)


## 时间同步

CentOS 8 系统做了不少更新，例如 nftables 代替iptables, dnf 代替yum成为默认包管理工具。这不，许多人发现CentOS 7 熟悉的 ntpdate 命令没有了，也不能用 yum 安装上，同步时间顿时成了一个难题。Chrony是一个开源软件，能用来于时钟服务器（NTP）同步，从而保持系统时间精确。


```
yum install -y chrony
```

vim /etc/chrony.conf
```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
##pool 2.centos.pool.ntp.org iburst

server 210.72.145.44 iburst
server ntp.aliyun.com iburst
```


重启服务
```
systemctl restart chronyd.service
```

进行时间同步
```
chronyc sources -v
```

## 安装流量控制器

```
yum install -y iproute-tc
```



## 安装 POD 网络附加组件

你必须部署一个基于 Pod 网络插件的 容器网络接口 (CNI)，以便你的 Pod 可以相互通信。 在安装网络之前，集群 DNS (CoreDNS) 将不会启动。

选用的组件为 falnnel






## 修改 docker cgroup 为 systemd


参考文章 [docker 与 systemd](https://github.com/laneston/blog/blob/main/linuxSystem/cgroup_.md)



## 允许 iptables 检查桥接流量

确保 br_netfilter 模块被加载。这一操作可以通过运行 lsmod | grep br_netfilter 来完成。若要显式加载该模块，可执行 sudo modprobe br_netfilter。

为了让你的 Linux 节点上的 iptables 能够正确地查看桥接流量，你需要确保在你的 sysctl 配置中将 net.bridge.bridge-nf-call-iptables 设置为 1。例如：

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```
sudo sysctl --system
```

## 开放端口

| 协议  | 方向  |  端口范围   | 目的                    |          使用者          |
| :---: | :---: | :---------: | :---------------------- | :----------------------: |
|  TCP  | 入站  |    6443     | Kubernetes API server   |           所有           |
|  TCP  | 入站  |  2379-2380  | etcd server client      | API	kube-apiserver, etcd |
|  TCP  | 入站  |    10250    | Kubelet API             |       自身, 控制面       |
|  TCP  | 入站  |    10259    | kube-scheduler          |           自身           |
|  TCP  | 入站  |    10257    | kube-controller-manager |           自身           |
|  TCP  | 入站  |    10250    | Kubelet API             |       自身, 控制面       |
|  TCP  | 入站  | 30000-32767 | NodePort Services       |           所有           |


## docker安装

| Kubernetes    版本 |    Docker 版本兼容情况     |
| :----------------: | :------------------------: |
|       1.21.x       | v20.10.2   版本以上不兼容  |
|       1.22.x       | v20.10.2    版本以上不兼容 |
|       1.23.x       | v20.10.7    版本以上不兼容 |




关于docker的安装可以参考我写的以下文档：

[github 文档](https://github.com/laneston/blog/blob/main/container/dorcker_install.md)

[csdn 文档](https://blog.csdn.net/weixin_39177986/article/details/124027595)



------------------------------

## 安装 kubectl

以下是单独安装 kubectl 的方式，可以在重装时使用。

Kubernetes 命令行工具，kubectl，使得你可以对 Kubernetes 集群运行命令。 你可以使用 kubectl 来部署应用、监测和管理集群资源以及查看日志。

<div align="center"><img src="https://github.com/laneston/Pictures/blob/master/Post-KubeEdge/kubectl_install.png"></div>


```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO https://dl.k8s.io/release/v1.23.0/bin/linux/amd64/kubectl
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
kubectl version --client --output=yaml
```

可以参考 [在 Linux 系统中安装并设置 kubectl](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl-linux/) 文档进行其他方式的安装设置方式。


---------------------------------


## 安装 kubeadm kubectl kubelet

1. 如果需要进行备份，则 **备份/etc/yum.repos.d/kubernetes.repo文件:**

```
cp /etc/yum.repos.d/kubernetes.repo /etc/yum.repos.d/kubernetes.repo.bak
```

2. 修改/etc/yum.repos.d/kubernetes.repo文件：

华为源，比较老旧，不推荐。
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://repo.huaweicloud.com/kubernetes/yum/repos/kubernetes-el7-$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://repo.huaweicloud.com/kubernetes/yum/doc/yum-key.gpg https://repo.huaweicloud.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

阿里源，推荐使用。
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

**提示 若您使用的yum中变量 $basearch 无法解析, 请把第二步配置文件中的$basearch修改为相应系统架构(aarch64/armhfp/ppc64le/s390x/x86_64).**


3. SELinux运行模式切换为宽容模式


```
setenforce 0
```

4. 更新索引文件



```
yum install -y kubelet-1.22.9 kubeadm-1.22.9 kubectl-1.22.9 --disableexcludes=kubernetes
systemctl enable kubelet
systemctl start kubelet
```


