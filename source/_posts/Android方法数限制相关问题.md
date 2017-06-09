---
title: Android方法数限制相关问题
date: 2016-11-22 18:28:48
tags: [Android]
categories: [Android]
---


### Android方法数限制相关问题
@(技术学习)[方法数]

[TOC]

#### 检查APK方法数量
dex-method-counts项目（Github地址：https://github.com/mihaip/dex-method-counts ）可以通过Ant或者Gradle执行。在window环境下也可以通过命令行执行jar文件的方式操作。（JAR包下载：http://download.csdn.net/download/qq376430645/9275961）
命令行下进入该项目所在目录，使用gradle build进行编译。在目录下build\jar目录可以找到编译完成的dex-method-counts.jar文件。

然后使用命令执行：Java -jar (dex-method-counts.jar所在目录) (想要统计的apk文件所在文件夹)

就可以详细的看到应用总共的方法数以及每个包的方法数。

例如： 
![Alt text](./1477443982101.png)

执行结果如下： 
![Alt text](./1477443993332.png)

可以看到这个应用的方法数是48185，以及每个引用包里面具体的方法数；这样也可以为我们选择依赖库提供一个维度的参考！

#### 方法数限制的根本原因

##### Android打包流程
![Alt text](./1477444662073.png)
1、IDE中的资源打包工具 (Android Asset Packaging Tool ，即图中的aapt) 会将应用中的资源文件进行编译，这些资源文件包括AndroidManifest.xml文件，为Activity定义的 XML 文件等等。在这个编译过程中也会产生一个R.java文件，这样你就可以在你的Java代码中引用这些资源了。
2、aidl 工具会将你项目中的所有.aidl接口转换成Java接口。
3、项目中的所有的Java代码，包括R.java和.aidl文件，都会被Java编译器编译，然后输出 .class 文件。
4、接着 dex 工具就会把上一步骤产生的 .class 文件转成 Dalvik 字节码，也就是.dex文件。同时项目中包含的所有第三方类库和 .class 文件也会被转换成.dex文件，这样讲方便下一步被打包成最终的.apk文件。
5、所有的不能编译的资源（比如图片等等）、编译后的资源文件和 .dex 文件会被 apkbuilder 工具打包成一个.apk文件。
6、一旦.apk文件被构建好之后，如果要把把它安装到设备上面去的话，它就必须用一个debug 或者发行key来对这个apk文件签名。
7、最后，如果应用程序已经被签名成为发行模式的apk，你还需要使用aipalign工具对.apk进行对齐优化。这样的话可以减少应用程序在设备上的内存消耗。
##### Dex与方法数记数
我们注意到在第四步的时候，会产生一个.dex文件。Android 从之前的Dalvik 到现在Android 5.0 默认的ART 运行时环境都能够执行这个.dex文件，它们还使用同一套指令集，即Dalvik指令集。**Dalvik指令集是使用16位寄存器来保存项目中所有的方法引用，包括第三方的方法**。
这就意味着Android的单个.dex文件最能引用65536个方法，在这之后的方法就无法引用了。这就是Android Dex 方法限制异常出现的原因，同时因为ART和Dalvik使用同一套指令集，这个限制在ART 运行时环境中也会存在。

#### 解决方法数限制问题

##### 方法一：配置dex.force.jumbo

可能有些同学会说，解决这个问题很简单，我们只需要在Project.proterty中配置一句话就Ok啦，
dex.force.jumbo=true
是的，加入了这句话，确实可以让你的应用通过编译，但是在一些2.3系统的机器上很容易出现INSTALL_FAILED_DEXOPT异常
##### 方法二：减少不必要的引用
1、利用dex-method-counts工具检查方法数量情况之后，考虑删除或替换相关的引用依赖。

2、使用ProGuard清除项目中无用的方法，不过效果不如上面的。
##### 方法三：分包：基于gradle构建Android项目，并实现分包


1、如果你的工程在eclipse中，那么你需要将该工程导入到Android中，此时需要你升级adt22以上

2、打开你工程的build.gradle文件，检查gradle插件是否是0.14.2版本之后，因为0.14.2之后gradle插件才支持分包

3、打开工程下某一个Moudle的build.gradle文件，添加对android-support-multidex.jar的依赖

![Alt text](./1477445813408.png)

4、去掉第三方jar包中重复的类
![Alt text](./1477445828909.png)

5、设置虚拟机堆内存空间大小，避免在编译期间OOM
![Alt text](./1477445838060.png)

6、gradle构建项目时，貌似默认是不会将so库加入工程的，所以为了避免此种情况发生，我们需要制定so库目录，对于从eclipse转换过来的工程，还需要制定src和资源文件路径

![Alt text](./1477445851314.png)

7、如果你的项目依赖了其他库， 分别在各个库工程中加入 multiDexEnabled = true 和 jniLibs.srcDirs =['libs']两个配置即可

8、如果你的项目没有自定义Application，那么你在AndroidManifest.xml中使用MultiDexApplication即可，如果你的项目有自定义Application,并且是继承是Application，那么只需要改为继承MultiDexApplication即可，如果你的项目时继承的其他Application，那么你需要重写

```
@Override
protected void attachBaseContext(Context base) {
    super.attachBaseContext(base);
    MultiDex.install(this);
}
```
经过上述配置，你的项目应该是已经成功分包了。如果分包成功，那么你解压你的apk文件，会发现有两个dex文件，通过上述的配置过程，我们发现此方案我们无法控制哪些类在main.dex中，哪些类在second.dex中，通过此种方案配置分包，可以兼容
![Alt text](./1477445900418.png)


彻底解决Android 应用方法数不能超过65K的问题5
##### 方法四：插件化

工作原理
如下图所示，首先宿主程序会到文件系统比如sd卡去加载apk，然后通过一个叫做proxy的activity去执行apk中的activity。
关于动态加载apk，理论上可以用到的有DexClassLoader、PathClassLoader和URLClassLoader。
DexClassLoader ：可以加载文件系统上的jar、dex、apk
PathClassLoader ：可以加载/data/app目录下的apk，这也意味着，它只能加载已经安装的apk
URLClassLoader ：可以加载Java中的jar，但是由于dalvik不能直接识别jar，所以此方法在Android中无法使用，尽管还有这个类
关于jar、dex和apk，dex和apk是可以直接加载的，因为它们都是或者内部有dex文件，而原始的jar是不行的，必须转换成dalvik所能识别的字节码文件，转换工具可以使用android sdk中platform-tools目录下的dx
转换命令 ：dx --dex --output=dest.jar src.jar
![Alt text](./1477445952994.png)
