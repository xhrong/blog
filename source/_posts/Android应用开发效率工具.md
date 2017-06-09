---
title: Android应用开发效率工具 
tags: [LeakCanary,Timber,Bugly,BlockCanary]
grammar_cjkRuby: true
categories: [Android]
date: 2017-04-15
---

### Memory：LeakCanary
**LeakCanary：A memory leak detection library for Android and Java.**

In your `build.gradle`:

```gradle
 dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
   testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
 }
```

In your `Application` class:

```java
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      // This process is dedicated to LeakCanary for heap analysis.
      // You should not init your app in this process.
      return;
    }
    LeakCanary.install(this);
    // Normal app init code...
  }
}
```
LeakCanary will automatically show a notification when an activity memory leak is detected in your debug build.

![enter description here][1]

注：对于复杂模块，内存泄漏可能存在多处，leakCanary的分析界面可能比较复杂。可能考虑先用Android Studio的Monitors先大致观察一下，初步定位问题，然后再通过leakCanary具体分析

![enter description here][2]

### ANR：AndroidPerformanceMonitor（BlockCanary）
**AndroidPerformanceMonitor：A transparent ui-block detection library for Android, app only needs one-line-code to setup.**


In your `build.gradle`:

```gradle
dependencies {
    // most often used way, enable notification to notify block event
    compile 'com.github.markzhai:blockcanary-android:1.5.0'

    // this way you only enable BlockCanary in debug package
    // debugCompile 'com.github.markzhai:blockcanary-android:1.5.0'
    // releaseCompile 'com.github.markzhai:blockcanary-no-op:1.5.0'
}
```

In your `Application` class:

```java
public class DemoApplication extends Application {
    @Override
    public void onCreate() {
        // ...
        // Do it on main process
        BlockCanary.install(this, new AppBlockCanaryContext()).start();
    }
}
```
Implement your application `BlockCanaryContext` context (strongly recommend you to check all these configs)：
```java
public class AppBlockCanaryContext extends BlockCanaryContext {

    /**
     * Implement in your project.
     *
     * @return Qualifier which can specify this installation, like version + flavor.
     */
    public String provideQualifier() {
        return "unknown";
    }

    /**
     * Implement in your project.
     *
     * @return user id
     */
    public String provideUid() {
        return "uid";
    }

    /**
     * Network type
     *
     * @return {@link String} like 2G, 3G, 4G, wifi, etc.
     */
    public String provideNetworkType() {
        return "unknown";
    }

    /**
     * Config monitor duration, after this time BlockCanary will stop, use
     * with {@code BlockCanary}'s isMonitorDurationEnd
     *
     * @return monitor last duration (in hour)
     */
    public int provideMonitorDuration() {
        return -1;
    }

    /**
     * Config block threshold (in millis), dispatch over this duration is regarded as a BLOCK. You may set it
     * from performance of device.
     *
     * @return threshold in mills
     */
    public int provideBlockThreshold() {
        return 1000;
    }

    /**
     * Thread stack dump interval, use when block happens, BlockCanary will dump on main thread
     * stack according to current sample cycle.
     * <p>
     * Because the implementation mechanism of Looper, real dump interval would be longer than
     * the period specified here (especially when cpu is busier).
     * </p>
     *
     * @return dump interval (in millis)
     */
    public int provideDumpInterval() {
        return provideBlockThreshold();
    }

    /**
     * Path to save log, like "/blockcanary/", will save to sdcard if can.
     *
     * @return path of log files
     */
    public String providePath() {
        return "/blockcanary/";
    }

    /**
     * If need notification to notice block.
     *
     * @return true if need, else if not need.
     */
    public boolean displayNotification() {
        return true;
    }

    /**
     * Implement in your project, bundle files into a zip file.
     *
     * @param src  files before compress
     * @param dest files compressed
     * @return true if compression is successful
     */
    public boolean zip(File[] src, File dest) {
        return false;
    }

    /**
     * Implement in your project, bundled log files.
     *
     * @param zippedFile zipped file
     */
    public void upload(File zippedFile) {
        throw new UnsupportedOperationException();
    }


    /**
     * Packages that developer concern, by default it uses process name,
     * put high priority one in pre-order.
     *
     * @return null if simply concern only package with process name.
     */
    public List<String> concernPackages() {
        return null;
    }

    /**
     * Filter stack without any in concern package, used with @{code concernPackages}.
     *
     * @return true if filter, false it not.
     */
    public boolean filterNonConcernStack() {
        return false;
    }

    /**
     * Provide white list, entry in white list will not be shown in ui list.
     *
     * @return return null if you don't need white-list filter.
     */
    public List<String> provideWhiteList() {
        LinkedList<String> whiteList = new LinkedList<>();
        whiteList.add("org.chromium");
        return whiteList;
    }

    /**
     * Whether to delete files whose stack is in white list, used with white-list.
     *
     * @return true if delete, false it not.
     */
    public boolean deleteFilesInWhiteList() {
        return true;
    }

    /**
     * Block interceptor, developer may provide their own actions.
     */
    public void onBlock(Context context, BlockInfo blockInfo) {

    }
}
```
Maximum log count is set to 500, you can rewrite it in your app `int.xml`.
```xml
<integer name="block_canary_max_stored_count">1000</integer>
```

