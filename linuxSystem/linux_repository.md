

# centos

<div align="center"><img src="https://github.com/laneston/Pictures/blob/master/HUAWEI_repository/HUAWEI_repository.png"></div>

## 备份配置文件

```
cd /etc/yum.repos.d
mv CentOS-Linux-BaseOS.repo CentOS-Linux-BaseOS.repo.bak
mv CentOS-Linux-AppStream.repo CentOS-Linux-AppStream.repo.bak
```

## 下载新源

华为源
```
wget -O /etc/yum.repos.d/CentOS-Base.repo https://repo.huaweicloud.com/repository/conf/CentOS-8-reg.repo
```

阿里源
```
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo

or

curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
```

## 清除原有源缓存并生成新源缓存

```
yum clean all
yum makecache
```


# ubuntu

/etc/apt/sources.list

```
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

# 镜像源重复

```
Repository extras is listed more than once in the configuration
```

进入 /etc/yum.repos.d 并删除对应的镜像源即可。