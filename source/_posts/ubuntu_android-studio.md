---
title: ubuntu安装android-studio
tags: [android]
---

## 安装JDK
1、由于我一键安装android-studio时openJDK依赖关系发生错误，所以这里手动下载JDK，先选Accept License Agreement后可下载。我的Linux系统为ubuntukylin 16.04 64位的，所以选择64位的版本。

2、进入默认的下载目录下 home/下载/ 进行解压文件
``` bash
   $ tar -zxvf jdk-8u91-linux-x64.tar.gz && sudo mv jdk1.8.0_91 /opt
```
<!--more-->
3、配置jdk环境变量
``` bash
   $ sudo vi /etc/profile
```
   在文件最后加上以下内容，注意自己的jdk路径：
   export JAVA_HOME=/opt/jdk1.8.0_91   
   export JRE_HOME=${JAVA_HOME}/jre    
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:${JAVA_HOME}/lib/tools.jar    
   export PATH=${JAVA_HOME}/bin:$PATH  


4、在该全件最后也添加3中内容
``` bash
   $ sudo vi /home/lc/.bashrc
```
5、同步更新3、4中的两个文件：
``` bash
   $ source /etc/profile && source /home/lc/.bashrc
```
使用java -version测试下环境变量是否设置成功。

## 安装android-studio和sdk
1、在www.android-studio.org中下载android-studio和sdk。同2中一样分别解压两个文件，但需要注意由于sdk以后会越来越大，所以建议将sdk解压到/home/下。防止根目录空间不足。

2、
``` bash
   $ sudo vi /opt/android-studio/bin/idea.porperties
```
在最后添加disable.android.first.run=true 否则会因s为网络问题一直卡在"Fetching Android SDK component information"

3、
``` bash
   $ sh /opt/android-studio-XXXX/bin/studio.sh
```
在弹窗中分别配置jdk和sdk的路径，至此恭喜你安装完成。

4、安装完成后需要安装不同版本的sdk，启动sdk有两种方式：
    1）在${SDK_HOME}/tools/下执行./android，就可以单独启动sdk进行不同版本SDK的安装，若出现网络问题，www.android-studio.org上有代理教程可以设置代理进行下载。
    2）在android-studio中直接启动SDK。

5、设置adb环境变量，在/etc/profile最后面加上如下内容，同步更新这个文件即可：
    export PATH=$PATH:/home/lc/android-sdk-linux/tools/
    export PATH=$PATH:/home/lc/android-sdk-linux/platform-tools/

## 问题汇总
1、由于64位ubuntu在编译打包APK时缺少X86下的C++语言库，所以需要安装库先：
    sudo apt-get install lib32stdc++6 lib32z1，否则会报如下错误：
    java.io.IOException: Cannot run program "/home/lc/workspace/android-sdk-linux/build-tools/23.0.2/aapt": error=2, 没有那个文件或目录

2、如果monitor中报错提示：pls config SDK，同时project structure中提示：ndk does not contain any platforms，则需要对ndk进行更新，否则gradle同步时会报系统内部错误。

3、如果存在android升级不生效，升级包下载后不重启的问题，则需要在/etc/profile中将${ANDOIR_STUDIO_HOME}/bin添加到环境变量中，因为找不到studio.sh，所以下载完升级包后无法自动重启升级。

到这里ubuntu上的android开发环境基本已经大功告成，可以尽情的玩耍了。
