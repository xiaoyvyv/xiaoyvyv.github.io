---
title: 在 Ubuntu 上编译 FreeRDP Android 端的二进制 so 库
date: 2022-02-11 14:46:30
tags: [android, linux]
---
### 在 Ubuntu 上编译 FreeRDP Android 端的二进制 so 库

需要用到的环境和软件：

- Java
- make    (Ubuntu 自带，没有需要安装)
- git    (自行安装)
- python3    (Ubuntu 自带，没有需要安装)
- Android Studio
- cmake    (使用Adnroid Studio 下载)
- Android Sdk    (使用Adnroid Studio 下载)
- Android Ndk    (使用Adnroid Studio 下载)

#### 1、安装 Jdk17

1. 下载官方 Jdk17 文件：

   ```shell
   mkdir /usr/local/java/jdk17
   wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz -P /usr/local/java/jdk17
   或
   wget http://49.232.8.65/jdk/jdk17/jdk-17_linux-x64_bin.tar.gz -P /usr/local/java/jdk17
   ```

2. 解压

   ```shell
   tar xf /home/jdk17/jdk-17_linux-x64_bin.tar.gz -C /usr/local/java/jdk17
   ```

3. 设置环境变量

   `gedit /etc/profile` ，在文件最下方添加以下内容：

   > 后面的版本号根据你自己解压下来的 Jdk 文件夹版本名称而定

   ```ini
   # JAVA
   export JAVA_HOME=/usr/local/java/jdk17/jdk-17.0.1
   export PATH=$JAVA_HOME/bin:$PATH
   ```

#### 2、安装 Android Studio

1. 下载最新版本的 Android Studio，下载完成后解压到你想要的目录即可。

   ```shell
   wget https://redirector.gvt1.com/edgedl/android/studio/ide-zips/2021.1.1.21/android-studio-2021.1.1.21-linux.tar.gz
   ```

   我这里解压后，将解压出来的 `android-studio` 目录放到了 `/usr/local/` 目录下：

   ```
   /usr/local/android-studio
   ```

   进入 `android-studio/bin` 目录，执行 `./studio.sh` 启动软件。

2. 启动成功后，配置完成，记录下你配置的 SDK 目录，建议一路配置都用默认即可

   我这里的 SDK 目录默认为：

   ```
   /root/Android/Sdk
   ```

   然后进入新建项目的界面，我们不用新建项目，点击左侧的 `Customize -> All settings` ，在设置搜索框里输入 `SDK` 按回车键。

   然后点开 `Android SDK` 的配置页面：选中 `SDK Tools` Tab，勾选中 `NDK (Size by side)` 和 `cmake` ，然后点击 `Apply`，等待下载完成关闭 Android Studio。

3. 然后进入 SDK 目录，你会发现 `cmake` 和 `ndk` 都在该目录内。

   > 注意：
   >
   > cmake 目录下没有直接放相关文件，而是多了一层版本号的文件夹，这里需要修改一下，将版本号文件夹内的全部文件复制一份到上一级目录下，免得后面执行编译脚本报错。

   `cmake` 路径修改，`$SDK/cmake/3.18.1/xxxxx -> $SDK/cmake/xxxxx` 

4. 添加环境变量

   `gedit /etc/profile` ，在文件最下方添加以下内容：

   > 版本号和你的有差异请自己替换

   ```ini
   # ANDROID_SDK
   export ANDROID_SDK=/root/Android/Sdk
   export PATH=$ANDROID_SDK/tools:$PATH
   
   # ANDROID_NDK
   export ANDROID_NDK=$ANDROID_SDK/ndk/23.1.7779620
   export PATH=$ANDROID_NDK:$PATH
   
   # ANDROID_CMAKE
   export ANDROID_CMAKE=$ANDROID_SDK/cmake/bin:$PATH
   export PATH=$ANDROID_CMAKE:$PATH
   ```

   然后刷新配置

   ```shell
   source /etc/profile
   ```

   此时可以输入：`java -version` 、`cmake` 、 `ndk-build` 进行检测是否配置好

#### 3、下载 FreeRdp

1. 在 `/root` 目录新建一个文件夹 `Work`，进入该文件夹：

2. 下载 Github 源码：

   ```shell
   git clone https://github.com/FreeRDP/FreeRDP.git
   ```

3. 开始编译

   进入克隆下来的 FreeRDP 目录，输入以下命令开始编译

   ```shell
   ./scripts/android-build-freerdp.sh --ndk $ANDROID_NDK --sdk $ANDROID_SDK --release
   ```

   若需要编译 H264，修改 `android-build-release.conf` 开启 H264 `WITH_OPENH264=1`

   ```sh
   ./scripts/android-build-freerdp.sh --ndk $ANDROID_NDK --sdk $ANDROID_SDK --openh264-ndk $ANDROID_NDK_15C --openh264 --release
   ```

   等待编译完成，若前面的步骤有错误，期间可能会编译报错，看错误提示解决后，重新执行上面的编译命令即可。

4. 查看编译好的 so 文件

   编译完成后，so 存放目录为：

   ```shell
   /root/Work/FreeRDP/client/Android/Studio/freeRDPCore/src/main/jniLibs
   ```

### 4、到此编译完成

剩下的就是把这个导出来，放到 Android Studio 相关的项目里就行了，具体相关的源码可以访问 FreeRDP Github 地址：[https://github.com/FreeRDP/FreeRDP](https://github.com/FreeRDP/FreeRDP)

