---
title: JVM概述
date: 2016-11-24 18:28:48
tags: [JVM]
categories: [JAVA]
---


## JVM概述
@(技术学习)[JVM, Java堆, Java栈]

[TOC]

### JVM的组成

#### JVM在JDK中的位置 
 ![Alt text](./1477555738109.png)
     从最底层的位置可以看出来JVM有多重要，而实际项目中JAVA应用的性能优化，OOM等异常的处理最终都得从JVM这儿来解决。 HotSpot是Oracle关于JVM的商标，区别于IBM，HP等厂商开发的JVM。Java HotSpot Client VM和Java HotSpot Server VM是JDK关于JVM的两种不同的实现，前者可以减少启动时间和内存占用，而后者则提供更加优秀的程序运行速度。在命令行，通过java -version可以查看关于当前机器JVM的信息。

![Alt text](./1477555771171.png)


 可以看出我装的是build 20.13-b02版本，HotSpot 类型Server模式的JVM。

#### JVM的组成
JVM由4大部分组成：ClassLoader，Runtime Data Area，Execution Engine，Native Interface。
![Alt text](./1477556615204.png)
 **ClassLoader:** 是负责加载class文件 ， class文件在文件开头有特定的文件标示 ， 并且ClassLoader只负责class文件的加载 ， 至于它是否可以运行，则由Execution Engine决定。

**Native Interface：** 是负责调用本地接口的。他的作用是调用不同语言的接口给JAVA用 ， 他会在Native Method Stack中记录对应的本地方法 ， 然后调用该方法时就通过Execution Engine加载对应的本地lib。原本多于用一些专业领域 ， 如JAVA驱动 ， 地图制作引擎等 ， 现在关于这种本地方法接口的调用已经被类似于Socket通信 ， WebService等方式取代。

**Execution Engine：** 是执行引擎 ， 也叫Interpreter。Class文件被加载后 ， 会把指令和数据信息放入内存中，Execution Engine则负责把这些命令解释给操作系统。

**Runtime Data Area：** 则是存放数据的， 分为五部分：Stack，Heap，Method Area，PC Register，Native Method Stack。几乎所有的关于java内存方面的问题 ， 都是集中在这块。

### Runtime Data Area

Runtime Data Area存放数据的， 分为五部分：Stack，Heap，Method Area，PC Register，Native Method Stack。几乎所有的关于java内存方面的问题 ， 都是集中在这块。下图是javapapers.com上关于Run-time Data Areas的描述：
![Alt text](./1477556158473.png)
![Alt text](./1477559142217.png)

可以看出它把Method Area划为了Heap的一部分，javapapers.com中认为Method Area是Heap的逻辑区域，但这取决于JVM的实现者，而HotSpot JVM中把Method Area划分为非堆内存，显然是不包含在Heap中的。下图是javacodegeeks.com中，2014年9月刊出的一片博文中关于Runtime Data Area的划分，其中指出，NonHeap包含PermGen和Code Cache，PermGen包含Method Area,而且PermGen在JAVA SE 8中已经不再用了。查阅资料（ https://abhirockzz.wordpress.com/2014/03/18/java-se-8-is-knocking-are-you-there/ ）得知，java8中PermGen已经从JVM中移除并被MetaSpace取代，java8中也不会见到OOM:PermGen Space的异常。目前Runtime Data Area可以用下图描述它的组成：
 ![Alt text](./1477556304394.png)


#### JVM Stack

虚拟机栈也是线程私有的，每创建一个线程，虚拟机就会为这个线程创建一个虚拟机栈，虚拟机栈表示Java方法执行的内存模型，每调用一个方法，就会生成一个栈帧（Stack Frame）用于存储方法的本地变量表、操作栈、方法出口等信息，当这个方法执行完后，就会弹出相应的栈帧。