Monitor app's label and icon can be configured by placing a `block_canary_icon` drawable in your xhdpi drawable directory and in `strings.xml`:
```xml
<string name="block_canary_display_activity_label">Blocks</string>
```
![enter description here][3]

### Crash：Bugly
**集成SDK**

在Module的build.gradle文件中添加依赖和属性配置：
```gradle
dependencies {
    compile 'com.tencent.bugly:crashreport:latest.release' //其中latest.release指代最新Bugly SDK版本号，也可以指定明确的版本号，例如2.2.0
}
```
**同时集成SDK和NDK**

在Module的build.gradle文件中添加依赖和属性配置：
```gradle
android {
    defaultConfig {
        ndk {
            // 设置支持的SO库架构
            abiFilters 'armeabi' //, 'x86', 'armeabi-v7a', 'x86_64', 'arm64-v8a'
        }
    }
}

dependencies {
    compile 'com.tencent.bugly:crashreport:latest.release' //其中latest.release指代最新Bugly SDK版本号，也可以指定明确的版本号，例如2.1.9
    compile 'com.tencent.bugly:nativecrashreport:latest.release' //其中latest.release指代最新Bugly NDK版本号，也可以指定明确的版本号，例如3.0
}
```
**参数配置**

在AndroidManifest.xml中添加权限：
```xml
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_LOGS" />
```
请避免混淆Bugly，在Proguard混淆文件中增加以下配置：
```
-dontwarn com.tencent.bugly.**
-keep public class com.tencent.bugly.**{*;}
```

**最简单的初始化**

获取APP ID并将以下代码复制到项目Application类onCreate()中，Bugly会为自动检测环境并完成配置：
```java
CrashReport.initCrashReport(getApplicationContext(), "注册时申请的APPID", false); 
```
为了保证运营数据的准确性，建议不要在异步线程初始化Bugly。

第三个参数为SDK调试模式开关，调试模式的行为特性如下：

 -  输出详细的Bugly SDK的Log；
 -  每一条Crash都会被立即上报； 
 -  自定义日志将会在Logcat中输出。
 -   建议在测试阶段建议设置成true，发布时设置为false。
 
**此外，Bugly2.0及以上版本还支持通过“AndroidManifest.xml”来配置APP信息。如果同时又通过代码中配置了APP信息，则最终以代码配置的信息为准。**

在“AndroidManifest.xml”的“Application”中增加“meta-data”配置项：
```xml
<application
    <!-- 配置APP ID -->
    <meta-data
            android:name="BUGLY_APPID"
            android:value="<APP_ID>" />
    <!-- 配置APP版本号 -->
    <meta-data
            android:name="BUGLY_APP_VERSION"
            android:value="<APP_Version>" />
    <!-- 配置APP渠道号 -->
    <meta-data
            android:name="BUGLY_APP_CHANNEL"
            android:value="<APP_Channel>" />
    <!-- 配置Bugly调试模式（true或者false）-->
    <meta-data
            android:name="BUGLY_ENABLE_DEBUG"
            android:value="<isDebug>" />
</application>
```
不同于“android:versionName”，“BUGLY_APP_VERSION”配置的是Bugly平台的APP版本号。

