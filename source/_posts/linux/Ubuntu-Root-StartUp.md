---
title: Ubuntu 设置开机 Root 用户登录
date: 2022-02-11 14:46:30
tags: linux
---

### Ubuntu 设置开机 Root 用户登录

​		默认安装 Ubuntu 18+ 都是不允许以root用户进行登录的，想要以root用户进行登录需要进行一些操作，主要是以下几个步骤：

- #### 第一步：以普通用户登录系统，创建root用户的密码

  在终端输入命令：

  ```shell
  sudo passwd root
  ```

  然后输入你要设置的密码，这样就完成了设置root用户密码的步骤。

- #### 第二步：设置登录界面选择其他用户登录按钮

  修改文件 ` /usr/share/lightdm/lightdm.conf.d/50-xxxxxx.conf` 文件，增加两行：

  ```ini
  greeter-show-manual-login=true
  all-guest=false
  ```

  保存即可。

- #### 第三步：设置允许 Root 进行登录操作，未设置在登录页会不允许 Root 用户

  进入 `/etc/pam.d` 目录，修改 `gdm-autologin` 和 `gdm-password` 文件

  `gedit gdm-autologin`

  ```ini
  # 注释掉这一行，保存
  # auth required pam_succeed_if.so user != root quiet_success
  ```

  `vi gdm-password`

  ```ini
  # 注释掉这一行，保存
  # auth required pam_succeed_if.so user != root quiet_success
  ```

- #### 第四步：修改 .profile 文件

  `gedit /root/.profile` ，将文件末尾的 `mesg n || true` 这一行修改成 `tty -s && mesg n || true`， 保存。

- #### 第五步：输入 reboot 重启系统，输入 root 用户名和密码，登录系统。