如果请求的栈的深度过大，虚拟机可能会抛出StackOverflowError异常，如果虚拟机的实现中允许虚拟机栈动态扩展，当内存不足以扩展栈的时候，会抛出OutOfMemoryError异常。

Stack是java栈内存，它等价于C语言中的栈， 栈的内存地址是不连续的， 每个线程都拥有自己的栈。 栈 里面存储着的是StackFrame， 栈帧分为三部分：局部变量区（Local Variables）、操作数栈（Operand Stack）和帧数据区（Frame Data）。

**局部变量区（Loca Variables）**
用来存储一个类的方法中所用到的局部变量。局部变量区被组织一个一个从0开始的字数组，byte、short、char在存储前被转换为int，boolean也被转换为int，0表示false，非0表示true，long和double占据两个字长。

**操作数栈（Operand Stack）**
用于存储运算所需要的操作数和结果。操作数栈也被组织为一个字数组，但不同于局部变量区，它不是通过数组下标访问的，而是能过栈的Push和Pop操作，前一个操作Push进的数据可以被下一个操作Pop出来使用。

**帧数据区（Frame Data）**
用于保存解析器对于java字节码进行解释过程中需要的信息，包括：上次调用的方法、局部变量指针和 操作数栈的栈顶和栈底指针。部分的作用主要有三部分：
- 常量池中数据的解析
- 方法执行完后处理方法返回，恢复调用方现场
- 方法执行过程中抛出异常时的异常处理，存储有一个异常表，当出现异常时虚拟机查找相应的异常表看是否有对应的Catch语句，如果没有就抛出异常终止这个方法调用

StackFrame在方法被调用时创建，在某个线程中，某个时间点上，只有一个 框架是活跃的，该框架被称为Current Frame，而框架中的方法被称为Current Method，其中定义的类为Current Class。局部变量和操作数栈上的操作总是引用当前框架。当Stack Frame中方法被执行完之后，或者调用别的StackFrame中的方法时，则当前栈变为另外一个StackFrame。Stack的大小是由两种类 型，固定和动态的，动态类型的栈可以按照线程的需要分配。 下面两张图是关于栈之间关系以及栈和非堆内存的关系基本描述（来自 http://www.programering.com/a/MzM3QzNwATA.html ）：
 ![Alt text](./1477556909018.png)

![Alt text](./1477557006989.png)

#### Heap
Heap是用来存放对象信息的，和Stack不同，Stack代表着一种运行时的状态。换句话说，栈是运行时单位，解决程序该如何执行的问题，而堆是存储的单位， 解决数据存储的问题。Heap是伴随着JVM的启动而创建，负责存储所有对象实例和数组的。堆的存储空间和栈一样是不需要连续的，它分为Young Generation和Old Generation（也叫 Tenured  Generation ）两大部分。Young Generation分为Eden和Survivor，Survivor又分为From Space和 ToSpace。所有通过new创建的对象的内存都在堆中分配，其大小可以通过-Xmx和-Xms来控制。结构图如下所示：

![Alt text](./1477557700474.png)


**新生代：**新建的对象都是用新生代分配内存，Eden空间不足的时候，会把存活的对象转移到Survivor中，新生代大小可以由-Xmn来控制，也可以用-XX:SurvivorRatio来控制Eden和Survivor的比例
**旧生代：**用于存放新生代中经过多次垃圾回收仍然存活的对象

**PermanentSpace：**和Heap经常一起提及的概念是PermanentSpace，它是用来加载类对象的专门的内存区，是非堆内存，和Heap一起组成JAVA内存， 它包含MethodArea区（在没有CodeCache的HotSpot JVM实现里，则MethodArea就相当于GenerationSpace） 。 在JVM初始化的时候，我们可以通过参数来分别指定，PermanentSpace的大小、堆的大小、以及Young Generation和Old Generation的比值、Eden区和From Space的比值，从而来细粒度的适应不同JAVA应用的内存需求。

