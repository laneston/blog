

<div align="center"><img src="https://github.com/laneston/Pictures/blob/master/HUAWEI_repository/HUAWEI_repository.png"></div>

## 备份配置文件

```
cp -a /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
mv CentOS-AppStream.repo CentOS-AppStream.repo.bak
```

## 清除原有源缓存并生成新源缓存

```
yum clean all
yum makecache
```
