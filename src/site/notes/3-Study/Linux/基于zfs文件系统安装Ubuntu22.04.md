---
{"dg-publish":true,"permalink":"/3-Study/Linux/基于zfs文件系统安装Ubuntu22.04/"}
---


# 硬件清单
	- 120G SSD X1
	- 500G SSD X1
	- 1T 机械硬盘 x1

# 安装流程
1. 首先，从 Ubuntu 官网下载 Ubuntu 22.04 的 ISO 镜像文件，并制作一个启动盘。你可以使用 Rufus、Etcher 等工具将镜像文件写入到 U 盘或其他可启动设备上
2. 将启动盘插入你的计算机，并重启。在启动过程中，按下相应的按键（通常是 F12、F10 或 ESC，取决于你的硬件）进入 BIOS/UEFI 设置，将启动顺序设置为首先从启动盘启动。
3. 当 Ubuntu 安装界面出现后，选择 "Try Ubuntu" 或 "Install Ubuntu"。这将载入 Ubuntu 的实时环境，你可以在这里进行安装操作。
4. 在实时环境中，双击 "Install Ubuntu" 图标开始安装过程。选择你的语言和键盘布局，然后点击 "Continue"。
5. 在 "Updates and other software" 页面，选择 "Normal installation" 或 "Minimal installation"，根据你的需求安装软件。确保勾选 "Install third-party software for graphics and Wi-Fi hardware, Flash, MP3 and other media" 以获得更好的硬件支持。点击 "Continue"。 
6. 在 "Installation type" 页面，选择 "Something else"，然后点击 "Continue"。这将进入手动分区编辑器。 
7. 在手动分区编辑器中，你需要为每块硬盘创建分区。以下是一个建议的分区方案：
    1. 120G SSD：
        1. 创建一个 EFI 系统分区 (ESP)，大小为 500MB，分区类型选择 "EFI System Partition"，挂载点选择 "/boot/efi"。
        2. 创建一个用于 ZFS 存储池的分区，使用剩余的空间，分区类型选择 "physical volume for encryption"。
    2. 500G SSD：
        1. 创建一个用于 ZFS 存储池的分区，使用整个硬盘空间，分区类型选择 "physical volume for encryption"。
    3. 1T 机械硬盘：
        1. 创建一个用于 ZFS 存储池的分区，使用整个硬盘空间，分区类型选择 "physical volume for encryption"。
8. 确认分区设置无误后，点击 "Install Now"。
9. 在弹出的确认对话框中，检查分区信息，然后点击 "Continue"。
10. 完成剩余的安装步骤，如选择时区、输入个人信息等。
11. 在安装过程中，安装程序将自动为你创建 ZFS 存储池和文件系统。但是，如果你希望对 ZFS 进行更详细的配置，你可以在安装完成后手动创建和管理 ZFS 存储池和文件系统。
12. 使用`sudo zfs list`命令查看ZFS存储池下的文件系统。这个命令将显示所有ZFS数据集及其属性，包括挂载点。
13. 使用`sudo zpool status`命令查看现有的`rpool`存储池的配置
14. 将新硬盘添加到zpool池`sudo zpool add rpool /dev/sdc
15. 使用以下命令创建快照，将`<filesystem>`替换为你在上一步找到的文件系统名称，将`<snapshot_name>`替换为你想要为快照命名的名称：

```
sudo zfs snapshot <filesystem>@<snapshot_name>
sudo zfs snapshot rpool/ROOT/ubuntu_XXXX@my_snapshot
sudo zfs snapshot -r rpool/ROOT/ubuntu_XXXX@my_snapshot   参考cp -r
```

16. 要查看已创建的快照列表，运行以下命令：

```
zfs list -t snapshot
```

17. 要恢复到先前的快照，你可以使用以下命令，将`<filesystem>`替换为你的文件系统名称，将`<snapshot_name>`替换为你要恢复到的快照名称：

```
sudo zfs rollback <filesystem>@<snapshot_name>
sudo zfs rollback rpool/ROOT/ubuntu_XXXX@my_snapshot
```

18. 快照脚本
```
#!/bin/bash 
# 获取当前日期和时间 
current_date=$(date +"%Y-%m-%d_%H-%M-%S") 
# 为所有ZFS文件系统创建快照 
zfs list -H -o name | while read -r filesystem; do 
sudo zfs snapshot "${filesystem}@${current_date}" 
done
```
