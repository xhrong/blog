---
title: ADB命令集锦
tags: [Android,ADB]
grammar_cjkRuby: true
categories: [Android]
date: 2022-11-24
---

# ADB命令集锦

## 安卓截屏并导出到电脑

  高版本安卓支持直接保存：

```
adb exec-out screencap -p > 1.png
```

  低版本安卓，先截图放在手机的sdcard里，然后pull到电脑端：

```
adb shell screencap /sdcard/1.png
adb pull /sdcard/1.png ./
```

----

## 同屏到PC端并可反向控制（ScrCpy）

scrcpy就是通过adb调试的方式来将手机屏幕投到电脑上，并可以通过电脑控制您的设备。它可以通过USB连接，也可以通过Wifi连接（类似于隔空投屏），而且不需要任何root权限，不需要在手机里安装任何程序。scrcpy同时兼容HarmonyOS、Windows、macOS和GNU / Linux。

下载地址：https://github.com/Genymobile/scrcpy

http://xhrong.github.io/attachments/scrcpy-win64-v2.1.1.zip

使用方法：
- 方法一，使用USB连接：

  1、通过数据线连接手机和电脑来实现投屏。第一步我们还是要开启手机的“开发者选项”，勾选“USB调试”。

  2、接着我们打开解压后的scrcpy文件（资源可在后台私信获取），直接运行scrcpy.exe。

- 方法二，使用WiFi连接：

  1、除了有线连接外，scrcpy还支持无线连接，如此更能摆脱对数据线的依赖。只是设置的步骤相对有线连接颇为复杂。而实现无线连接的基本要求是手机与电脑连接同一Wi-Fi。

  2、其次，在数据线连接手机的情况下，我们输入“adb tcpip 5555”，回车之后出现“restarting in TCP mode port: 5555”之后便可断开数据连

相关资源：
- https://zhuanlan.zhihu.com/p/383724229
- https://blog.csdn.net/pz641/article/details/131241121
- https://blog.csdn.net/qq_42747139/article/details/104163312 （比较详细的使用参数说明）
- https://sspai.com/post/56402
----

## 设置Device Owner
```
adb shell dpm set-device-owner com.afwsamples.testdpc/.DeviceAdminReceiver
```

但是设置时候可能出现提示已经存在账号，解决方法：
1、检查设置 – 账号删除掉所有账号；
2、如果操作完1之后，问题依旧，显示：
```
java.lang.IllegalStateException:Notallowedtosetthedevice owner because there are already some accounts on the device
```
就执行如下操作：
```
adb shell pm list users
```
显示结果为：
```
Users:
UserInfo{0:鏈轰富:13} running
UserInfo{999:Multi-App:4000030} running
```
结果显示存在两个账户，删除第二个999即可。
```
adb shell pm remove-user 用户ID
```
adb代码如下：
```
adb shell pm remove-user 999
```
执行删除用户指令成功后会提示”Success”,然后再重新设置set-device-owner即可。

其它相关命令：


```
usage: dpm [subcommand] [options]
usage: dpm set-active-admin [ --user <USER_ID> | current ] <COMPONENT>
usage: dpm set-device-owner [ --user <USER_ID> | current *EXPERIMENTAL* ] [ --name <NAME> ] <COMPONENT>
usage: dpm set-profile-owner [ --user <USER_ID> | current ] [ --name <NAME> ] <COMPONENT>
usage: dpm remove-active-admin [ --user <USER_ID> | current ] [ --name <NAME> ] <COMPONENT>

dpm set-active-admin: Sets the given component as active admin for an existing user.

dpm set-device-owner: Sets the given component as active admin, and its package as device owner.

dpm set-profile-owner: Sets the given component as active admin and profile owner for an existing user.

dpm remove-active-admin: Disables an active admin, the admin must have declared android:testOnly in the application in its manifest. This will also remove device and profile owners.

dpm clear-freeze-period-record: clears framework-maintained record of past freeze periods that the device went through. For use during feature development to prevent triggering restriction on setting freeze periods.

dpm force-network-logs: makes all network logs available to the DPC and triggers DeviceAdminReceiver.onNetworkLogsAvailable() if needed.

dpm force-security-logs: makes all security logs available to the DPC and triggers DeviceAdminReceiver.onSecurityLogsAvailable() if needed.
usage: dpm mark-profile-owner-on-organization-owned-device: [ --user <USER_ID> | current ] <COMPONENT>
```
----


## 恢复出厂
```
adb shell reboot recovery
```
如果遇到提示“no command”，可按如下操作解决：

只需先按住电源键(电源键不松开)，然后再按一下 音量加即可进入官方Recovery，可进行双清恢复出厂设置 (wipe data/factory reset)


----

