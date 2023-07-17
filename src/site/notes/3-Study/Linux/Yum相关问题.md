---
{"dg-publish":true,"permalink":"/3-Study/Linux/Yum相关问题/"}
---


## 1.执行yum相关的命令的时候报下面的错误

![](https://img2020.cnblogs.com/blog/1901802/202005/1901802-20200513134811571-34981284.png)

##  2.解决方法（重新构建rpm数据库）
 `cd /var/lib/rpm`
 `rm -rf \_\_db\*
 `rpm --rebuilddb

##  3.重装Yum
1、删除现有Python
[root@Xxxx ~]# rpm -qa|grep python|xargs rpm -ev --allmatches --nodeps ##强制删除已安装程序及其关联

[root@Xxxx~]# whereis python |xargs rm -frv ##删除所有残余文件 ##xargs，允许你对输出执行其他某些命令

[root@Xxxx~]# whereis python ##验证删除，返回无结果

2、删除现有的yum
[root@Xxxx ~]# rpm -qa|grep yum|xargs rpm -ev --allmatches --nodeps

[root@Xxxx~]# whereis yum |xargs rm -frv

3、下载对应的Python2和Yum的rpm包**
```
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-chardet-2.2.1-3.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-devel-2.7.5-89.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-iniparse-0.4-9.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-libs-2.7.5-89.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-pycurl-7.19.0-19.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-setuptools-0.9.8-7.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-urlgrabber-3.10-10.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/rpm-python-4.11.3-45.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/libxml2-python-2.9.1-6.el7.5.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-2.7.5-89.el7.x86_64.rpm

wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-168.el7.centos.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-aliases-1.1.31-54.el7_8.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-protectbase-1.1.31-54.el7_8.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-updateonboot-1.1.31-54.el7_8.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-utils-1.1.31-54.el7_8.noarch.rpm


```

4，安装下载的rpm包
rpm -ivh --force \*.rpm --nodeps

5，查看安装是否成功

rpm -qa yum

6. 导入证书

rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7

7.添加阿里的源

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

8.清除缓存 生成新的缓存

yum clean all

yum makecache

9,更新
yum -y update：升级所有包同时也升级软件和系统内核；​
yum -y upgrade：只升级所有包，不升级软件和系统内核