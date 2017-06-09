---
title: Android的类加载机制
tags: [ClassLoader]
grammar_cjkRuby: true
categories: [Android]
date: 2017-05-29
---
## ClassLoader是什么
参考：Java类加载机制

## Android中的类加载器的层次结构
![enter description here][1]

- BootClassLoader : 主要用于加载系统的类，包括java和android系统的类库

- URLClassLoader： 只能用于加载jar文件，但是由于 dalvik 不能直接识别jar，所以在 Android 中无法使用这个加载器

- BaseDexClassLoader：作为PathClassLoader和DexClassLoader的父类，封装了共用的核心业务逻辑

- PathClassLoader : 主要用于加载应用内中的类, 它加载的路径是固定的, 因此无法指定

- DexClassLoader : 可以用于加载任意路径的zip,jar或者apk文件, 可以实现动态加载
## 双亲委派模型

参考：Java类加载机制

```java
        ClassLoader loader = getClassLoader();
        while (loader!=null){
            System.out.println("LLLLL:"+loader);
            loader = loader.getParent();
        }
```
输入结果：
![enter description here][2]

## Android类加载过程


![enter description here][3]

![enter description here][4]

![enter description here][5]

### 1、生成Element数组dexElements，生成pathList

BaseDexClassLoader的构造方法如下：
```java
public BaseDexClassLoader(String dexPath, File optimizedDirectory,
        String libraryPath, ClassLoader parent) {
    super(parent);
    this.pathList = new DexPathList(this, dexPath, libraryPath, optimizedDirectory);
}
```
BaseDexClassLoader中的构造方法包含4个参数，他们分别是：

(1) dexPath : 指目标类所在的APK或jar文件的路径.类装载器将从该路径中寻找指定的目标类,该类必须是APK或jar的全路径.如果要包含多个路径,路径之间必须使用特定的分割符分隔,分隔符为File.pathSeparator(“:”).

(2) optimizedDirectory : 由于dex文件被包含在APK或者Jar文件中,因此在装载目标类之前需要先从APK或Jar文件中解压出dex文件,该参数就是指定解压出的dex文件存放的路径.

(3) libraryPath : 指目标类中所使用的C/C++库存放的路径

(4) parent : 是指该装载器的父装载器

在BaseDexClassLoader中生成DexPathList类的对象pathList,在DexPathList的构造方法中调用DexPathList类中的makePathElements()方法，在makePathElements()方法中会调用splitDexPath()方法，如果dexPath包含多个路径，splitDexPath()方法能够提取出File.pathSeparator分隔的多个路径，并将多个路径放入List中返回，然后makePathElements()方法会调用loadDexFile()方法遍历List中每个包含dex的文件生成DexFile,用生成的DexFile生成Element对象，用这些Element对象构成Element数组dexElements。

DexPathList.java中的loadDexFile()方法源码如下：
```java
private static DexFile loadDexFile(File file, File optimizedDirectory)
        throws IOException {
    if (optimizedDirectory == null) {
        return new DexFile(file);
    } else {
        String optimizedPath = optimizedPathFor(file, optimizedDirectory);
        return DexFile.loadDex(file.getPath(), optimizedPath, 0);
    }
}
```
一个应用程序中所有类加载到内存中，就是在makePathElements执行loadDexFile()方法来实现的。

optimizedDirectory是一个内部存储路径，用来缓存我们需要加载的dex文件的。DexClassLoader可以指定自己的optimizedDirectory，所以它可以加载外部的dex，因为这个dex会被复制到内部路径的optimizedDirectory；而PathClassLoader没有optimizedDirectory，所以它只能加载内部的dex，这些大都是存在系统中已经安装过的apk里面的。

它们的不同之处总结如下是：

DexClassLoader可以加载jar/apk/dex，可以从SD卡中加载未安装的apk；

PathClassLoader只能加载系统中已经安装过的apk，Android中大部分类的加载默认采用此类

### 2、pathList调用BaseDexClassLoader中的findClass()方法

调用BaseDexClassLoader的findClass()方法，在此方法，pathList调用DexPathList中的findClass()方法，BaseDexClassLoader的findClass()方法如下：
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

### 3、调用DexPathList中的findClass()方法

pathList是DexPathList类型的一个对象，DexPathList中的findClass()方法源码如下：
```java
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
DexPathList的findClass()方法，遍历dexElements元素的DexFile实例，也就是遍历所有加载过的dex文件，一个个调用loadClassBinaryName()方法，看能不能找到我们想要的类，如果可以，返回这个类。

DexFile.java中的loadClassBinaryName()方法：
```java
public Class loadClassBinaryName(String name, ClassLoader loader, List<Throwable> suppressed) {
    return defineClass(name, loader, mCookie, suppressed);
}

private static Class defineClass(String name, ClassLoader loader, Object cookie,
List<Throwable> suppressed) {
    Class result = null;
    try {
        result = defineClassNative(name, loader, cookie);
    } catch (NoClassDefFoundError e) {
        if (suppressed != null) {
            suppressed.add(e);
        }
    } catch (ClassNotFoundException e) {
        if (suppressed != null) {
            suppressed.add(e);
        }
    }
    return result;
}

private static native Class defineClassNative(String name, 
    ClassLoader loader, Object cookie) throws ClassNotFoundException, NoClassDefFoundError;
```
4、返回

将加载的类返回ClassLoader类的loadClass()方法。


## 应用场景
![enter description here][6]
### 应用分包（MultiDEX）

解决Android方法数限制问题

详见：[Android方法数限制相关问题][7]

### 热更新

在MultiDex基础上，通过Hook技术和反射技术， dexElements数组头部插入新的DEX文件路径， 实现Dex文件的动态替换，从而修复线上BUG。
（这只是热更新的一种方法）

详见：[Android热更新技术综述][8]

### 插件化（热部署）

在MultiDex基础上，通过Hook技术和反射技术， dexElements数组动态插入新的DEX文件路径， 实现Dex文件的动态加载，功能的扩展。
（这只是插件化的一种方法）


### 应用加固
解决壳程序调起原程序的关键。

详见：[Android应用加固技术综述][9]

## 参考资料
http://www.itwendao.com/article/detail/253115.html
http://www.jianshu.com/p/3afa47e9112e
https://segmentfault.com/a/1190000005113493


  [1]: ./images/1495604330409.jpg " "
  [2]: ./images/1495608141427.jpg " "
  [3]: ./images/1495604531656.jpg " "
  [4]: ./images/1495608455115.jpg " "
  [5]: ./images/1495605755419.jpg " "
  [6]: ./images/1495595796415.jpg " "
  [7]: http://xhrong.github.io/2016/11/22/Android%E6%96%B9%E6%B3%95%E6%95%B0%E9%99%90%E5%88%B6%E7%9B%B8%E5%85%B3%E9%97%AE%E9%A2%98/
  [8]: http://xhrong.github.io/2017/04/23/Android%E7%83%AD%E6%9B%B4%E6%96%B0%E6%8A%80%E6%9C%AF%E7%BB%BC%E8%BF%B0/
  [9]: http://xhrong.github.io/2017/04/29/Android%20%E5%BA%94%E7%94%A8%E5%8A%A0%E5%9B%BA%E6%8A%80%E6%9C%AF%E7%BB%BC%E8%BF%B0/