通过“AndroidManifest.xml”配置后的初始化方法如下：
```java
CrashReport.initCrashReport(getApplicationContext());
```

**Bugly日志附加信息**

我们提供了一些信息记录API供您补充额外的内容。这些信息会随着异常一起上报。例如App环境、用户属性等等。主要包含以下接口：

1、设置用户ID 您可能会希望能精确定位到某个用户的异常，我们提供了用户ID记录接口。 例：网游用户登录后，通过该接口记录用户ID，在页面上可以精确定位到每个用户发生Crash的情况。
```java
CrashReport.setUserId("9527");  //该用户本次启动后的异常日志用户ID都将是9527
```
2、主动上报开发者Catch的异常 您可能会关注某些重要异常的Catch情况。我们提供了上报这类异常的接口。 例：统计某个重要的数据库读写问题比例。
```java
try {
    //...
} catch (Throwable thr) {
    CrashReport.postCatchedException(thr);  // bugly会将这个throwable上报
}
```
3、自定义日志功能 我们提供了自定义Log的接口，用于记录一些开发者关心的调试日志，可以更全面地反应App异常时的前后文环境。使用方式与android.util.Log一致。用户传入TAG和日志内容。该日志将在Logcat输出，并在发生异常时上报。有如下
```java
BuglyLog.v(tag, log)
BuglyLog.d(tag, log)
BuglyLog.i(tag, log)
BuglyLog.w(tag, log)
BuglyLog.e(tag, log)
```
注意：
- 使用BuglyLog接口时，为了减少磁盘IO次数，我们会先将日志缓存在内存中。当缓存大于一定阈值（默认10K），会将它持久化至文件。您可以通过setCache(int byteSize)接口设置缓存大小，范围为0-30K。例：BuglyLog.setCache(12 * 1024) //将Cache设置为12K
- 如果您没有使用BuglyLog接口，且初始化Bugly时isDebug参数设置为false，该Log功能将不会有新的资源占用；
- 为了方便开发者调试，当初始化Bugly的isDebug参数为true时，异常日志同时还会记录Bugly本身的日志。请在App发布时将其设置为false；
- 上报Log最大30K。

自定义日志可在跟踪日志中查看
![enter description here][4]

### Bug：BugShaker
**BugShaker：Shake to send a bug report by Email**

Specify BugShaker-Android as a dependency in your build.gradle file:
```gradle
dependencies {
    compile 'com.github.stkent:bugshaker:{latest-version}'
}
```
Configure the shared BugShaker instance in your custom Application class, then call assemble and start to begin listening for shakes:
```java
public class CustomApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        BugShaker.get(this)
                 .setEmailAddresses("someone@example.com")   // required
                 .setEmailSubjectLine("Custom Subject Line") // optional
                 .setAlertDialogType(AlertDialogType.NATIVE) // optional
                 .setLoggingEnabled(BuildConfig.DEBUG)       // optional
                 .setIgnoreFlagSecure(true)                  // optional
                 .assemble()                                 // required
                 .start();                                   // required
    }
}
```
It is recommended that logging always be disabled in production builds.

该库通过摇一摇利用Email上传BUG信息，信息包括：截屏（如果能够获取的话）、硬件信息、日期信息
通过适当的拓展修改，可以支持追加日志信息。

### Log：Timber
**Timber：A logger with a small, extensible API which provides utility on top of Android's normal Log class.**

in gradle.build
```gradle
compile 'com.jakewharton.timber:timber:4.5.1'
````

in Application.java

```java
package com.example.timber;

import android.app.Application;
import android.util.Log;
import timber.log.Timber;

import static timber.log.Timber.DebugTree;

public class ExampleApp extends Application {
  @Override public void onCreate() {
    super.onCreate();

    if (BuildConfig.DEBUG) {
      Timber.plant(new DebugTree());
    } else {
      Timber.plant(new CrashReportingTree());
    }
  }

