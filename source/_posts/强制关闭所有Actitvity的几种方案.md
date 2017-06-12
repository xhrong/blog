---
title: 强制关闭所有Actitvity的几种方案 
tags: [Android,强制退出]
grammar_cjkRuby: true
categories: [Android]
date: 2017-06-04
---
### 通过广播

这种方式需要修改已有Activity的继承关系

```java
//自定义一个广播接收器,用来接收应用程序退出广播.
public class ExitAppReceiver extends BroadcastReceiver {
 
    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO Auto-generated method stub
        if (context != null) {
 
            if (context instanceof Activity) {
 
                ((Activity) context).finish();
            } else if (context instanceof FragmentActivity) {
 
                ((FragmentActivity) context).finish();
            } else if (context instanceof Service) {
 
                ((Service) context).stopSelf();
            }
        }
    }
}


//应用程序中所有Activity的基类
public class BaseActivity extends Activity  {
     
    private ExitAppReceiver exitReceiver = new ExitAppReceiver();
       //自定义退出应用Action,实际应用中应该放到整个应用的Constant类中.
    private static final String EXIT_APP_ACTION = "com.micen.exit_app";
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        registerExitReceiver();
    }
 
    private void registerExitReceiver() {
 
            IntentFilter exitFilter = new IntentFilter();
        exitFilter.addAction(EXIT_APP_ACTION);
        registerReceiver(exitReceiver, exitFilter);
    }
 
    private void unRegisterExitReceiver() {
 
        unregisterReceiver(exitReceiver);
    }
 
    @Override
    protected void onDestroy() {
        // TODO Auto-generated method stub
        super.onDestroy();
        unRegisterExitReceiver();
    }
 
    @Override
    protected void onStart() {
        super.onStart();
    }
 
    @Override
    protected void onStop() {
        super.onStop();
    }
}

//最后在要退出App的方法中添加以下发送广播代码即可.
Intent intent = new Intent();
intent.setAction(EXIT_APP_ACTION);
sendBroadcast(intent);

```

### 管理Activity列表

这种方式需要修改已有Activity的继承关系

```java
public class BaseActivity extends Activity {

    public static final String TAG = "BaseActivity";
    public static ArrayList<Activity> activityList = new ArrayList<Activity>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        activityList.add(this);
        Log.i(TAG, activityList.toString());
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        activityList.remove(this);
        Log.i(TAG, activityList.toString());
    }

    /**
     * 完完全全退出应用程序
     */

    public void exitApp() {
        if (activityList.size() > 0) {
            for (Activity activity : activityList) {
                activity.finish();
            }
            android.os.Process.killProcess(android.os.Process.myPid());
        }
    }
}
```
### 通过ActivityLifecycleCallbacks
这种方式不用调整已有Activity的继承关系，代码侵入性最低

```java
public  class AbsSuperApplication extends Application {

    /**
     * 维护Activity 的list
     */
    private static List<Activity> mActivitys = Collections
            .synchronizedList(new LinkedList<Activity>());

    @Override
    public void onCreate() {
        super.onCreate();
        registerActivityListener();
    }

    /**
     * @param activity 作用说明 ：添加一个activity到管理里
     */
    public void pushActivity(Activity activity) {
        mActivitys.add(activity);
        LogUtils.d("activityList:size:"+mActivitys.size());
    }

    /**
     * @param activity 作用说明 ：删除一个activity在管理里
     */
    public void popActivity(Activity activity) {
        mActivitys.remove(activity);
        LogUtils.d("activityList:size:"+mActivitys.size());
    }

    /**
     * get current Activity 获取当前Activity（栈中最后一个压入的）
     */
    public static Activity currentActivity() {
        if (mActivitys == null||mActivitys.isEmpty()) {
            return null;
        }
        Activity activity = mActivitys.get(mActivitys.size()-1);
        return activity;
    }

    /**
     * 结束当前Activity（栈中最后一个压入的）
     */
    public static void finishCurrentActivity() {
        if (mActivitys == null||mActivitys.isEmpty()) {
            return;
        }
        Activity activity = mActivitys.get(mActivitys.size()-1);
        finishActivity(activity);
    }

    /**
     * 结束指定的Activity
     */
    public static void finishActivity(Activity activity) {
        if (mActivitys == null||mActivitys.isEmpty()) {
            return;
        }
        if (activity != null) {
            mActivitys.remove(activity);
            activity.finish();
            activity = null;
        }
    }

    /**
     * 结束指定类名的Activity
     */
    public static void finishActivity(Class<?> cls) {
        if (mActivitys == null||mActivitys.isEmpty()) {
            return;
        }
        for (Activity activity : mActivitys) {
            if (activity.getClass().equals(cls)) {
                finishActivity(activity);
            }
        }
    }

    /**
     * 按照指定类名找到activity
     *
     * @param cls
     * @return
     */
    public static Activity findActivity(Class<?> cls) {
        Activity targetActivity = null;
        if (mActivitys != null) {
            for (Activity activity : mActivitys) {
                if (activity.getClass().equals(cls)) {
                    targetActivity = activity;
                    break;
                }
            }
        }
        return targetActivity;
    }

    /**
     * 结束所有Activity
     */
    public static void finishAllActivity() {
        if (mActivitys == null) {
            return;
        }
        for (Activity activity : mActivitys) {
            activity.finish();
        }
        mActivitys.clear();
    }

    /**
     * 退出应用程序
     */
    public  static void appExit() {
        try {
            LogUtils.e("app exit");
            finishAllActivity();
        } catch (Exception e) {
        }
    }


    private void registerActivityListener() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
            registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
                @Override
                public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                    /**
                     *  监听到 Activity创建事件 将该 Activity 加入list
                     */
                    pushActivity(activity);

                }

                @Override
                public void onActivityStarted(Activity activity) {

                }

                @Override
                public void onActivityResumed(Activity activity) {

                }

                @Override
                public void onActivityPaused(Activity activity) {

                }

                @Override
                public void onActivityStopped(Activity activity) {

                }

                @Override
                public void onActivitySaveInstanceState(Activity activity, Bundle outState) {

                }

                @Override
                public void onActivityDestroyed(Activity activity) {
                    if (null==mActivitys&&mActivitys.isEmpty()){
                        return;
                    }
                    if (mActivitys.contains(activity)){
                        /**
                         *  监听到 Activity销毁事件 将该Activity 从list中移除
                         */
                        popActivity(activity);
                    }
                }
            });
        }
    }

}

```