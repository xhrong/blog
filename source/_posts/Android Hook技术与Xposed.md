---
title: Android Hook技术与Xposed
tags: [Hook,Xposed]
grammar_cjkRuby: true
categories: [Android]
date: 2017-05-20
---


## 原理

### 代理

代理对象持有了被代理对象的引用，当被代理方法执行的时候，在执行的前后添加一些逻辑或是修改返回的结果。

举个简单的例子如下。先抽象个简单的狗。
```java
public interface IDog {
    void eat();
    void drink();
}
```
创建一个具体的黑色狗狗
```java
public class BlankDog implements IDog{
    @Override
    public void eat() {
        Log.i("BlankDog", "----  eat  -----");
    }

    @Override
    public void drink() {
        Log.i("BlankDog", "----  drink  -----");
    }
}
```
最重要的代理黑狗
```java
public class DogProxy implements IDog{
    private BlankDog mBlankDog;

    public DogProxy(BlankDog blankDog){
        this.mBlankDog = blankDog;
    }

    @Override
    public void eat() {
        Log.i("","----  在吃之前先撒个欢  -----");
        mBlankDog.eat();
    }

    @Override
    public void drink() {
        mBlankDog.drink();
        Log.i("","----  在喝之后撒个欢  -----");
    }
}
```
发现所有黑狗会的技能，代理狗都会，而且还做的一样好，甚至在吃喝之前还撒个欢，有了自己的个性。

**Hook技术中，使用代理替换原有对象，从而实现在原有流程上添加额外的业务。
而这个“替换”是通过反射来实现的。**

### 反射

这里面涉及到几个的反射方法

Class.forName()

通过类名获取Class对象。

setAccessible()

改变访问对象的可见性，常常用来访问private属性的对象。

invoke(Object receiver, Object... args)

通过对象和参数列表执行方法。

get(Object object)

获取Feild的值。

**Hook技术中，通过反射来实现原有对象的替换，以增加额外的业务**

## 过程
Hook技术实现，有三个过程，即：编写代理类，实现业务扩展；寻找代理点，实现原对象的替换；确定代理时机，执行对换替换。

其中第二步是关键。


### 编写代理类
这一步较为简单，通过代理类，扩展原对象的功能，添加新业务。

```java
public class OnClickListenerProxy implements View.OnClickListener{
    private View.OnClickListener object;
    private HookListenerContract.OnClickListener mlistener;

    public OnClickListenerProxy(View.OnClickListener object, HookListenerContract.OnClickListener listener){
        this.object = object;
        this.mlistener = listener;
    }

    @Override
    public void onClick(View v) {
        if(mlistener != null) mlistener.doInListener(v);
        if(object != null) object.onClick(v);
    }
}
```
### 寻找代理点

这一步最关键，需要深入分析源码，找到合适的被代理对象，并通过反射技术用代理对象替换原对象。

```java
 mClassView = Class.forName("android.view.View");
 Method method = mClassView.getDeclaredMethod("getListenerInfo");
 method.setAccessible(true);
 Object listenerInfoObject = method.invoke(view);

 Class mClassListenerInfo = Class.forName("android.view.View$ListenerInfo");

 Field feildOnClickListener = mClassListenerInfo.getDeclaredField("mOnClickListener");
 feildOnClickListener.setAccessible(true);
 View.OnClickListener mOnClickListenerObject = (View.OnClickListener) feildOnClickListener.get(listenerInfoObject);

View.OnClickListener onClickListenerProxy = new OnClickListenerProxy(mOnClickListenerObject, mListenerManager.mOnClickListener);

feildOnClickListener.set(listenerInfoObject, onClickListenerProxy);

```
### 确定代理时机

这一步也很关键，合适的时机能够保证代理的正确执行。

```java
public class BaseActivity extends AppCompatActivity {
    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);

        ListenerManager.Builer builer = new ListenerManager.Builer();
        builer.buildOnClickListener(new HookListenerContract.OnClickListener() {
            @Override
            public void doInListener(View v) {
                Toast.makeText(BaseActivity.this, "单击时我执行", Toast.LENGTH_SHORT).show();
            }
        }).buildOnLongClickListener(new HookListenerContract.OnLongClickListener() {
            @Override
            public void doInListener(View v) {
                Toast.makeText(BaseActivity.this, "长按时我执行", Toast.LENGTH_SHORT).show();
            }
        }).buildOnFocusChangeListener(new HookListenerContract.OnFocusChangeListener() {
            @Override
            public void doInListener(View v, boolean hasFocus) {
                Toast.makeText(BaseActivity.this, "焦点变化时我执行", Toast.LENGTH_SHORT).show();
            }
        });
        HookCore.getInstance().startHook(this, ListenerManager.create(builer));
    }
}
```
## Xposed
  Xposed是Android平台上较为出名的一个开源框架。在这个框架下，我们可以加载很多插件App，这些插件App可以直接或间接操纵普通应用甚至系统上的东西。Xposed原理上是Hook Android 系统的核心进程Zygote来达到修改程序运行过程和结果。
  
  **Xposed能够Hook第三方应用**
  
  **Xposed需要Root权限**


## 参考文献

http://www.open-open.com/lib/view/open1477296293290.html
http://blog.csdn.net/zhongwn/article/details/54381337
http://blog.csdn.net/zhangmiaoping23/article/details/52315745
http://blog.csdn.net/zhangmiaoping23/article/details/52315815