#### PC Register 
PC Register是程序计数寄存器，每个JAVA线程都有一个单独的PC Register，他是一个指针，由Execution Engine读取下一条指令。如果该线程正在执行java方法，则PC Register存储的是 正在被执行的指令的地址，如果是本地方法，PC Register的值没有定义。PC寄存器非常小，只占用一个字宽，可以持有一个returnAdress或者特定平台的一个指针。

#### Method Area
在HotSpot JVM的实现中属于非堆区，非堆区包括两部分：Permanet Generation和Code Cache，而Method Area属于Permanert Generation的一部分。Permanent Generation用来存储类信息，比如说： class definitions，structures，methods， field， method (data and code) 和 constants。Code Cache用来存储Compiled Code，即编译好的本地代码，在HotSpot JVM中通过JIT(Just In Time) Compiler生成，JIT是即时编译器，他是为了提高指令的执行效率，把字节码文件编译成本地机器代码，如下图：

![Alt text](./1477558200207.png)

引用一个经典的案例来理解Stack，Heap和Method Area的划分，就是Sring a="xx"；Stirng b="xx"，问是否a==b? 首先==符号是用来判断两个对象的引用地址是否相同，而在上面的题目中，a和b按理来说申请的是Stack中不同的地址，但是他们指向Method Area中Runtime Constant Pool的同一个地址，按照网上的解释，在a赋值为“xx”时，会在Runtime Contant Pool中生成一个String Constant，当b也赋值为“xx”时，那么会在常量池中查看是否存在值为“xx”的常量，存在的话，则把b的指针也指向“xx”的地址，而不是新生 成一个 String Constant 。我查阅了网络上大家关于 String Constant 的存储的说说法，存在略微差别的是，它存储在哪里，有人说Heap中会分配出一个常量池，用来存储常量，所有线程共享它。而有人说常量池是Method Area的一部分，而Method Area属于非堆内存，那怎么能说常量池存在于堆中？

其实两种理解都没错。Method Area的确从逻辑上讲可以是Heap的一部分，在某些JVM实现里从堆上开辟一块存储空间来记录常量是符合JVM常量池设计目的的，所以前一种说法没问 题。对于后一种说法，HotSpot JVM的实现中的确是把方法区划分为了非堆内存，意思就是它不在堆上。 我在HotSpot JVM做了个简单的实验，定义多个常量之后，程序抛出OOM：PermGen Space异常，印证了JVM实现中常量池是在Permanent Space中的说法。但是，我的JDK版本是1.6的。查阅资料，JDK1.7中InternedStrings已经不再存储在 PermanentSpace中，而是放到了Heap中；JDK8中PermanentSpace已经被完全移除，InternedStrings也被放 到了MetaSpace中（如果出现内存溢出，会报OOM:MetaSpace，这里有个关于两者性能对比的文章： http://blog.csdn.net/zhyhang/article/details/17246223 ）。 所 以，仁者见仁，智者见智，一个馒头足以引发血案，就算是同一个商家的JVM，毕竟JDK版本在更新，或许正如StackOverFlow上大神们所说，对 于理解JVM Runtime Data Area这一部分的划分逻辑，还是去看对应版本的JDK源码比较靠谱，或者是参考不同的版本JVM Specification（ http://docs.oracle.com/javase/specs/ ）。

#### Native Method Stack
 Native Method Stack是供本地方法（非java）使用的栈。每个线程持有一个Native Method Stack。

### JVM的运行原理

