# 安装包

Docker 安装其实分为三个组件 containier.io，docker-ce, docker-cli [下载链接](https://download.docker.com/linux/centos/8/x86_64/stable/Packages/)

1. containerd.io - 从本质上讲，此守护进程为系统 API 接口，目的是 Docker 与操作系统解耦，同时为非Docker容器管理器提供容器服务（例如LXC）；

2. docker-ce-cli - CLI 工具来控制守护程序，如果要控制远程 Docker 守护程序，可以自行安装它们；

3. docker-ce - Docker守护进程是完成所有管理工作的部分，在 Linux 上需要另外两个协同。


## 卸载

如果已经安装了 docker 需要先进行卸载操作：

```
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine 
```

## 依赖

进行依赖安装：

```
sudo yum install yum-utils device-mapper-persistent-data lvm2
```

## 安装

安装组件，安装前需要将[下载链接](https://download.docker.com/linux/centos/8/x86_64/stable/Packages/)中的安装包都下载到本地。

```
yum install ./containerd.io-1.5.11-3.1.el8.x86_64.rpm
yum install ./docker-ce-20.10.9-3.el8.x86_64.rpm ./docker-ce-cli-20.10.9-3.el8.x86_64.rpm ./docker-scan-plugin-0.17.0-3.el8.x86_64.rpm ./docker-ce-rootless-extras-20.10.14-3.el8.x86_64.rpm ./docker-compose-plugin-2.3.3-3.el8.x86_64.rpm 
```

## 启动

```
systemctl start docker
docker version
```

查询版本信息，如若有打印信息，则说明安装成功。

# 镜像


## 拉取最新镜像


```
[root@VM-0-4-centos ~]# docker pull ubuntu:latest
latest: Pulling from library/ubuntu
4d32b49e2995: Pull complete 
Digest: sha256:bea6d19168bbfd6af8d77c2cc3c572114eb5d113e6f422573c93cb605a0e2ffb
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```


## 查看镜像

```
docker images
```

## 进入容器

```
docker run -it ubuntu
```

## 退出镜像

```
exit
```