## 查看CPU信息
```
adb shell cat /proc/cpuinfo 
```
结果：
```
processor       : 0
BogoMIPS        : 52.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm lrcpc dcpop asimddp  
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x2
CPU part        : 0xd05
CPU revision    : 0

processor       : 1
BogoMIPS        : 52.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm lrcpc dcpop asimddp  
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x2
CPU part        : 0xd05
CPU revision    : 0

processor       : 2
BogoMIPS        : 52.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm lrcpc dcpop asimddp  
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x2
CPU part        : 0xd05
CPU revision    : 0

省略

processor       : 7
BogoMIPS        : 52.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm lrcpc dcpop asimddp
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x3
CPU part        : 0xd0a
CPU revision    : 1

Serial          : 007424880a214886
Hardware        : Unisoc T616
```
----

## 发送广播、启动Activity、启动Service、强制停止


- **adb shell am broadcast [options] <INTENT>**

作用：发送一个广播

举例：adb shell am broadcast -a "action_finish" （发送一个广播去关闭一个activity）

举例：adb shell am broadcast -a android.intent.action.MASTER_CLEAR（恢复出厂设置的方法，会清除内存所有内容）

举例：adb shell am broadcast -n com.lt.test/.MyBroadcast

详细说明：

```
adb shell am broadcast -a com.android.test --es test_string "this is test string" --ei test_int 100 --ez test_boolean true
```

adb shell am broadcast 后面的参数有：
```
[-a <ACTION>]
[-d <DATA_URI>]
[-t <MIME_TYPE>] 
[-c <CATEGORY> [-c <CATEGORY>] ...] 
[-e|--es <EXTRA_KEY> <EXTRA_STRING_VALUE> ...] 
[--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> ...] 
[-e|--ei <EXTRA_KEY> <EXTRA_INT_VALUE> ...] 
[-n <COMPONENT>]
[-f <FLAGS>] [<URI>]
```


- **adb shell am start [options] <INTENT>**

作用：启动一个activity

举例：adb shell am start -a com.lt.test.action.SECOND

举例：adb shell am start -n com.lt.test/.MyActivity



- **adb shell am startservice [options] <INTENT>**

作用：启动一个service

举例：adb shell am startservice -a com.lt.test.action.ONESERVICE

举例：adb shell am startservice -n com.lt.test/.MyService

 
- **adb shell am force-stop <PACKAGE>**
  
作用：强制关闭一个应用程序

举例：adb shell am force-stop com.lt.test

----

## 修改屏幕分辨率、DPI

查看分辨率
```
C:\Users\54013>adb shell wm size
Physical size: 1080x2340
```
修改分辨率
```
C:\Users\54013>adb shell wm size 1080x1920
C:\Users\54013>adb shell wm size
Physical size: 1080x2340
Override size: 1080x1920
```
恢复默认的分辨率
```
C:\Users\54013>adb shell wm size reset
```

查看dpi
```
C:\Users\54013>adb shell wm density
Physical density: 480
```
修改dpi
```
C:\Users\54013>adb shell wm density 320
C:\Users\54013>adb shell wm density
Physical density: 480
Override density: 320
```
恢复默认的dpi
```
C:\Users\54013>adb shell wm density reset
```
----

## 内存分析VSS RSS PSS USS
一个进程的内存信息可以用VSS RSS PSS USS来表示，含义如下：
- VSS（Virtual Set Size）：虚拟内存大小（ps命令用VSZ表示）；
- RSS（Resident Set Size）：常驻内存大小，应用使用的共享和非共享页面的数量；
- PSS（Proportional Set Size）：按比例分摊的内存大小，应用使用的非共享内存加上共享内存的均匀分摊数量（例如，如果三个进程共享 3MB，则每个进程的 PSS 为 1MB）；
- USS（Unique Set Size）：进程独占用的物理内存（不包含共享库占用的内存）；

举例：已知：

（1）共享库libTest.so所占内存共100M，被进程A和进程B共同占用；

（2）进程A申请了100M内存，但是实际使用了60M内存；

则，进程A的VSS=100M+100M=200M; RSS=60M+100M=150M; PSS=60M+100/2=110M; USS=60M;

所以，如果想要知道所有进程使用了多少内存，那么可以使用PSS 或 RSS。计算 PSS 需要花很长时间，因为系统需要确定共享的页面以及共享页面的进程数量。RSS 不区分共享和非共享页面（因此计算起来更快），更适合跟踪内存分配量的变化。

命令：
```
adb shell ps

adb shell dumpsys meminfo

adb shell cat /proc/meminfo

adb shell procrank
```
工具：Android Studio Profiler

详细说明：https://blog.csdn.net/weixin_55626853/article/details/121236583

----

