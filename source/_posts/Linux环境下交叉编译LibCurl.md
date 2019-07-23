---
title: Linux环境下交叉编译LibCurl
tags: [NDK,Curl]
grammar_cjkRuby: true
categories: [Android]
date: 2019-07-24
---

1、下载源码


2、设置环境变量

```sh
#LDFLAGS是告诉编译器从哪里寻找需要的库文件
export LDFLAGS="-L/usr/my/android-ndk-r20/platforms/android-21/arch-arm/usr/lib"

#CPPFLAGS可选的编译器选项，在编译 C/C++ 代码文件的时候使用。在最新的ndk中没有这个目录了，所以不要了。
# export CPPFLAGS="-I/usr/my/android-ndk-r20/platforms/android-21/arch-arm/usr/include"

```

3、从NDK导出编译工具
```sh
#NDK_HOME为安装路径
export NDK_HOME=/usr/my/android-ndk-r20
$NDK_HOME/build/tools/make-standalone-toolchain.sh --arch=arm --platform=android-21 --install-dir=$HOME/android-toolchain --toolchain=arm-linux-androideabi-4.9

# 将生成的目录添加到环境变量里
export PATH=$PATH:/usr/my/android-ndk-r20/android-toolchain/bin

```

4、生成Config文件
```sh
#系统默认gcc，需要指定使用ndk中的arm-gcc：
SYSROOT=/usr/my/android-ndk-r20/platforms/android-21/arch-arm
export CC="/usr/my/android-ndk-r20/android-toolchain/bin/arm-linux-androideabi-gcc --sysroot=$SYSROOT"

#生成config文件
./configure --host=arm-linux-androideabi --without-ssl --disable-ftp --disable-gopher --disable-file --disable-imap --disable-ldap --disable-ldaps --disable-pop3 --disable-proxy --disable-rtsp --disable-smtp --disable-telnet --disable-tftp --without-gnutls --without-libidn --without-librtmp --disable-dict
```
5、编译
```sh
make

```
生成libcurl的库文件，在lib/.libs下。

如果过程中出现异常，请仔细检查各环境变量的路径是否正确。

参考资料：
1、https://blog.csdn.net/cnhua57inyu/article/details/41693661
2、https://www.jianshu.com/p/3bbad4b1b099