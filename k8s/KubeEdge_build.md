# 云端部署

kubeEdge 的 [安装过程](https://blog.csdn.net/weixin_39177986/article/details/124807924) 就不在这里细说了，想要了解的可以查看连接中的博客。

## 查看工具是否安装

```
yum list installed | grep kubelet
yum list installed | grep kubeadm
yum list installed | grep kubectl
```


执行 kubeadm init 命令时遇到以下问题：

```
# kubeadm init
[init] Using Kubernetes version: v1.24.0
[preflight] Running pre-flight checks
        [WARNING FileExisting-tc]: tc not found in system path
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR CRI]: container runtime is not running: output: time="2022-05-17T11:09:36+08:00" level=fatal msg="getting status of runtime: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService", error: exit status 1
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

打印信息中显示了1句警告和2句错误提示，我们先解决警告信息：

```
dnf install -y iproute-tc
```

## 容器运行时



容器不光是 Docker，还有其他容器，比如 CoreOS 的 rkt。为了保证容器生态的健康发展，保证不同容器之间能够兼容，包含 Docker、CoreOS、Google在内的若干公司共同成立了一个叫 Open Container Initiative（OCI） 的组织，其目是制定开放的容器规范。

runtime 是容器真正运行的地方，runtime 需要跟操作系统 kernel 紧密协作，为容器提供运行环境。

lxc、runc 和 rkt 是目前主流的三种容器 runtime。

- lxc 是 Linux 上老牌的容器 runtime，Docker 最初也是用 lxc 作为 runtime。
- runc 是 Docker 自己开发的容器 runtime，符合 oci 规范，也是现在 Docker 的默认 runtime。
- rkt 是 CoreOS 开发的容器 runtime，符合 oci 规范，因而能够运行 Docker 的容器。






# 边端部署