## 模拟手机按键
执行 adb shell input 命令
```
Usage: input [<source>] [-d DISPLAY_ID] <command> [<arg>...]

The sources are:
      dpad
      keyboard
      mouse
      touchpad
      gamepad
      touchnavigation
      joystick
      touchscreen
      stylus
      trackball

-d: specify the display ID.
      (Default: -1 for key event, 0 for motion event if not specified.)
The commands and default sources are:
      text <string> (Default: touchscreen)
      keyevent [--longpress] <key code number or name> ... (Default: keyboard)
      tap <x> <y> (Default: touchscreen)
      swipe <x1> <y1> <x2> <y2> [duration(ms)] (Default: touchscreen)
      draganddrop <x1> <y1> <x2> <y2> [duration(ms)] (Default: touchscreen)
      press (Default: trackball)
      roll <dx> <dy> (Default: trackball)
      motionevent <DOWN|UP|MOVE> <x> <y> (Default: touchscreen)
  ```

- 模拟点击事件
  
  adb shell input tap x坐标 y坐标

  adb shell input tap 528 1539

- 输入文本

  首先需要把光标移到输入框，然后执行以下命令

  adb shell input text zengzengzeng

- 模拟滑动事件

  adb shell swip <起点x> <起点y> <终点x> <终点y> <滑动时长>

  adb shell input swipe 528 1539 528 1300 2000

  2000为滑动时间，单位是毫秒

- 返回键

  adb shell input keyevent 4

- 返回home键（置应用于后台）

  adb shell input keyevent 3

- 音量放大

  adb shell input keyevent 24

- 音量缩小

  adb shell input keyevent 25
  
  ----

## 获取当前Activity的UI Hierchary（视图层次结构）

```
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml
```

uiautomator使用说明：

``` shell
C:\>adb shell uiautomator help
Usage: uiautomator <subcommand> [options]
 
Available subcommands:
 
help: displays help message
 
runtest: executes UI automation tests
    runtest <class spec> [options]
    <class spec>: <JARS> < -c <CLASSES> | -e class <CLASSES> >
      <JARS>: a list of jar files containing test classes and dependencies. If
        the path is relative, it's assumed to be under /data/local/tmp. Use
        absolute path if the file is elsewhere. Multiple files can be
        specified, separated by space.
      <CLASSES>: a list of test class names to run, separated by comma. To
        a single method, use TestClass#testMethod format. The -e or -c option
        may be repeated. This option is not required and if not provided then
        all the tests in provided jars will be run automatically.
    options:
      --nohup: trap SIG_HUP, so test won't terminate even if parent process
               is terminated, e.g. USB is disconnected.
      -e debug [true|false]: wait for debugger to connect before starting.
      -e runner [CLASS]: use specified test runner class instead. If
        unspecified, framework default runner will be used.
      -e <NAME> <VALUE>: other name-value pairs to be passed to test classes.
        May be repeated.
      -e outputFormat simple | -s: enabled less verbose JUnit style output.
 
dump: creates an XML dump of current UI hierarchy
    dump [--verbose][file]
      [--compressed]: dumps compressed layout information.
      [file]: the location where the dumped XML should be stored, default is
      /storage/self/primary/window_dump.xml
 
events: prints out accessibility events until terminated

```
----


## ADB给应用授权

方式一：
```
adb shell pm grant com.paget96.batteryguru android.permission.PACKAGE_USAGE_STATS
```
方式二：
```
adb shell appops set com.iflytek.openmdm android:get_usage_stats allow
```

扩展：adb授权应用android.permission.PACKAGE_USAGE_STATS权限后，除了可查询应用使用情况，还可实现监听当前前置应用Activity的功能。代码如下：

```java
    public String getTopActivity(Context context) {
        String topActivityName = "";
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            UsageStatsManager mUsageStatsManager = (UsageStatsManager) context.getSystemService(Context.USAGE_STATS_SERVICE);
            long time = System.currentTimeMillis();
            UsageEvents usageEvents = mUsageStatsManager.queryEvents(time - 1000 * 3, System.currentTimeMillis() + (1 * 1000));
            UsageEvents.Event event = new UsageEvents.Event();
            while (usageEvents.hasNextEvent()) {
                if (event != null && !TextUtils.isEmpty(event.getPackageName()) && event.getEventType() == UsageEvents.Event.MOVE_TO_FOREGROUND) {
                    topActivityName = event.getClassName();
                } else {
                    topActivityName = "";
                }
                usageEvents.getNextEvent(event);
            }
        } else {//兼容Android5以下
            ActivityManager am = (ActivityManager) context.getSystemService(context.ACTIVITY_SERVICE);
            List<ActivityManager.RunningTaskInfo> taskInfo = am.getRunningTasks(1);
            ComponentName componentInfo = taskInfo.get(0).topActivity;
            topActivityName = componentInfo.getClassName();
        }
        return topActivityName;
    }
```
-----

## 查看当前Top应用的包名和Activity名

```
adb shell dumpsys activity activities | findstr mFoc
```

