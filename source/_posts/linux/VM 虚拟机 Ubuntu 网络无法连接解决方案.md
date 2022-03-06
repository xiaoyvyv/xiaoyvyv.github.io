---
title: VM 虚拟机 Ubuntu 无法连接到网络，解决方案 
date: 2022-02-11 14:46:30
tags: [linux, vm-ware, ubuntu]
---

### VM 虚拟机 Ubuntu 无法连接到网络，解决方案

#### 1、检查网络相关的管理器是否正常

​		桌面右上角设置点开，查看 Ubuntu 网络设置中是否有联网的相关设置：【Wired】，没有的话需要恢复。

![](https://img-blog.csdnimg.cn/a50180af2433476f93ffafcdbc09de9a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBATHRNYW1iYQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

- ####  解决办法

```sh
# 打开终端，依次输入下面命令执行
sudo service network-manager stop
sudo rm /var/lib/NetworkManager/NetworkManager.state
sudo service network-manager start

# 到这应该就能看到按钮了，如果还看不到接着执行下面的
# 将其中的 managed=false 改为 managed=true
sudo gedit /etc/NetWorkManager/NetworkManager.conf
sudo service network-manager restart
```

#### 2、重置虚拟机网络配置

​		关闭虚拟机，在 VM ware 的编辑菜单里面，点击 `虚拟网络编辑器` ，然后依次点击：

> 1. 【更改设置】
>
> 2. 【还原默认设置】
> 3. 【确定】

​		然后将虚拟机 Ubuntu 的网络适配器设置中，设备状态的两个选项打勾，网络适配模式选择为【NAT 模式】，点击确定保存配置，然后启动虚拟机。正常情况到此就可以正常上网了。若还不行，可以尝试下面第三步。

#### 3、 设置 Windows 主机共享网络适配器

​		打开 Windows 的网络适配器面板，打开你的外部有网的那一个适配器的 `属性` 面板，选择 `共享` TAB，然后选中【允许其他网络用户通过此计算机的 Internet 连接】，选择专有网络为 【VMnet8】，然后确定，重启虚拟机即可。



