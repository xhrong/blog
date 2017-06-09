---
title: Java类加载机制 
tags: [ClassLoader]
grammar_cjkRuby: true
categories: [Java]
date: 2017-05-28
---

## ClassLoader是什么

java.lang.ClassLoader类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个Java 类，即 java.lang.Class类的一个实例。

ClassLoader提供了一系列的方法，比较重要的方法如：

 ![enter description here][1]

 
## JVM中类加载器的层次结构

Java 中的类加载器大致可以分成两类，一类是系统提供的，另外一类则是由 Java 应用开发人员编写的。 

### 引导类加载器（bootstrap class loader）：

它用来加载 Java 的核心库(jre/lib/rt.jar)，是用原生C++代码来实现的，并不继承自java.lang.ClassLoader。

加载扩展类和应用程序类加载器，并指定他们的父类加载器，在java中获取不到。 

### 扩展类加载器（extensions class loader）：

它用来加载 Java 的扩展库(jre/ext/*.jar)。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。 

### 系统类加载器（system class loader）：

它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。

### 自定义类加载器（custom class loader）：

除了系统提供的类加载器以外，开发人员可以通过继承 java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求。


## 双亲委派模型

### 原理介绍
 ![enter description here][2]
 备注：上图表示的是包含关系，不是继承关系
 
![enter description here][3]

ClassLoader使用的是双亲委托模型来搜索类的，每个ClassLoader实例都有一个父类加载器的引用（不是继承的关系，是一个包含的关系），虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其它ClassLoader实例的的父类加载器。当一个ClassLoader实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。
 
### 为什么要使用双亲委托这种模型：避免类的重复加载
因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义的类型，这样会存在非常大的安全隐患，而双亲委托的方式，就可以避免这种情况，因为String已经在启动时就被引导类加载器（Bootstrcp ClassLoader）加载，所以用户自定义的ClassLoader永远也无法加载一个自己写的String，除非你改变JDK中ClassLoader搜索类的默认算法。

###  如何判定两个class是相同：同一个类加载器加载的&类名相同
 JVM在判定两个class是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，JVM才认为这两个class是相同的。就算两个class是同一份class字节码，如果被两个不同的ClassLoader实例所加载，JVM也会认为它们是两个不同class。比如网络上的一个Java类org.classloader.simple.NetClassLoaderSimple，javac编译之后生成字节码文件NetClassLoaderSimple.class，ClassLoaderA和ClassLoaderB这两个类加载器并读取了NetClassLoaderSimple.class文件，并分别定义出了java.lang.Class实例来表示这个类，对于JVM来说，它们是两个不同的实例对象，但它们确实是同一份字节码文件，如果试图将这个Class实例生成具体的对象进行转换时，就会抛运行时异常**java.lang.ClassCastException**，提示这是两个不同的类型。
 
	 
### 双亲委派关系不是继承关系！

 - 双亲委派在代码实现上，采用的关联关系，即下层ClassLoader保存了上层ClassLoader的引用。这种关系和ClassLoader子类的继承关系并没有直接关系，也不存在对等关系。
![enter description here][4]

- ClassLoader类体系中，BootstrapClassLoader是非JAVA的，独立存在。ClassLoader类本身是一个抽象类，是一切ClassLoader子类的最父类。而做为JAVA预置的ExtClassLoader和AppClassLoaer，其直接父类是我们常用来动态加载JAR包的URLClassLoader。我们自定义ClassLoader时通常全直接继承抽象的ClassLoader类做为直接父类。
![enter description here][5]



## 自定义ClassLoader
 因为Java中提供的默认ClassLoader，只加载指定目录下的jar和class，如果我们想加载其它位置的类或jar时，比如：我要加载网络上的一个class文件，通过动态加载到内存之后，要调用这个类中的方法实现我的业务逻辑。在这样的情况下，默认的ClassLoader就不能满足我们的需求了，所以需要定义自己的ClassLoader。
### 步骤
定义自已的类加载器分为两步：
1、继承java.lang.ClassLoader
2、重写父类的findClass方法
父类有那么多方法，为什么偏偏只重写findClass方法？      因为JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法，当在loadClass方法中搜索不到类时，loadClass方法就会调用findClass方法来搜索类，所以我们只需重写该方法即可。如没有特殊的要求，一般不建议重写loadClass搜索类的算法。下图是API中ClassLoader的loadClass方法：
![enter description here][6]

### Sample:自定义一个NetworkClassLoader，用于加载网络上的class文件
```java
package classloader;

import java.io.ByteArrayOutputStream;
import java.io.InputStream;
import java.net.URL;

/**
 * 加载网络class的ClassLoader
 */
public class NetworkClassLoader extends ClassLoader {
	
	private String rootUrl;

	public NetworkClassLoader(String rootUrl) {
		this.rootUrl = rootUrl;
	}

	@Override
	protected Class<?> findClass(String name) throws ClassNotFoundException {
		Class clazz = null;
		//if (clazz == null) {	//检查该类是否已被加载过
			byte[] classData = getClassData(name);	//根据类的二进制名称,获得该class文件的字节码数组
			if (classData == null) {
				throw new ClassNotFoundException();
			}
			clazz = defineClass(name, classData, 0, classData.length);	//将class的字节码数组转换成Class类的实例
		//} 
		return clazz;
	}

	private byte[] getClassData(String name) {
		InputStream is = null;
		try {
			String path = classNameToPath(name);
			URL url = new URL(path);
			byte[] buff = new byte[1024*4];
			int len = -1;
			is = url.openStream();
			ByteArrayOutputStream baos = new ByteArrayOutputStream();
			while((len = is.read(buff)) != -1) {
				baos.write(buff,0,len);
			}
			return baos.toByteArray();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (is != null) {
			   try {
			      is.close();
			   } catch(IOException e) {
			      e.printStackTrace();
			   }
			}
		}
		return null;
	}

	private String classNameToPath(String name) {
		return rootUrl + "/" + name.replace(".", "/") + ".class";
	}

}
```

### 关于defineClass
defineClass将class的字节码数组转换成Class类的实例。其内部过程如下：

JVM将类加载过程分为三个步骤：装载（Load），链接（Link）和初始化(Initialize)

![enter description here][7]

1) 装载：

　　查找并加载类的二进制数据；

2)链接：

　　验证：确保被加载类信息符合JVM规范、没有安全方面的问题。

　　准备：为类的静态变量分配内存，并将其初始化为默认值。

　　解析：把虚拟机常量池中的符号引用转换为直接引用。

3)初始化：

　　为类的静态变量赋予正确的初始值。

## 应用场景

### 动态加载JAR包或.class文件

#### 两种作用域

**第一种方法：**
```java
URL url= file.toURI().toURL();//将File类型转为URL类型，file为jar包路径  
URLClassLoader urlClassLoader=new URLClassLoader(new URL[] {url});  
Class c=urlClassLoader.loadClass("类名");  
```
此种方法是构造一个新的URLClassLoader对象，利用该对象加载指定路径下的jar包，此种方法只能在此处加载该jar包中的类，调用其方法，不能在程序中的其他地方调用。如果将urlClassLoader声明为静态的则可以在其它地方调用

**第二种方法：**
```java
URL url= file.toURI().toURL();//将File类型转为URL类型，file为jar包路径  
//得到系统类加载器  
URLClassLoader urlClassLoader= (URLClassLoader) ClassLoader.getSystemClassLoader();  
//因为URLClassLoader中的addURL方法的权限为protected所以只能采用反射的方法调用addURL方法  
Method add = URLClassLoader.class.getDeclaredMethod("addURL", new Class[] { URL.class });                                 
add.setAccessible(true);  
add.invoke(urlClassLoader, new Object[] {url });  
Class c=Class.forName("类名");  
//或者  
Class c=urlClassLoader.loadClass("类名");  
```
此种方法是得到系统类加载器，利用该加载器加载指定路径下的jar包，此种方法与java命令中的javac -cp是同等效果，都能在当前运行环境中改变CLASSPATH，所以利用该方法加载jar包后，在程序任一地方都能加载该jar包中的类，调用其方法。

导入多个jar包时，第一种方法加载jar包中的类时，需知道加载该jar包的URLClassLoader，第二种方法则不需要，可使用Class.forName("类名");加载类

#### 基于Interface或顶层父类
1、定义接口
```java
package loader;

public interface HelloIface {

    public String hello();
    
    public String sayHi(String name);

}
```
2、实现接口

在其他插件类实现此接口，并导出为jar，如D:/tmp/test.jar
```java
package loader;

public class HelloImpl implements HelloIface{

	@Override
	public String hello() {
		return "hello,JAVA世界";
	}

	@Override
	public String sayHi() {
		return "Hi,JAVA World " + name;
	}
}
```
3、动态加载类
```java
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;

public class LoadJar {
	public static void main(String[] args) {
		String classPath = "loader.HelloImpl";// Jar中的所需要加载的类的类名
		String jarPath = "file:///D:/tmp/test.jar";// jar所在的文件的URL

		loadJar1(classPath, jarPath);
		loadClass(classPath);
		loadJar2(classPath, jarPath);
		loadClass(classPath);
	}

	public static void loadJar1(String classPath, String jarPath) {
		ClassLoader cl;
		try {
			// 从Jar文件得到一个Class加载器
			cl = new URLClassLoader(new URL[] { new URL(jarPath) });
			// 从加载器中加载Class
			Class<?> c = cl.loadClass(classPath);
			// 从Class中实例出一个对象
			HelloIface impl = (HelloIface) c.newInstance();
			// 调用Jar中的类方法
			System.out.println(impl.hello());
			System.out.println(impl.sayHi("zhangsan"));
			
			try {
				HelloIface impl2 = (HelloIface) Class.forName(classPath)
						.newInstance();
				System.out.println(impl2.hello());
			} catch (ClassNotFoundException e) {
				System.out.println("非系统加载器加载的JAR,不能通过Class.forName使用");
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}


	
	public static void loadClass(String classPath){
		try {
			HelloIface impl2 = (HelloIface) Class.forName(classPath)
					.newInstance();
			System.out.println(impl2.hello());
		} catch (Exception e) {
			System.out.println("非系统加载器加载的JAR,不能通过Class.forName使用");
		}
	}
}
```
#### 基于反射

在使用第三方jar和.class文件时，无法定义契约的情况下，可以在分析jar或.class文件的元信息，通过反射实现功能调用。
```java
public static void loadJar2(String classPath, String jarPath) {
		URLClassLoader urlLoader = (URLClassLoader) ClassLoader.getSystemClassLoader();
		Class<URLClassLoader> sysClass = URLClassLoader.class;
		try {
			//改变方法的可见性（即通过反映访问本来不可以访问的方法）
			Method method = sysClass.getDeclaredMethod("addURL", new Class[] { URL.class });
			method.setAccessible(true);
			method.invoke(urlLoader, new URL(jarPath));
			
			Class<?> objClass = urlLoader.loadClass(classPath);
			Object instance = objClass.newInstance();
			Method method2 = objClass.getDeclaredMethod("sayHi", new Class[]{ String.class});
			System.out.println(method2.invoke(instance, "zhangsan"));
	
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```
### 插件技术

插件技术是一种通用技术，在Java语境下，该技术是借助ClassLoader动态加载机制来实现的。


  [1]: ./images/1495588683692.jpg " "
  [2]: ./images/1495589908935.jpg " "
  [3]: ./images/1495588515133.jpg " "
  [4]: ./images/1495588811914.jpg " "
  [5]: ./images/1495588760460.jpg " "
  [6]: ./images/1495590863815.jpg " "
  [7]: ./images/1495594387684.jpg " "