---
title: Android热更新技术综述 
tags: [HotFix,Tinker]
grammar_cjkRuby: true
categories: [Android]
date: 2017-04-23
---


### 原理
#### ClassLoader & Multidex

Java在运行时加载对应的类是通过ClassLoader来实现的，ClassLoader本身是一个抽象来，Android中使用PathClassLoader类作为Android的默认的类加载器， PathClassLoader其实实现的就是简单的从文件系统中加载类文件。PathClassLoade本身继承自BaseDexClassLoader，BaseDexClassLoader重写了findClass方法，该方法是ClassLoader的核心
```java
@Override
protected Class<?> findClass(String name) throws ClassNotFoundException {
    List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
    Class c = pathList.findClass(name, suppressedExceptions);
    if (c == null) {
        ClassNotFoundException cnfe = new ClassNotFoundException("Didn't find class \"" + name + "\" on path: " + pathList);
        for (Throwable t : suppressedExceptions) {
            cnfe.addSuppressed(t);
        }
        throw cnfe;
    }
    return c;
}
```
BaseDexClassLoader将findClass方法委托给了pathList对象的findClass方法，pathList对象是在BaseDexClassLoader的构造函数中new出来的， 它的类型是DexPathList。看下DexPathList.findClass源码是如何做的：

```
public Class findClass(String name, List<Throwable> suppressed) {
    for (Element element : dexElements) {
        DexFile dex = element.dexFile;

        if (dex != null) {
            Class clazz = dex.loadClassBinaryName(name, definingContext, suppressed);
            if (clazz != null) {
                return clazz;
            }
        }
    }
    if (dexElementsSuppressedExceptions != null) {
        suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
    }
    return null;
}
```
直接就是遍历dexElements列表，然后通过调用element.dexFile对象上的loadClassBinaryName方法来加载类，如果返回值不是null，就表示加载类成功，会将这个Class对象返回。 而dexElements对象是在DexPathList类的构造函数中完成初始化的。

```java
this.dexElements = makeDexElements(splitDexPath(dexPath), optimizedDirectory,
                                           suppressedExceptions);
```									   							 
makeDexElements所做的事情就是遍历我们传递来的dexPath，然后一次加载每个dex文件。

上面分析了Android中的类的加载的流程，可以看出来DexPathList对象中的dexElements列表是类加载的一个核心，一个类如果能被成功加载，那么它的dex一定 会出现在dexElements所对应的dex文件中，并且dexElements中出现的顺序也很重要，在dexElements前面出现的dex会被优先加载，一旦Class被加载成功， 就会立即返回，也就是说，我们的如果想做hotpatch，一定要保证我们的hotpacth dex文件出现在dexElements列表的前面。

要实现热更新，就需要我们在运行时去更改PathClassLoader.pathList.dexElements，由于这些属性都是private的，因此需要通过反射来修改。另外，构造我们自己的dex文件 所对应的dexElements数组的时候，我们也可以采取一个比较取巧的方式，就是通过构造一个DexClassLoader对象来加载我们的dex文件，并且调用一次dexClassLoader.loadClass(dummyClassName); 方法，这样，dexClassLoader.pathList.dexElements中，就会包含我们的dex，通过把dexClassLoader.pathList.dexElements插入到系统默认的classLoader.pathList.dexElements列表前面，就可以让系统优先加载我们的dex中的类，从而可以实现热更新了。下面展示一部分代码
```java
private static synchronized Boolean injectAboveEqualApiLevel14(
            String dexPath, String defaultDexOptPath, String nativeLibPath, String dummyClassName) {
    Log.i(TAG, "--> injectAboveEqualApiLevel14");
    PathClassLoader pathClassLoader = (PathClassLoader) DexInjector.class.getClassLoader();
    DexClassLoader dexClassLoader = new DexClassLoader(dexPath, defaultDexOptPath, nativeLibPath, pathClassLoader);
    try {
        dexClassLoader.loadClass(dummyClassName);
        Object dexElements = combineArray(
                getDexElements(getPathList(pathClassLoader)),
                getDexElements(getPathList(dexClassLoader)));


        Object pathList = getPathList(pathClassLoader);
        setField(pathList, pathList.getClass(), "dexElements", dexElements);
    } catch (Throwable e) {
        e.printStackTrace();
        return false;
    }
    Log.i(TAG, "<-- injectAboveEqualApiLevel14 End.");
    return true;
}
```
 
**总结：要实现热更新的关键是将Patch注入到ClassLoader里，并保证其先于原始的模块被加载。方法是通过Multidex实现分包，利用反射动态修改ClassLoader的属性**

基于这一原理的方案有：HotFix（RocooFix）、Tinker、Nuwa

#### Native Hack
AndFix不同于QQ空间超级补丁技术和微信Tinker通过增加或替换整个DEX的方案，提供了一种运行时在Native修改Filed指针的方式，实现方法的替换，达到即时生效无需重启，对应用无性能消耗的目的。

原理图如下：

![enter description here][1]
### 方案

![enter description here][2]



### 参考资料
http://blog.csdn.net/hb707934728/article/details/52152739


  [1]: ./images/1493694410962.jpg " "
  [2]: ./images/1493099184966.jpg " "