  /** A tree which logs important information for crash reporting. */
  private static class CrashReportingTree extends Timber.Tree {
    @Override protected void log(int priority, String tag, String message, Throwable t) {
      if (priority == Log.VERBOSE || priority == Log.DEBUG) {
        return;
      }

      FakeCrashLibrary.log(priority, tag, message);

      if (t != null) {
        if (priority == Log.ERROR) {
          FakeCrashLibrary.logError(t);
        } else if (priority == Log.WARN) {
          FakeCrashLibrary.logWarning(t);
        }
      }
    }
  }
}

```
通过Timber.Tree的方式，可以植入第三方的日志逻辑，也可以对日志统一进入过滤处理。

### Statistics

Bugly已经有了基本的运营统计功能，如果不需要细化统计各功能的具体使用情况，基本上已经满足一般的运营分析需要了。

如果需要详细分析各功能点的使用情况，可以考虑接口Umeng统计功能。具体接入方法详见以下文档：http://dev.umeng.com/analytics/android-doc/integration

### 集成应用

在gradle.build中添加如下代码：
```gradle
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5'
    releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
    testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
    compile 'com.github.markzhai:blockcanary-android:1.5.0'
    compile 'com.jakewharton.timber:timber:4.5.1'
    compile 'com.tencent.bugly:crashreport:latest.release'
    compile 'com.tencent.bugly:nativecrashreport:latest.release'
    compile 'com.github.stkent:bugshaker:1.3.0'
```

自定义Application如下：
```java
package com.xhr.android.rframework;

import android.app.Application;
import android.os.Environment;
import android.util.Log;

import com.github.moduth.blockcanary.BlockCanary;
import com.github.moduth.blockcanary.BlockCanaryContext;
import com.github.stkent.bugshaker.BugShaker;
import com.github.stkent.bugshaker.flow.dialog.AlertDialogType;
import com.squareup.leakcanary.LeakCanary;
import com.tencent.bugly.crashreport.BuglyLog;
import com.tencent.bugly.crashreport.CrashReport;
import com.xhr.and.rframework.utils.LogUtils;

import java.io.File;

import timber.log.Timber;


/**
 * Created by xhrong on 2016/11/9.
 */
public class MyAppilcation extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        //内存泄漏监控
        if (LeakCanary.isInAnalyzerProcess(this)) {
            // This process is dedicated to LeakCanary for heap analysis.
            // You should not init your app in this process.
            return;
        }
        LeakCanary.install(this);

        // UI阻塞监控
        BlockCanary.install(this, new BlockCanaryContext()).start();

        //本地日志
        if (BuildConfig.DEBUG) {
            Timber.plant(new Timber.DebugTree());
        } else {
            Timber.plant(new CrashReportingTree());
        }

        //Bugly崩溃上传
        CrashReport.initCrashReport(getApplicationContext(), "b3838323f7", true);
        //Bugly日志上传，做为崩溃信息的附加信息
        BuglyLog.setCache(10 * 1024);

        //摇一摇上传BUG和当时的日志信息
        BugShaker.get(this)
                .setEmailAddresses("xhrong@iflytek.com")   // required
                .setEmailSubjectLine("Custom Subject Line") // optional
                .setLoggingEnabled(true)
                .setLogFilePath(Environment.getExternalStorageDirectory() + "/logs")
                .setAlertDialogType(AlertDialogType.APP_COMPAT)
                .assemble()
                .start();
    }

    /**
     * A tree which logs important information for crash reporting.
     */
    private static class CrashReportingTree extends Timber.Tree {
        @Override
        protected void log(int priority, String tag, String message, Throwable t) {
            if (priority == Log.VERBOSE || priority == Log.DEBUG) {
                return;
            }
            //TODO:自定义日志上传
        }
    }
}
```
  [1]: ./images/1491960900277.jpg " "
  [2]: ./images/1491961463413.jpg " "
  [3]: ./images/1491962641052.jpg " "
  [4]: ./images/1491978620983.jpg " "