JVM是java的核心和基础，在java编译器和os平台之间的虚拟处理器。它是一种基于下层的操作系统和硬件平台并利用软件方法来实现的抽象的计算机，可以在上面执行java的字节码程序。
JVM是Java程序运行的容器,但是他同时也是操作系统的一个进程,因此他也有他自己的运行的生命周期,也有自己的代码和数据空间。JVM在整个jdk中处于最底层,负责与操作系统的交互,用来屏蔽操作系统环境,提供一个完整的Java运行环境,因此也就虚拟计算机.操作系统装入JVM是通过jdk中Java.exe来完成,通过下面4步来完成JVM环境。
1.创建JVM装载环境和配置, 调用操作系统API判断系统的CPU架构，根据对应CPU类型寻找位于JRE目录下的/lib/jvm.cfg文件
2.装载JVM.dll
3.初始化JVM.dll并挂接到JNIENV(JNI调用接口)实例
4.调用JNIEnv实例装载并处理class类。[2] 
![Alt text](./1477560551227.png)
java编译器只需面向JVM，生成JVM能理解的代码或字节码文件。Java源文件经编译器，编译成字节码程序，通过JVM将每一条指令翻译成不同平台机器码，通过特定平台运行。JVM执行程序的过程 ：
1.加载.class文件
2.管理并分配内存
3.执行垃圾收集

class文件是字节码文件，它按照JVM的规范，定义了变量，方法等的详细信息，JVM管理并且分配对应的内存来执 行程序，同时管理垃圾回收。直到程序结束，一种情况是JVM的所有非守护线程停止，一种情况是程序调用System.exit()，JVM的生命周期也结 束。

### 类加载器子系统（Class Loader）
类加载器子系统负责加载编译好的.class字节码文件，并装入内存，使JVM可以实例化或以其它方式使用加载后的类。JVM的类加载子系统支持在运行时的动态加载，动态加载的优点有很多，例如可以节省内存空间、灵活地从网络上加载类，动态加载的另一好处是可以通过命名空间的分隔来实现类的隔离，增强了整个系统的安全性。

#### ClassLoader的分类：
a.启动类加载器（BootStrap Class Loader）：负责加载rt.jar文件中所有的Java类，即Java的核心类都是由该ClassLoader加载。在Sun JDK中，这个类加载器是由C++实现的，并且在Java语言中无法获得它的引用。

b.扩展类加载器（Extension Class Loader）：负责加载一些扩展功能的jar包。

c.系统类加载器（System Class Loader）：负责加载启动参数中指定的Classpath中的jar包及目录，通常我们自己写的Java类也是由该ClassLoader加载。在Sun JDK中，系统类加载器的名字叫AppClassLoader。

d.用户自定义类加载器（User Defined Class Loader）：由用户自定义类的加载规则，可以手动控制加载过程中的步骤。

#### ClassLoader的工作原理
类加载分为装载、链接、初始化三步。
1、装载

通过类的全限定名和ClassLoader加载类，主要是将指定的.class文件加载至JVM。当类被加载以后，在JVM内部就以“类的全限定名+ClassLoader实例ID”来标明类。

在内存中，ClassLoader实例和类的实例都位于堆中，它们的类信息都位于方法区。

装载过程采用了一种被称为“双亲委派模型（Parent Delegation Model）”的方式，当一个ClassLoader要加载类时，它会先请求它的双亲ClassLoader（其实这里只有两个ClassLoader，所以称为父ClassLoader可能更容易理解）加载类，而它的双亲ClassLoader会继续把加载请求提交再上一级的ClassLoader，直到启动类加载器。只有其双亲ClassLoader无法加载指定的类时，它才会自己加载类。

双亲委派模型是JVM的第一道安全防线，它保证了类的安全加载，这里同时依赖了类加载器隔离的原理：不同类加载器加载的类之间是无法直接交互的，即使是同一个类，被不同的ClassLoader加载，它们也无法感知到彼此的存在。这样即使有恶意的类冒充自己在核心包（例如java.lang）下，由于它无法被启动类加载器加载，也造成不了危害。

由此也可见，如果用户自定义了类加载器，那就必须自己保障类加载过程中的安全。

2、链接

链接的任务是把二进制的类型信息合并到JVM运行时状态中去。

链接分为以下三步：

