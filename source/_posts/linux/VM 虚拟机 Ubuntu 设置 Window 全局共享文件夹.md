---
title: VM 虚拟机设置 Windows 共享文件夹
date: 2022-02-11 14:46:30
tags: [linux, vm-ware, ubuntu, share-dir]
---

### VM 虚拟机设置共享文件夹

#### 1、VM ware 自带共享文件夹功能 

​		要使用此功能，一定要先安装 `VMware tools` 安装完毕一定要重启虚拟机。

- **打开 虚拟机 --> 设置**

![](https://pic3.zhimg.com/80/v2-00b85a0bbb4431147c8f3874474a913a_720w.jpg)

- **设置共享文件夹路径**

  在 `虚拟机设置` 中，选择 `选项` 标签，再点击 `共享文件夹`，然后点击右侧的 `总是启用` ，之后再点击 `添加…` ，添加 Windows 主机上的共享目录，后面就跟着提示一步步操作即可。

![](https://pic2.zhimg.com/80/v2-5bf4f8cd50e8b0266364b4af5127a1d1_720w.jpg)

#### 2、打开创建的虚拟机查看共享文件夹

​		root 用户组，进入目录 `/mnt/hgfs/` 即可看见配置的 Windows 全局共享文件夹。如果没有该目录则是 `VMware tools` 没有安装，请先安装后重启，在从头操作一遍即可。

​		若已经有 `/mnt/hgfs/` 目录，但是里面没有刚刚配置的共享文件夹，可以尝试执行下面的命令来挂载共享目录：

```sh
sudo vmhgfs-fuse .host:/ /mnt/hgfs
```

​		然后重新进入 `/mnt/hgfs/` 即可。但是这样每次重启都要重新执行该命令才会显示，可以通过下面的方法解决：

```sh
gedit /etc/fstab
```

​		在最后添加一行:

```ini
.host:/ /mnt/hgfs fuse.vmhgfs-fuse allow_other 0 0
```

​		重启虚拟机即可。