---
title: AspectJ在Android中的应用 
tags: [AspectJ,AOP]
grammar_cjkRuby: true
date: 2017-10-13
categories: [Android]
---


## AOP的概念

### 术语
AOP（Aspect-Oriented Programming，面向方面编程）利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为 “Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低 模块间的耦合度，并有利于未来的可操作性和可维护性。AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为； 那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手 将这些剖开的切面复原，不留痕迹。

使用“横切”技术，AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横 切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处都基本相似。比如权限认证、日志、事务处理。Aop 的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。AOP的核心思想就是“将应用程序中的商业逻辑同对其提供支持的通用服务进行分离。”

实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的 方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。然而殊途同归，实现AOP的技术特性却是相同的，分别为：

- join point（连接点）：是程序执行中的一个精确执行点，例如类中的一个方法。它是一个抽象的概念，在实现AOP时，并不需要去定义一个join point。
- point cut（切入点）：本质上是一个捕获连接点的结构。在AOP中，可以定义一个point cut，来捕获相关方法的调用。
- advice（通知）：是point cut的执行代码，是执行“方面”的具体逻辑。
- aspect（方面）：point cut和advice结合起来就是aspect，它类似于OOP中定义的一个类，但它代表的更多是对象间横向的关系。
- introduce（引入）：为对象引入附加的方法或属性，从而达到修改对象结构的目的。有的AOP工具又将其称为mixin。
- target（目标类）：是指那些将使用advice的类，一般是指独立的那些商务模型。
- proxy（代理类）：使用了proxy的模式。是指应用了advice的对象，看起来和target对象很相似。
- weaving（织入）：是指应用aspects到一个target对象创建proxy对象的过程：complie time，classload time，runtime

上述的技术特性组成了基本的AOP技术，大多数AOP工具均实现了这些技术。它们也可以是研究AOP技术的基本术语。

![enter description here][1]

### 织入时机

- **At run-time:** your code has to explicitly ask for the enhanced code, e.g. by using a Dynamic Proxy (this is arguably not true code injection). Anyway here is an example I created for testing it.

- **At load-time:** the modification are performed when the target classes are being loaded by Dalvik or ART. Byte-code or Dex-code weaving.

- **At build-time:** you add an extra step to your build process to modify the compiled classes before packaging and deploying your application. Source-code weaving.

![enter description here][2]

### 应用场景

- Logging
- Persistance
- Performance monitoring
- Data Validation
- Caching
- Many others


## AspectJ
### Aspectj是什么
官方网站的的介绍是这样的：
a seamless aspect-oriented extension to the Javatm programming language（一种基于Java平台的面向切面编程的语言）
Java platform compatible（兼容Java平台，可以无缝扩展）
easy to learn and use（易学易用）

### Aspectj能做什么
clean modularization of crosscutting concerns, such as error checking and handling, synchronization, context-sensitive behavior, performance optimizations, monitoring and logging, debugging support, and multi-object protocols。
         大意是说：干净的模块化横切关注点（也就是说单纯，基本上无侵入），如错误检查和处理，同步，上下文敏感的行为，性能优化，监控和记录，调试支持，多目标的协议。

### 其他AOP框架

Jboss Aop：

Spring Aop：Spring自己原生的Aop，只能用一个词来形容：难用。　你需要实现大量的接口，继承大量的类，所以spring aop一度被千夫所指。这于他的无侵入，低耦合完全冲突。不过Spring对开源的优秀框架，组建向来是采用兼容，并入的态度。所以，后来的Spring 就提供了Aspectj支持，也就是我们后来所说的基于纯POJO的Aop。

**区别：** Spring Aop采用的动态织入，而Aspectj是静态织入。静态织入：指在编译时期就织入，即：编译出来的class文件，字节码就已经被织入了。动态织入又分静动两种，静则指织入过程只在第一次调用时执行；动则指根据代码动态运行的中间状态来决定如何操作，每次调用Target的时候都执行。有不清楚的同学，可以自己补下基础的代理知识。

## Android中集成AspectJ
### 引入AspectJ框架

```gradle
apply plugin: 'com.android.application'
import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main

// 有省略

final def log = project.logger
final def variants = project.android.applicationVariants
variants.all { variant ->
    if (!variant.buildType.isDebuggable()) {
        log.debug("Skipping non-debuggable build type '${variant.buildType.name}'.")
        return;
    }

    JavaCompile javaCompile = variant.javaCompile
    javaCompile.doLast {
        String[] args = ["-showWeaveInfo",
                         "-1.8",
                         "-inpath", javaCompile.destinationDir.toString(),
                         "-aspectpath", javaCompile.classpath.asPath,
                         "-d", javaCompile.destinationDir.toString(),
                         "-classpath", javaCompile.classpath.asPath,
                         "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]
        log.debug "ajc args: " + Arrays.toString(args)

        MessageHandler handler = new MessageHandler(true);
        new Main().run(args, handler);
        for (IMessage message : handler.getMessages(null, true)) {
            switch (message.getKind()) {
                case IMessage.ABORT:
                case IMessage.ERROR:
                case IMessage.FAIL:
                    log.error message.message, message.thrown
                    break;
                case IMessage.WARNING:
                    log.warn message.message, message.thrown
                    break;
                case IMessage.INFO:
                    log.info message.message, message.thrown
                    break;
                case IMessage.DEBUG:
                    log.debug message.message, message.thrown
                    break;
            }
        }
    }
}

// 有省略

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:24.2.1'
    testCompile 'junit:junit:4.12'
    compile files('libs/aspectjrt.jar')
}
```