a.验证：校验.class文件的正确性，确保该文件是符合规范定义的，并且适合当前JVM使用。

b.准备：为类分配内存，同时初始化类中的静态变量赋值为默认值。

c.解析（可选）：主要是把类的常量池中的符号引用解析为直接引用，这一步可以在用到相应的引用时再解析。

3、初始化

初始化类中的静态变量，并执行类中的static代码、构造函数。

JVM规范严格定义了何时需要对类进行初始化：

a、通过new关键字、反射、clone、反序列化机制实例化对象时。

b、调用类的静态方法时。

c、使用类的静态字段或对其赋值时。

d、通过反射调用类的方法时。

e、初始化该类的子类时（初始化子类前其父类必须已经被初始化）。

f、JVM启动时被标记为启动类的类（简单理解为具有main方法的类）。

### JVM内存管理

![Alt text](./1477615516864.png)


#### 垃圾收集算法
##### 跟踪收集器
跟踪收集器采用的为集中式的管理方式，全局记录对象之间的引用状态，执行时从一些列GC  Roots的对象做为起点，从这些节点向下开始进行搜索所有的引用链，当一个对象到GC  Roots 没有任何引用链时，则证明此对象是不可用的。
下图中，对象Object6、Object7、Object8虽然互相引用，但他们的GC Roots是不可到达的，所以它们将会被判定为是可回收的对象。
![Alt text](./1477615288982.png)

可作为GC Roots 的对象包括：
虚拟机栈(栈帧中的本地变量表)中的引用对象。
方法区中的类静态属性引用的对象
方法区中的常量引用的对象
本地方法栈中JNI的引用对象。

主要有复制、标记清除、标记压缩三种实现算法。 
###### **1. 标记 - 清除算法** 
标记清除算法是最基础的收集算法，其他收集算法都是基于这种思想。标记清除算法分为“标记”和“清除”两个阶段：首先标记出需要回收的对象，标记完成之后统一清除对象。
它的主要缺点：
①.标记和清除过程效率不高 
②.标记清除之后会产生大量不连续的内存碎片。

![Alt text](./1477615301316.png)
![Alt text](./1477615313879.png)


###### **2. 复制算法** 
它将可用内存容量划分为大小相等的两块，每次只使用其中的一块。当这一块用完之后，就将还存活的对象复制到另外一块上面，然后在把已使用过的内存空间一次清理掉。这样使得每次都是对其中的一块进行内存回收，不会产生碎片等情况，只要移动堆订的指针，按顺序分配内存即可，实现简单，运行高效。
主要缺点：
内存缩小为原来的一半。
                            
![Alt text](./1477615328010.png)
![Alt text](./1477615334358.png)


###### **3. 标记  - 整理算法**
标记操作和“标记-清除”算法一致，后续操作不只是直接清理对象，而是在清理无用对象完成后让所有存活的对象都向一端移动，并更新引用其对象的指针。
主要缺点：
在标记-清除的基础上还需进行对象的移动，成本相对较高，好处则是不会产生内存碎片。

![Alt text](./1477615353531.png)
![Alt text](./1477615359322.png)


引用计数收集器
引用计数收集器采用的是分散式管理方式，通过计数器记录对象是否被引用。当计数器为0时说明此对象不在被使用，可以被回收。
主要缺点：
循环引用的场景下无法实现回收，例如下面的图中，ObjectC和ObjectB相互引用，那么ObjectA即便释放了对ObjectC、ObjectB的引用，也无法回收。sunJDK在实现GC时未采用这种方式。
![Alt text](./1477615371170.png)

#### 垃圾收集器
HotSpot JVM收集器
![Alt text](./1477615452569.png)

 上面有7中收集器，分为两块，上面为新生代收集器，下面是老年代收集器。如果两个收集器之间存在连线，就说明它们可以搭配使用。
