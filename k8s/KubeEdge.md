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

## docker安装

[github 文档](https://github.com/laneston/blog/blob/main/container/dorcker_install.md)

[csdn 文档](https://blog.csdn.net/weixin_39177986/article/details/124027595)
