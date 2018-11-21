---
title: Android应用防卸载和设备管理器权限获取
tags: [防卸载,设备管理器,权限]
grammar_cjkRuby: true
categories: [Android]
date: 2018-11-21
---
### 设备管理器权限范围
```xml
<device-admin xmlns:android="http://schemas.android.com/apk/res/android">
  <uses-policies>
    <limit-password />
    <watch-login />
    <reset-password />
    <force-lock />
    <wipe-data />
    <expire-password />
    <encrypted-storage />
    <disable-camera />
  </uses-policies>
</device-admin>
```
立即锁屏 Lock device immediately
锁屏前的不活动时间 Maximum inactivity time lock  
指定存储区域加密 Require storage encryption 
擦拭数据(恢复出厂) Wipe the device's data   
禁用相机 Disable camera

开启手机PIN或密码 Password enabled
密码最小长度 Minimum password length
密码是字母数字 Alphanumeric password required
密码必须是字母数字符号混合 Complex password required 
密码有效期 Password expiration timeout
密码失败次数 Maximum failed password attempts 

![enter description here][1]


### 设备管理器激活

1、 创建 MyReceiver 类,必须继承DeviceAdminReceiver类
```java
public class MyReceiver extends DeviceAdminReceiver {       
    @Override
    public void onEnabled(Context context, Intent intent) {
        //设备管理可用
    }

    @Override
    public void onDisabled(Context context, Intent intent) {
        //设备管理不可用
    }

    @Override
    public void onPasswordChanged(Context context, Intent intent) {     
    }
    ...
}
```
2、在AndroidManifest.xml
```xml
<receiver 
        android:name=".MyReceiver"
        android:label="用户可看的权限标题"
        android:description="用户可看的权限描述"
        android:permission="android.permission.BIND_DEVICE_ADMIN"> 
        必须拥有的权限,确保只有系统可以与DeviceAdminReceiver子类交互
    <meta-data
            android:name="android.app.device_admin"
            android:resource="@xml/device_xxx" />

    <intent-filter>
        <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
        当用户启用设备管理员APP时,将DeviceAdminReceiver子类设置为接收器
    </intent-filter>        
</receiver>
```
3、在res/xml/device_xxx.xml
在xml中声明APP需要的设备管理策略,如密码,锁屏,擦除数据,加密等等
```xml
<device-admin xmlns:android="http://schemas.android.com/apk/res/android">
  <uses-policies>
    <limit-password />
    <watch-login />
    <reset-password />
    <force-lock />
    <wipe-data />
    <expire-password />
    <encrypted-storage />
    <disable-camera />
  </uses-policies>
</device-admin>
```
4、激活APP设备管理
```java
ComponentName myReceiverName = new ComponentName(this,MyReceiver.class);    
Intent intent = new Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN);    
intent.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN, myReceiverName);
intent.putExtra(DevicePolicyManager.EXTRA_ADD_EXPLANATION,"给用户介绍文字");
```
5、取消APP设备管理,卸载APP
```java
DevicePolicyManager dpm = (DevicePolicyManager) getSystemService(DEVICE_POLICY_SERVICE);
ComponentName myReceiverName = new ComponentName(this,MyReceiver.class);
// 取消设备管理
dpm.removeActiveAdmin(myReceiverName);
// 卸载
Intent intent = new Intent();
intent.setAction("android.intent.action.VIEW");
intent.addCategory("android.intent.category.DEFAULT");
intent.setData(Uri.parse("package:"+getPackageName()));
startActivity(intent);
```

### 锁屏/禁用相机/擦除SD卡数据/恢复出厂
```java
DevicePolicyManager dpm = (DevicePolicyManager) getSystemService(DEVICE_POLICY_SERVICE);
ComponentName myReceiverName = new ComponentName(this,MyReceiver.class);
// 2.锁屏/擦除数据/   
if(dpm.isAdminActive(myReceiverName)){  //是否激活APP设备管理
    // 立刻锁屏
    dpm.lockNow();
    // 设置自动锁屏,不活动时间
    dpm.setMaximumTimeToLock(myReceiverName, timeMs);
    // 重置锁屏密码       
    dpm.resetPassword("123456", 0);

    // 禁用相机
    dpm.setCameraDisabled(myReceiverName, false);

    // 擦除SD卡数据
    //dpm.wipeData(DevicePolicyManager.WIPE_EXTERNAL_STORAGE);

    // 恢复出厂
    //dpm.wipeData(0);
}else{
    Toast.makeText(this, "还没有激活APP设备管理", 0).show();
}
```

### 使用dpm激活设备管理器并防卸载

Usage :
usage: dpm [subcommand] [options]
usage: dpm set-device-owner <COMPONENT>
usage: dpm set-profile-owner <COMPONENT> <USER_ID>

dpm set-device-owner: Sets the given component as active admin, and its package as device owner.
dpm set-profile-owner: Sets the given component as active admin and profile owner for an existing user.

Example：

![enter description here][2]

注意：Also notice that this tool is working only if no account is set for the user (make sure no account is set in Settings > Accounts) before its use.

**执行后，设备管理器会自动激活，并且无法解除激活。因为卸载应用必须先解除激活，从而保证应用不会被卸载。**

![enter description here][3]

可通过下面解除激活，然后卸载应用

![enter description here][4]


### 参考资料

1.https://github.com/googlesamples/android-AppRestrictionEnforcer
2.http://florent-dupont.blogspot.com/2015/01/android-shell-command-dpm-device-policy.html
3.https://stackoverflow.com/questions/21183328/how-to-make-my-app-a-device-owner
4.https://blog.csdn.net/ice_eyes/article/details/52951343?locationNum=2&fps=1
5.https://blog.csdn.net/qq_32115439/article/details/71270479


  [1]: ./images/1542799168143.jpg
  [2]: ./images/1542799814414.jpg
  [3]: ./images/1542799950085.jpg
  [4]: ./images/1542799869851.jpg