以上操作可以封装成gradle插件的形成，方便集成。具体参见Hugo。


### 确定Pointcut

确定Pointcut有两种方式，一种基于注解，一种基于类正则表达式。

#### 基于注解确定Pointcut
1、定义注解
``` java
/**
 * Copyright (C) 2014 android10.org. All rights reserved.
 * @author Fernando Cejas (the android10 coder)
 */
package org.android10.gintonic.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Indicates that the annotated method is being traced (debug mode only) and
 * will use {@link android.util.Log} to print debug data:
 * - Method name
 * - Total execution time
 * - Value (optional string parameter)
 */
@Retention(RetentionPolicy.CLASS)
@Target({ ElementType.CONSTRUCTOR, ElementType.METHOD })
public @interface DebugTrace {}

```

2、设置Advice
```java
 private static final String POINTCUT_METHOD =
      "execution(@org.android10.gintonic.annotation.DebugTrace * *(..))";

  private static final String POINTCUT_CONSTRUCTOR =
      "execution(@org.android10.gintonic.annotation.DebugTrace *.new(..))";

  @Pointcut(POINTCUT_METHOD)
  public void methodAnnotatedWithDebugTrace() {}

  @Pointcut(POINTCUT_CONSTRUCTOR)
  public void constructorAnnotatedDebugTrace() {}



  @Around("methodAnnotatedWithDebugTrace() || constructorAnnotatedDebugTrace() || methodAnnotated() || methodAnootatedWith()")
  public Object weaveJoinPoint(ProceedingJoinPoint joinPoint) throws Throwable {
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    String className = methodSignature.getDeclaringType().getSimpleName()+"x";
    String methodName = methodSignature.getName();

    Object[] paramValues = joinPoint.getArgs();
    String[] paramNames = ((CodeSignature) joinPoint.getStaticPart()
            .getSignature()).getParameterNames();

    for(int i=0;i<paramNames.length;i++){
      Log.e("param",paramNames[i]+","+paramValues[i]);

    }
    if(methodName.contains("test")){
      paramValues[0]=20;
      joinPoint.proceed( paramValues);
    }

    final StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    Object result = joinPoint.proceed();
    stopWatch.stop();

    DebugLog.log(className, buildLogMessage(methodName, stopWatch.getTotalTimeMillis()));

    return result;
  }

```

3、使用注解标注Jointpoint

```java

    @DebugTrace
    private void test(int i){
        Log.e("dddd","ddddd"+i);
    }
```

#### 基于类正则表达示确定Pointcut
1、确定正则匹配

```java
  //进行类似于正则表达式的匹配，被匹配到的方法都会被截获
  ////截获任何包中以类名以Activity、Layout结尾，并且该目标类和当前类是一个Object的对象的所有方法
  private static final String PC_METHOD =
          "execution(* *..CoreApiClient+.*(..))";
  //精确截获MyFrameLayou的onMeasure方法
  private static final String PC_CALL = "call(* org.android10.viewgroupperformance.component.MyFrameLayout.onMeasure(..))";

  private static final String PC_METHOD_MAINACTIVITY = "execution(* *..OnClickListener.onClick(..))";

  //切点，ajc会将切点对应的Advise编织入目标程序当中
  @Pointcut(PC_METHOD)
  public void methodAnnotated() {}

  @Pointcut(PC_METHOD_MAINACTIVITY)
  public void methodAnootatedWith(){}
```

![enter description here][3]

在我的例子中，我使用了execution，也就是以方法执行时为切点，触发Aspect类。而execution里面的字符串是触发条件，也是具体的切点。我来解释一下参数的构成。“execution(@com.example.administrator.aspectjdemo.AspectJAnnotation  \* \*(..)”这个条件是所有加了AspectJAnnotation注解的方法或属性都会是切点，范围比较广。
“\*\*”表示是任意包名
“..”表示任意类型任意多个参数
“com.example.administrator.aspectjdemo.AspectJAnnotation”这是我的项目包名下需要指定类的绝对路径
那知道了这些基本上就能定义自己想要的切点了。比如我只想在MainActivity的onCreate()方法执行时作为切点。那么字符串应该是“\* com.example.administrator.aspectjdemo.MainActivity.onCreate(..)”，那么这个切定就很明确，只有一个。不同的切点可以用与或来连接。看清楚哦，前面有个\*号。因为onCreate方法时编译时执行，所以在回调时传入的参数必须是父类JoinPoint。上面除了execution还有其他很多切点类型。比如call方法回调时，get和set分别是获取和执行时。等等，就不举例了。

2、设置Advice

同“基于注解确定Pointcut”

### 设置Advice
![enter description here][4]

after和before没有返回值，但是around的目标是替代原JPoint的，所以它一般会有返回值，而且返回值的类型需要匹配被选中的JPoint。

**通过Advice，我们能够截获入参，修改参数值，调整返回值等，达到过滤、验证、拦截、记录等效果**
 
 具体参考“ 基于注解确定Pointcut -> 设置Advice”
 
 ### demo
 
 https://github.com/xhrong/AndAOP
 
  
  ### 参考文献
  
  http://blog.csdn.net/sw5131899/article/details/53885957
  http://blog.csdn.net/innost/article/details/49387395


  [1]: ./images/1508227845590.jpg
  [2]: ./images/1508228008994.jpg
  [3]: ./images/1508229368860.jpg
  [4]: ./images/1508229938049.jpg
 