##### Serial(串行GC)收集器
Serial收集器是一个新生代收集器，单线程执行，使用复制算法。它在进行垃圾收集时，必须暂停其他所有的工作线程(用户线程)。是Jvm client模式下默认的新生代收集器。对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。
##### ParNew(并行GC)收集器
ParNew收集器其实就是serial收集器的多线程版本，除了使用多条线程进行垃圾收集之外，其余行为与Serial收集器一样。
##### Parallel Scavenge(并行回收GC)收集器
Parallel Scavenge收集器也是一个新生代收集器，它也是使用复制算法的收集器，又是并行多线程收集器。parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而parallel Scavenge收集器的目标则是达到一个可控制的吞吐量。吞吐量= 程序运行时间/(程序运行时间 + 垃圾收集时间)，虚拟机总共运行了100分钟。其中垃圾收集花掉1分钟，那吞吐量就是99%。
##### Serial Old(串行GC)收集器
Serial Old是Serial收集器的老年代版本，它同样使用一个单线程执行收集，使用“标记-整理”算法。主要使用在Client模式下的虚拟机。
##### Parallel Old(并行GC)收集器
Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。
##### CMS(并发GC)收集器
CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。CMS收集器是基于“标记-清除”算法实现的，整个收集过程大致分为4个步骤：
①.初始标记(CMS initial mark)
②.并发标记(CMS concurrenr mark)
③.重新标记(CMS remark)
④.并发清除(CMS concurrent sweep)
     其中初始标记、重新标记这两个步骤任然需要停顿其他用户线程。初始标记仅仅只是标记出GC ROOTS能直接关联到的对象，速度很快，并发标记阶段是进行GC ROOTS 根搜索算法阶段，会判定对象是否存活。而重新标记阶段则是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间会被初始标记阶段稍长，但比并发标记阶段要短。
     由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以整体来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。
CMS收集器的优点：并发收集、低停顿，但是CMS还远远达不到完美，器主要有三个显著缺点：
CMS收集器对CPU资源非常敏感。在并发阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致引用程序变慢，总吞吐量下降。CMS默认启动的回收线程数是：(CPU数量+3) / 4。
CMS收集器无法处理浮动垃圾，可能出现“Concurrent Mode Failure“，失败后而导致另一次Full  GC的产生。由于CMS并发清理阶段用户线程还在运行，伴随程序的运行自热会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在本次收集中处理它们，只好留待下一次GC时将其清理掉。这一部分垃圾称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行，
即需要预留足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分内存空间提供并发收集时的程序运作使用。在默认设置下，CMS收集器在老年代使用了68%的空间时就会被激活，也可以通过参数-XX:CMSInitiatingOccupancyFraction的值来提供触发百分比，以降低内存回收次数提高性能。要是CMS运行期间预留的内存无法满足程序其他线程需要，就会出现“Concurrent Mode Failure”失败，这时候虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。所以说参数-XX:CMSInitiatingOccupancyFraction设置的过高将会很容易导致“Concurrent Mode Failure”失败，性能反而降低。
最后一个缺点，CMS是基于“标记-清除”算法实现的收集器，使用“标记-清除”算法收集后，会产生大量碎片。空间碎片太多时，将会给对象分配带来很多麻烦，比如说大对象，内存空间找不到连续的空间来分配不得不提前触发一次Full  GC。为了解决这个问题，CMS收集器提供了一个-XX:UseCMSCompactAtFullCollection开关参数，用于在Full  GC之后增加一个碎片整理过程，还可通过-XX:CMSFullGCBeforeCompaction参数设置执行多少次不压缩的Full  GC之后，跟着来一次碎片整理过程。
##### G1收集器
G1(Garbage First)收集器是JDK1.7提供的一个新收集器，G1收集器基于“标记-整理”算法实现，也就是说不会产生内存碎片。还有一个特点之前的收集器进行收集的范围都是整个新生代或老年代，而G1将整个Java堆(包括新生代，老年代)。
