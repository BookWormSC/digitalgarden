---
{"dg-publish":true,"permalink":"/3-Study/Linux/使用zfs文件系统拓展RockyLinux 9/"}
---

# 目标
```
在120G SSD硬盘上安装RockyLinux 9，分区如下：
/dev/sdb1    /boot   1G
/dev/sdb2    swap    32G
/dev/sdb3    /       30G
/dev/sdb4            8G    # for zil   zfs写缓存
/dev/sdb5            49G   # for l2arc zfs读缓存

1T HDD硬盘：
/dev/sdc              1T   # zfs主要数据盘，后面用于迁移 /home , /var , /opt
```



# 1，安装zfs
[RHEL-based distro — OpenZFS documentation](https://openzfs.github.io/openzfs-docs/Getting%20Started/RHEL-based%20distro/index.html)
```
# 安装zfs源
dnf install -y https://zfsonlinux.org/epel/zfs-release-2-2$(rpm --eval "%{dist}").noarch.rpm

# 安装epel源
dnf install -y epel-release

# 安装dev内核
dnf install -y kernel-devel

# 禁用zfs源
## zfs源是dkms(动态内核模块支持)---ZFS on Linux for EL 7 - dkms
dnf config-manager --disable zfs

# 启用zfs-kmod源
## kmod是内核模块
dnf config-manager --enable zfs-kmod

# 安装zfs主程序
dnf install -y zfs

# 加载zfs内核模块
modprobe zfs

# 安装分区工具
dnf install -y gdisk dosfstools
```
# 2，配置zfs
##  1，始终在启动时加载OpenZFS模块
默认情况下，当检测到 ZFS 池时，会自动加载 `OpenZFS` 内核模块。如果您希望`**始终在启动时加载**`模块，则必须创建一个 `/etc/modules-load.d/zfs.conf` 文件：
```
# 在启动时自动加载zfs模块
echo zfs >/etc/modules-load.d/zfs.conf
```
## 2，开机自启zfs服务
```
# 开机自启zfs服务
systemctl enable --now zfs-import-scan.service zfs-mount zfs-import.target zfs-zed zfs.target

# 查看zfs-import-scan服务状态
systemctl status zfs-import-scan
```
## 3，创建ZFS文件系统
### 1，创建一个ZFS存储池
```
# 创建一个存储池
## zpool create -f 存储池名称 磁盘1 磁盘2 磁盘3 磁盘4
zpool create -f zfspool /dev/sdc
zpool add zfspool cache /dev/sdb5
zpool add zfspool log /dev/sdb4
```
### 2，创建新的ZFS文件系统

```
# 创建用于挂载/home,/var,/opt的文件系统
zfs create zfspool/home
zfs create zfspool/var
zfs create zfspool/opt
```
### 3，备份系统目录

```
mv /home /home_old
mv /var /var_old
mv /opt /opt_old
```
### 4，将新的ZFS文件系统挂载到根目录下

```
zfs set mountpoint=/home zfspool/home
zfs set mountpoint=/var zfspool/var
zfs set mountpoint=/opt zfspool/opt
```
### 5，旧的目录的内容复制到新的目录

```
cp -a /home_old/* /home/
cp -a /var_old/* /var/
cp -a /opt_old/* /opt/
```
