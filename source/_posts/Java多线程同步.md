---
title: Java多线程同步 
tags: [Java,多线程,同步]
date: 2016-12-26
grammar_cjkRuby: true
categories: [Java]
---

## Java多线程同步

### 进程与线程

####  进程和线程区别

1、定义

**进程：** 是具有一定独立功能的程序关于某个数据集合上的一次运行活动，进程是系统进行资源分配和调度的一个独立单位。

**线程：** 是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位。线程自己基本上不拥有系统资源，只拥有一点在运行中必不可少的资源(如程序计数器，一组寄存器和栈)，但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。

2、 区别

相对进程而言，线程是一个更加接近于执行体的概念，它可以与同进程中的其他线程共享数据，但拥有自己的栈空间，拥有独立的执行序列。

进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。

线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。

一个程序至少有一个进程，一个进程至少有一个线程。

#### Java进程的建立方法

Java.lang.Runtime.exec 方法和 Java.lang.ProcessBuilder.start 方法都可以创建一个本地的进程，然后返回代表这个进程的 Java.lang.Process 引用。

1、Runtime.exec 方法建立一个本地进程

该方法在 JDK1.5 中，可以接受 6 种不同形式的参数传入。
```java
 Process exec(String command) 
 Process exec(String [] cmdarray) 
 Process exec(String [] cmdarrag, String [] envp) 
 Process exec(String [] cmdarrag, String [] envp, File dir) 
 Process exec(String cmd, String [] envp) 
 Process exec(String command, String [] envp, File dir)
 ```
他们主要的不同在于传入命令参数的形式，提供的环境变量以及定义执行目录。

2、ProcessBuilder.start 方法来建立一个本地的进程

如果希望在新创建的进程中使用当前的目录和环境变量，则不需要任何配置，直接将命令行和参数传入 ProcessBuilder 中，然后调用 start 方法，就可以获得进程的引用。
Process p = new ProcessBuilder("command", "param").start();
也可以先配置环境变量和工作目录，然后创建进程。
```java
 ProcessBuilder pb = new ProcessBuilder("command", "param1", "param2"); 
 Map<String, String> env = pb.environment(); 
 env.put("VAR", "Value"); 
 pb.directory("Dir"); 
 Process p = pb.start();
 ```
可以预先配置 ProcessBuilder 的属性是通过 ProcessBuilder 创建进程的最大优点。而且可以在后面的使用中随着需要去改变代码中 pb 变量的属性。如果后续代码修改了其属性，那么会影响到修改后用 start 方法创建的进程，对修改之前创建的进程实例没有影响。

#### 线程状态

![线程状态][1]
　

 - 新生状态（New）： 当一个线程的实例被创建即使用new关键字和Thread类或其子类创建一个线程对象后，此时该线程处于新生(new)状态，处于新生状态的线程有自己的内存空间，但该线程并没有运行，此时线程还不是活着的（not alive）；

 - 就绪状态（Runnable）： 通过调用线程实例的start()方法来启动线程使线程进入就绪状态(runnable)；处于就绪状态的线程已经具备了运行条件，但还没有被分配到CPU即不一定会被立即执行，此时处于线程就绪队列，等待系统为其分配CPCU，等待状态并不是执行状态； 此时线程是活着的（alive）；

 - 运行状态（Running）： 一旦获取CPU(被JVM选中)，线程就进入运行(running)状态，线程的run()方法才开始被执行；在运行状态的线程执行自己的run()方法中的操作，直到调用其他的方法而终止、或者等待某种资源而阻塞、或者完成任务而死亡；如果在给定的时间片内没有执行结束，就会被系统给换下来回到线程的等待状态；此时线程是活着的（alive）；

 - 阻塞状态（Blocked）：通过调用join()、sleep()、wait()或者资源被暂用使线程处于阻塞(blocked)状态；处于Blocking状态的线程仍然是活着的（alive）

 - 死亡状态（Dead）：当一个线程的run()方法运行完毕或被中断或被异常退出，该线程到达死亡(dead)状态。此时可能仍然存在一个该Thread的实例对象，当该Thready已经不可能在被作为一个可被独立执行的线程对待了，线程的独立的call stack已经被dissolved。一旦某一线程进入Dead状态，他就再也不能进入一个独立线程的生命周期了。对于一个处于Dead状态的线程调用start()方法，会出现一个运行期(runtime exception)的异常；处于Dead状态的线程不是活着的（not alive）。
 
 
### JAVA多线程的几种实现方式
#### 1、继承Thread类，并重写run函数
（1）定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体。

（2）创建Thread子类的实例，即创建了线程对象。

（3）调用线程对象的start()方法来启动该线程。

```java
 /**
 * 使用继承java.lang.Thread类的方式创建一个线程
 */
public class ThreadTest extends Thread {

    /**
     * 重写（Override）run()方法 JVM会自动调用该方法
     */
    public void run() {
        System.out.println("I'm running!");
    }
}
```

**下面是Thread类中常用的方法：**
 - start方法：用来启动一个线程，当调用start方法后，系统才会开启一个新的线程来执行用户定义的子任务，在这个过程中，会为相应的线程分配需要的资源。
- run方法：是不需要用户来调用的，当通过start方法启动一个线程之后，当线程获得了CPU执行时间，便进入run方法体去执行具体的任务。注意，继承Thread类必须重写run方法，在run方法中定义具体要执行的任务。

- sleep方法：有两个重载版本：
sleep(long millis)     //参数为毫秒
sleep(long millis,int nanoseconds)    //第一参数为毫秒，第二个参数为纳秒

  sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务。
  sleep方法不会释放锁，也就是说如果当前线程持有对某个对象的锁，则即使调用sleep方法，其他线程也无法访问这个对象。
  注意，如果调用了sleep方法，必须捕获InterruptedException异常或者将该异常向上层抛出。当线程睡眠时间满后，不一定会立即得到执行，因为此时可能CPU正在执行其他的任务。所以说调用sleep方法相当于让线程进入阻塞状态。

- yield方法：调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样不会释放锁。但是yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。

  注意，调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。

- join方法：有三个重载版本：
join()
join(long millis)     //参数为毫秒
join(long millis,int nanoseconds)    //第一参数为毫秒，第二个参数为纳秒
 　
  假如在main线程中，调用thread.join方法，则main方法会等待thread线程执行完毕或者等待一定的时间。如果调用的是无参join方法，则等待thread执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的时间。
  实际上调用join方法是调用了Object的wait方法，wait方法会让线程进入阻塞状态，并且会释放线程占有的锁，并交出CPU执行权限。由于wait方法会让线程释放对象锁，所以join方法同样会让线程释放对一个对象持有的锁。具体的wait方法使用在后面文章中给出。

- interrupt方法：顾名思义，即中断的意思。单独调用interrupt方法可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程；另外，通过interrupt方法和isInterrupted()方法来停止正在运行的线程。
  通过interrupt方法可以中断处于阻塞状态的线程。interrupt方法不能中断正在运行中的线程。

　　但是如果配合isInterrupted()能够中断正在运行的线程，因为调用interrupt方法相当于将中断标志位置为true，那么可以通过调用isInterrupted()判断中断标志是否被置位来中断线程的执行。

　　但是一般情况下不建议通过这种方式来中断线程，一般会在MyThread类中增加一个属性 isStop来标志是否结束while循环，然后再在while循环中判断isStop的值。
  
``` java

    class MyThread extends Thread{
	
        private volatile boolean isStop = false;
        @Override
        public void run() {
            int i = 0;
            while(!isStop){
                i++;
            }
        }
         
        public void setStop(boolean stop){
            this.isStop = stop;
        }
		
    }
```
  
  那么就可以在外面通过调用setStop方法来终止while循环。


- stop方法：已经是一个废弃的方法，它是一个不安全的方法。因为调用stop方法会直接终止run方法的调用，并且会抛出一个ThreadDeath错误，如果线程持有某个对象锁的话，会完全释放锁，导致对象状态不一致。所以stop方法基本是不会被用到的。


**以下是关系到线程属性的几个方法：**

- getId：得到线程ID

- getName和setName：得到或者设置线程名称。

- getPriority和setPriority：获取和设置线程优先级。

- setDaemon和isDaemon：设置线程是否成为守护线程和判断线程是否是守护线程。
守护线程和用户线程的区别在于：守护线程依赖于创建它的线程，而用户线程则不依赖。举个简单的例子：如果在main线程中创建了一个守护线程，当main方法运行完毕之后，守护线程也会随着消亡。而用户线程则不会，用户线程会一直运行直到其运行完毕。在JVM中，像垃圾收集器线程就是守护线程。

- currentThread()用来获取当前线程。


#### 2、实现Runnable接口，并写run函数
（1）定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。

（2）创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。

（3）调用线程对象的start()方法来启动该线程。

```java
/**
 * 通过实现Runnable接口创建一个线程
 */
public class ThreadTest implements Runnable {
    public void run() {
            System.out.println("I'm running!");
    }
}
```
#### 3、实现Callable接口，并重写call函数
（1）创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。

（2）创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。

（3）使用FutureTask对象作为Thread对象的target创建并启动新线程。

（4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值（ *get方法会阻塞当前线程，直到获取返回结果*）


　**1.使用Callable+Future获取执行结果**
```java
public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        executor.shutdown();
         
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
         
        System.out.println("主线程在执行任务");
         
        try {
            System.out.println("task运行结果"+result.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
         
        System.out.println("所有任务执行完毕");
    }
}
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程在进行计算");
        Thread.sleep(3000);
        int sum = 0;
        for(int i=0;i<100;i++)
            sum += i;
        return sum;
    }
}
```
 　　执行结果：
```
子线程在进行计算
主线程在执行任务
task运行结果4950
所有任务执行完毕
```

**2.使用Callable+FutureTask获取执行结果**

```java
public class Test {
    public static void main(String[] args) {
        //第一种方式
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        executor.submit(futureTask);
        executor.shutdown();
         
        //第二种方式，注意这种方式和第一种方式效果是类似的，只不过一个使用的是ExecutorService，一个使用的是Thread
        /*Task task = new Task();
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        Thread thread = new Thread(futureTask);
        thread.start();*/
         
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
         
        System.out.println("主线程在执行任务");
         
        try {
            System.out.println("task运行结果"+futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
         
        System.out.println("所有任务执行完毕");
    }
}
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程在进行计算");
        Thread.sleep(3000);
        int sum = 0;
        for(int i=0;i<100;i++)
            sum += i;
        return sum;
    }
}
```

如果为了可取消性而使用 Future 但又不提供可用的结果，则可以声明 Future<?> 形式类型、并返回 null 作为底层任务的结果。

#### 4、Thread、Runnable、Callable创建线程方式的对比

采用实现Runnable、Callable接口的方式创见多线程时，优势是：
- 线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。
- 在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。

劣势是：
- 编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

使用继承Thread类的方式创建多线程时优势是：
- 编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。

劣势是：
- 线程类已经继承了Thread类，所以不能再继承其他父类。

#### 5、线程池

##### Thread的弊端

new Thread的弊端
- 每次new Thread新建对象性能差。
- 线程缺乏统一管理，可能无限制新建线程，相互之间竞争，及可能占用过多系统资源导致死机或oom。
- 缺乏更多功能，如定时执行、定期执行、线程中断。

相比new Thread，Java提供的四种线程池的好处在于：
- 重用存在的线程，减少对象创建、消亡的开销，性能佳。
- 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。
- 提供定时执行、定期执行、单线程、并发数控制等功能。

##### 四种线程池
Java通过Executors提供四种线程池，分别为：
1、newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
2、newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
3、newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
4、newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO，LIFO，优先级)执行。

######  newCachedThreadPool
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。示例代码如下：

```java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();  
for (int i = 0; i < 10; i++) {  
    final int index = i;  
    try {  
        Thread.sleep(index * 1000);  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
  
    cachedThreadPool.execute(new Runnable() {  
  
        @Override  
        public void run() {  
            System.out.println(index);  
        }  
    });  
}  
```

线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。
 
###### newFixedThreadPool
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。示例代码如下：
```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);  
for (int i = 0; i < 10; i++) {  
    final int index = i;  
    fixedThreadPool.execute(new Runnable() {  
        @Override  
        public void run() {  
            try {  
                System.out.println(index);  
                Thread.sleep(2000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
        }  
    });  
}  
```
因为线程池大小为3，每个任务输出index后sleep 2秒，所以每两秒打印3个数字。
定长线程池的大小最好根据系统资源进行设置。如Runtime.getRuntime().availableProcessors()。可参考PreloadDataCache。
 
###### newScheduledThreadPool
创建一个定长线程池，支持定时及周期性任务执行。延迟执行示例代码如下：
```java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);  
scheduledThreadPool.schedule(new Runnable() {  
  
    @Override  
    public void run() {  
        System.out.println("delay 3 seconds");  
    }  
}, 3, TimeUnit.SECONDS);  
```
表示延迟3秒执行。
 
定期执行示例代码如下：
```java
scheduledThreadPool.scheduleAtFixedRate(new Runnable() {  
  
    @Override  
    public void run() {  
        System.out.println("delay 1 seconds, and excute every 3 seconds");  
    }  
}, 1, 3, TimeUnit.SECONDS);  
```
表示延迟1秒后每3秒执行一次。
ScheduledExecutorService比Timer更安全，功能更强大。
 
###### newSingleThreadExecutor
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。示例代码如下：
```java
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();  
for (int i = 0; i < 10; i++) {  
    final int index = i;  
    singleThreadExecutor.execute(new Runnable() {  
  
        @Override  
        public void run() {  
            try {  
                System.out.println(index);  
                Thread.sleep(2000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
        }  
    });  
}  
```
结果依次输出，相当于顺序执行各个任务

### 线程同步的几种方法

#### synchronized

在Java中，每一个对象都拥有一个锁标记（monitor），也称为监视器，多线程同时访问某个对象时，线程只有获取了该对象的锁才能访问。

在Java中，可以使用synchronized关键字来标记一个方法或者代码块，当某个线程调用该对象的synchronized方法或者访问synchronized代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。


##### synchronized方法

- 当一个线程正在访问一个对象的synchronized方法，那么其他线程不能访问该对象的其他synchronized方法。这个原因很简单，因为一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所以无法访问该对象的其他synchronized方法。

- 当一个线程正在访问一个对象的synchronized方法，那么其他线程能访问该对象的非synchronized方法。这个原因很简单，访问非synchronized方法不需要获得该对象的锁，假如一个方法没用synchronized关键字修饰，说明它不会使用到临界资源，那么其他线程是可以访问这个方法的，

- 如果一个线程A需要访问对象object1的synchronized方法fun1，另外一个线程B需要访问对象object2的synchronized方法fun1，即使object1和object2是同一类型），也不会产生线程安全问题，因为他们访问的是不同的对象，所以不存在互斥问题。

##### synchronized代码块

synchronized代码块类似于以下这种形式：

```java
    synchronized(synObject) {
         
    }
```
　　当在某个线程中执行这段代码块，该线程会获取对象synObject的锁，从而使得其他线程无法同时访问该代码块。

　　synObject可以是this，代表获取当前对象的锁，也可以是类中的一个属性，代表获取该属性的锁。

　    synchronized代码块使用起来比synchronized方法要灵活得多。因为也许一个方法中只有一部分代码只需要同步，如果此时对整个方法用synchronized进行同步，会影响程序执行效率。而使用synchronized代码块就可以避免这个问题，synchronized代码块可以实现只对需要同步的地方进行同步。

　　另外，每个类也会有一个锁，它可以用来控制对static数据成员的并发访问。并且如果一个线程执行一个对象的非static synchronized方法，另外一个线程需要执行这个对象所属类的static synchronized方法，此时不会发生互斥现象，因为访问static synchronized方法占用的是类锁，而访问非static synchronized方法占用的是对象锁，所以不存在互斥现象。
  
　　从反编译获得的字节码可以看出，synchronized代码块实际上多了monitorenter和monitorexit两条指令。monitorenter指令执行时会让对象的锁计数加1，而monitorexit指令执行时会让对象的锁计数减1，其实这个与操作系统里面的PV操作很像，操作系统里面的PV操作就是用来控制多个线程对临界资源的访问。对于synchronized方法，执行中的线程识别该方法的 method_info 结构是否有 ACC_SYNCHRONIZED 标记设置，然后它自动获取对象的锁，调用方法，最后释放锁。如果有异常发生，线程自动释放锁。
	  
　　有一点要注意：对于synchronized方法或者synchronized代码块，当出现异常时，JVM会自动释放当前线程占用的锁，因此不会由于异常导致出现死锁现象。
  

#### Lock
  
##### synchronized的缺陷

我们了解到如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里获取锁的线程释放锁只会有两种情况：

　　1）获取锁的线程执行完了该代码块，然后线程释放对锁的占有；

　　2）线程执行发生异常，此时JVM会让线程自动释放锁。

　　那么如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程便只能干巴巴地等待，试想一下，这多么影响程序执行效率。

　　因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断），通过Lock就可以办到。

　　再举个例子：当有多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作会发生冲突现象，但是读操作和读操作不会发生冲突现象。

　　但是采用synchronized关键字来实现同步的话，就会导致一个问题：

　　如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。

　　因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过Lock就可以办到。

　　另外，通过Lock可以知道线程有没有成功获取到锁。这个是synchronized无法办到的。

　　总结一下，也就是说Lock提供了比synchronized更多的功能。但是要注意以下几点：

　　1）Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；

　　2）Lock和synchronized有一点非常大的不同，采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。
  
  
 ##### Lock接口
Lock接口中每个方法的使用，lock()、tryLock()、tryLock(long time, TimeUnit unit)和lockInterruptibly()是用来获取锁的。unLock()方法是用来释放锁的。newCondition()这个方法暂且不在此讲述，会在后面的线程协作一文中讲述。

　lock()方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。

　由于在前面讲到如果采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一般来说，使用Lock必须在try{}catch{}块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。通常使用Lock来进行同步的话，是以下面这种形式去使用的：
```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){
     
}finally{
    lock.unlock();   //释放锁
}
```
　tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。

　tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

　　所以，一般情况下通过tryLock来获取锁时是这样使用的：
```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){
         
     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```
 　lockInterruptibly()方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。
　
 由于lockInterruptibly()的声明中抛出了异常，所以lock.lockInterruptibly()必须放在try块中或者在调用lockInterruptibly()的方法外声明抛出InterruptedException。
  
  因此lockInterruptibly()一般的使用形式如下：

```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```
　　注意，当一个线程获取了锁之后，是不会被interrupt()方法中断的。因为本身在前面的文章中讲过单独调用interrupt()方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程。

　　因此当通过lockInterruptibly()方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。

　　而用synchronized修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。
  
  
##### ReentrantLock

　　ReentrantLock，意思是“可重入锁”，关于可重入锁的概念在下一节讲述。ReentrantLock是唯一实现了Lock接口的类，并且ReentrantLock提供了更多的方法。下面通过一些实例看具体看一下如何使用ReentrantLock。
```java
public class Test {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();
    private Lock lock = new ReentrantLock();    //注意这个地方
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }  
     
    public void insert(Thread thread) {
        lock.lock();
        try {
            System.out.println(thread.getName()+"得到了锁");
            for(int i=0;i<5;i++) {
                arrayList.add(i);
            }
        } catch (Exception e) {
            // TODO: handle exception
        }finally {
            System.out.println(thread.getName()+"释放了锁");
            lock.unlock();
        }
    }
}
 ```

tryLock()的使用方法
```java
public class Test {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();
    private Lock lock = new ReentrantLock();    //注意这个地方
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }  
     
    public void insert(Thread thread) {
        if(lock.tryLock()) {
            try {
                System.out.println(thread.getName()+"得到了锁");
                for(int i=0;i<5;i++) {
                    arrayList.add(i);
                }
            } catch (Exception e) {
                // TODO: handle exception
            }finally {
                System.out.println(thread.getName()+"释放了锁");
                lock.unlock();
            }
        } else {
            System.out.println(thread.getName()+"获取锁失败");
        }
    }
}
```
lockInterruptibly()响应中断的使用方法：
```java
public class Test {
    private Lock lock = new ReentrantLock();   
    public static void main(String[] args)  {
        Test test = new Test();
        MyThread thread1 = new MyThread(test);
        MyThread thread2 = new MyThread(test);
        thread1.start();
        thread2.start();
         
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread2.interrupt();
    }  
     
    public void insert(Thread thread) throws InterruptedException{
        lock.lockInterruptibly();   //注意，如果需要正确中断等待锁的线程，必须将获取锁放在外面，然后将InterruptedException抛出
        try {  
            System.out.println(thread.getName()+"得到了锁");
            long startTime = System.currentTimeMillis();
            for(    ;     ;) {
                if(System.currentTimeMillis() - startTime >= Integer.MAX_VALUE)
                    break;
                //插入数据
            }
        }
        finally {
            System.out.println(Thread.currentThread().getName()+"执行finally");
            lock.unlock();
            System.out.println(thread.getName()+"释放了锁");
        }  
    }
}
 
class MyThread extends Thread {
    private Test test = null;
    public MyThread(Test test) {
        this.test = test;
    }
    @Override
    public void run() {
         
        try {
            test.insert(Thread.currentThread());
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName()+"被中断");
        }
    }
}
```

##### ReadWriteLock

　　ReadWriteLock也是一个接口，在它里面只定义了两个方法：
```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading.
     */
    Lock readLock();
 
    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing.
     */
    Lock writeLock();
}
```
 　　一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。下面的ReentrantReadWriteLock实现了ReadWriteLock接口。

##### ReentrantReadWriteLock

　　ReentrantReadWriteLock里面提供了很多丰富的方法，不过最主要的有两个方法：readLock()和writeLock()用来获取读锁和写锁。

　　下面通过几个例子来看一下ReentrantReadWriteLock具体用法。
```java
public class Test {
    private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
     
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
    }  
     
    public void get(Thread thread) {
        rwl.readLock().lock();
        try {
            long start = System.currentTimeMillis();
             
            while(System.currentTimeMillis() - start <= 1) {
                System.out.println(thread.getName()+"正在进行读操作");
            }
            System.out.println(thread.getName()+"读操作完毕");
        } finally {
            rwl.readLock().unlock();
        }
    }
}
```

　　说明thread1和thread2在同时进行读操作。

　　这样就大大提升了读操作的效率。

　　不过要注意的是，如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。

　　如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。

##### Lock和synchronized的选择

　　总结来说，Lock和synchronized有以下几点不同：

　　1）Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；

　　2）synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

　　3）Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；

　　4）通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

　　5）Lock可以提高多个线程进行读操作的效率。

　　在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。所以说，在具体使用时要根据适当情况选择。
  
#### volatile
##### volatile关键字的两层语义

　　一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

　　1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

　　2）禁止进行指令重排序。　volatile关键字禁止指令重排序有两层意思：

　　当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；

　    在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。


   所以：
   
   -  原子性：volatile 不能保证操作的原子性，可以考虑使用Lock、synchronized和原子类来保证
   -  有序性：volatile 能够一定程度上保证有序性
   -  可见性：volatile可以保证可见性
   -  
##### volatile关键字的使用场景
synchronized关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。通常来说，使用volatile必须具备以下2个条件：

　　1）对变量的写操作不依赖于当前值

　　2）该变量没有包含在具有其他变量的不变式中

　　实际上，这些条件表明，可以被写入 volatile 变量的这些有效值独立于任何程序的状态，包括变量的当前状态。

　　事实上，我的理解就是上面的2个条件需要保证操作是原子性操作，才能保证使用volatile关键字的程序在并发时能够正确执行。

　　下面列举几个Java中使用volatile的几个场景。

**1.状态标记量**

```java
volatile boolean flag = false;
 
while(!flag){
    doSomething();
}
 
public void setFlag() {
    flag = true;
}
 
volatile boolean inited = false;
//线程1:
context = loadContext();  
inited = true;            
 
//线程2:
while(!inited ){
sleep()
}
doSomethingwithconfig(context);
 
```
**2.double check**

```java
class Singleton{
    private volatile static Singleton instance = null;
     
    private Singleton() {
         
    }
     
    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```


#### ThreadLocal

ThreadLocal 不是用于解决共享变量的问题的，不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制，理解这点对正确使用ThreadLocal至关重要。

应用场景：

The class below generates unique identifiers local to each thread. A thread's id is assigned the first time it invokes ThreadId.get() and remains unchanged on subsequent calls.

```java
 import java.util.concurrent.atomic.AtomicInteger;

 public class ThreadId {
     // Atomic integer containing the next thread ID to be assigned
     private static final AtomicInteger nextId = new AtomicInteger(0);

     // Thread local variable containing each thread's ID
     private static final ThreadLocal<Integer> threadId =
         new ThreadLocal<Integer>() {
             @Override protected Integer initialValue() {
                 return nextId.getAndIncrement();
         }
     };

     // Returns the current thread's unique ID, assigning it if necessary
     public static int get() {
         return threadId.get();
     }
 }
 
 ```
Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).


#### CountDownLatch、CyclicBarrier和Semaphore


##### CountDownLatch用法


　　CountDownLatch可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

　　CountDownLatch类只提供了一个构造器：  public CountDownLatch(int count) {  };  //参数count为计数值
 　
下面这3个方法是CountDownLatch类中最重要的方法：
  
- public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行

- public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行

- public void countDown() { };  //将count值减1

 看一个例子就清楚CountDownLatch的用法了：

```java
public class Test {
     public static void main(String[] args) {   
         final CountDownLatch latch = new CountDownLatch(2);
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                     Thread.sleep(3000);
                     System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                     latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         try {
             System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
     }
}
```

> 注：当前线程，等待所有线程

##### CyclicBarrier用法

　　字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：


- public CyclicBarrier(int parties, Runnable barrierAction) 
 
- public CyclicBarrier(int parties) 

    参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

　然后CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：


- public int await() 
- public int await(long timeout, TimeUnit unit)

第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。

举个例子就明白了：

假若有若干个线程都要进行写数据操作，并且只有所有线程都完成写数据操作之后，这些线程才能继续做后面的事情，此时就可以利用CyclicBarrier了：
```java
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N,new Runnable() {
            @Override
            public void run() {
                System.out.println("当前线程"+Thread.currentThread().getName());   
            }
        });
         
        for(int i=0;i<N;i++)
            new Writer(barrier).start();
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}

```
结果：
```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2正在写入数据...
线程Thread-3正在写入数据...
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
当前线程Thread-3
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
```

　　从上面输出结果可以看出，每个写入线程执行完写数据操作之后，就在等待其他线程写入操作完毕。

　　当所有线程线程写入操作完毕之后，所有线程就继续进行后续的操作了。

　　如果说想在所有线程写入操作完之后，进行额外的其他操作可以为CyclicBarrier提供Runnable参数：从结果可以看出，当四个线程都到达barrier状态后，会从四个线程中选择一个线程去执行Runnable。

> 注：所有线程，等待所有线程

三.Semaphore用法

　　Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

Semaphore类位于java.util.concurrent包下，它提供了2个构造器：

- public Semaphore(int permits)      //参数permits表示许可数目，即同时可以允许多少线程进行访问

- public Semaphore(int permits, boolean fair)   //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可

 下面说一下Semaphore类中比较重要的几个方法，首先是acquire()、release()方法：

- public void acquire() throws InterruptedException {  }     //获取一个许可
- public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
- public void release() { }          //释放一个许可
- public void release(int permits) { }    //释放permits个许可
　　acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

　　release()用来释放许可。注意，在释放许可之前，必须先获获得许可。      这4个方法都会被阻塞。
  
 如果想立即得到执行结果，使用下面几个方法：

- public boolean tryAcquire() { };    //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
- public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
- public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
- public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false

 　　另外还可以通过availablePermits()方法得到可用的许可数目。

　　下面通过一个例子来看一下Semaphore的具体使用：

　　假若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现：
```java
public class Test {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }
     
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
         
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();           
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
  　　执行结果：
	```
工人0占用一个机器在生产...
工人1占用一个机器在生产...
工人2占用一个机器在生产...
工人4占用一个机器在生产...
工人5占用一个机器在生产...
工人0释放出机器
工人2释放出机器
工人3占用一个机器在生产...
工人7占用一个机器在生产...
工人4释放出机器
工人5释放出机器
工人1释放出机器
工人6占用一个机器在生产...
工人3释放出机器
工人7释放出机器
工人6释放出机器
　```
 
 >注：类似线程池
 
#### 总结

- CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：

  CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
  而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；
  另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。

- Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。


#### wait、notify、notifyAll和Condition
##### wait、notify和notifyAll

-  wait()、notify()和notifyAll()方法是本地方法，并且为final方法，无法被重写。

-  调用某个对象的wait()方法能让当前线程阻塞，并且当前线程必须拥有此对象的monitor（即锁）

-  调用某个对象的notify()方法能够唤醒一个正在等待这个对象的monitor的线程，如果有多个线程都在等待这个对象的monitor，则只能唤醒其中一个线程；

-  调用notifyAll()方法能够唤醒所有正在等待这个对象的monitor的线程；

- 如果调用某个对象的wait()方法，当前线程必须拥有这个对象的monitor（即锁），因此调用wait()方法必须在同步块或者同步方法中进行（synchronized块或者synchronized方法）。

- 同样地，调用某个对象的notify()方法，当前线程也必须拥有这个对象的monitor，因此调用notify()方法必须在同步块或者同步方法中进行（synchronized块或者synchronized方法）。
  
- 调用某个对象的wait()方法，相当于让当前线程交出此对象的monitor，然后进入等待状态，等待后续再次获得此对象的锁（Thread类中的sleep方法使当前线程暂停执行一段时间，从而让其他线程有机会继续执行，但它并不释放对象锁）；

- notify()方法能够唤醒一个正在等待该对象的monitor的线程，当有多个线程都在等待该对象的monitor的话，则只能唤醒其中一个线程，具体唤醒哪个线程则不得而知。

- nofityAll()方法能够唤醒所有正在等待该对象的monitor的线程，这一点与notify()方法是不同的。

要注意：
1、notify()和notifyAll()方法只是唤醒等待该对象的monitor的线程，并不决定哪个线程能够获取到monitor。

2、一个线程被唤醒不代表立即获取了对象的monitor，只有等调用完notify()或者notifyAll()并退出synchronized块，释放对象锁后，其余线程才可获得锁执行。
```java
public class Test {
    public static Object object = new Object();
    public static void main(String[] args) {
        Thread1 thread1 = new Thread1();
        Thread2 thread2 = new Thread2();
         
        thread1.start();
         
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
         
        thread2.start();
    }
     
    static class Thread1 extends Thread{
        @Override
        public void run() {
            synchronized (object) {
                try {
                    object.wait();
                } catch (InterruptedException e) {
                }
                System.out.println("线程"+Thread.currentThread().getName()+"获取到了锁");
            }
        }
    }
     
    static class Thread2 extends Thread{
        @Override
        public void run() {
            synchronized (object) {
                object.notify();
                System.out.println("线程"+Thread.currentThread().getName()+"调用了object.notify()");
            }
            System.out.println("线程"+Thread.currentThread().getName()+"释放了锁");
        }
    }
}
```
结果是：
```
线程Thread-1调用了object.notify()
线程Thread-1释放了锁
线程Thread-0获取到了锁
```

##### Condition
它用来替代传统的Object的wait()、notify()实现线程间的协作，相比使用Object的wait()、notify()，使用Condition1的await()、signal()这种方式实现线程间协作更加安全和高效。因此通常来说比较推荐使用Condition，阻塞队列实际上是使用了Condition来模拟线程间协作。

Condition是个接口，基本的方法就是await()和signal()方法；
Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition() 
 调用Condition的await()和signal()方法，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用
　　Conditon中的await()对应Object的wait()；

　　Condition中的signal()对应Object的notify()；

　　Condition中的signalAll()对应Object的notifyAll()。
  ```java
  public class Test {
    private int queueSize = 10;
    private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);
    private Lock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();
     
    public static void main(String[] args)  {
        Test test = new Test();
        Producer producer = test.new Producer();
        Consumer consumer = test.new Consumer();
          
        producer.start();
        consumer.start();
    }
      
    class Consumer extends Thread{
          
        @Override
        public void run() {
            consume();
        }
          
        private void consume() {
            while(true){
                lock.lock();
                try {
                    while(queue.size() == 0){
                        try {
                            System.out.println("队列空，等待数据");
                            notEmpty.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.poll();                //每次移走队首元素
                    notFull.signal();
                    System.out.println("从队列取走一个元素，队列剩余"+queue.size()+"个元素");
                } finally{
                    lock.unlock();
                }
            }
        }
    }
      
    class Producer extends Thread{
          
        @Override
        public void run() {
            produce();
        }
          
        private void produce() {
            while(true){
                lock.lock();
                try {
                    while(queue.size() == queueSize){
                        try {
                            System.out.println("队列满，等待有空余空间");
                            notFull.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.offer(1);        //每次插入一个元素
                    notEmpty.signal();
                    System.out.println("向队列取中插入一个元素，队列剩余空间："+(queueSize-queue.size()));
                } finally{
                    lock.unlock();
                }
            }
        }
    }
}
```

### 同步容器
#### Java中的同步容器类

　　在Java中，同步容器主要包括2类：

　　1）Vector、Stack、HashTable

　　2）Collections类中提供的静态工厂方法创建的类

　　Vector实现了List接口，Vector实际上就是一个数组，和ArrayList类似，但是Vector中的方法都是synchronized方法，即进行了同步措施。

　　Stack也是一个同步容器，它的方法也用synchronized进行了同步，它实际上是继承于Vector类。

　　HashTable实现了Map接口，它和HashMap很相似，但是HashTable进行了同步处理，而HashMap没有。

　　Collections类是一个工具提供类，注意，它和Collection不同，Collection是一个顶层的接口。在Collections类中提供了大量的方法，比如对集合或者容器进行排序、查找等操作。最重要的是，在它里面提供了几个静态工厂方法来创建同步容器类
  
  
 #### 同步容器的缺陷

1、性能问题：从同步容器的具体实现源码可知，同步容器中的方法采用了synchronized进行了同步，那么很显然，这必然会影响到执行性能。



2.线程安全问题：同步容器并非真的是安全

　　也有有人认为Vector中的方法都进行了同步处理，那么一定就是线程安全的，事实上这可不一定。看下面这段代码：
```java
public class Test {
    static Vector<Integer> vector = new Vector<Integer>();
    public static void main(String[] args) throws InterruptedException {
        while(true) {
            for(int i=0;i<10;i++)
                vector.add(i);
            Thread thread1 = new Thread(){
                public void run() {
                    for(int i=0;i<vector.size();i++)
                        vector.remove(i);
                };
            };
            Thread thread2 = new Thread(){
                public void run() {
                    for(int i=0;i<vector.size();i++)
                        vector.get(i);
                };
            };
            thread1.start();
            thread2.start();
            while(Thread.activeCount()>10)   {
                 
            }
        }
    }
}
```
运行的结果：

![enter description here][2]

　　正如大家所看到的，这段代码报错了：数组下标越界。

　　也许有朋友会问：Vector是线程安全的，为什么还会报这个错？很简单，对于Vector，虽然能保证每一个时刻只能有一个线程访问它，但是不排除这种可能：

　　当某个线程在某个时刻执行这句时：

for(int i=0;i<vector.size();i++)
    vector.get(i);
	
假若此时vector的size方法返回的是10，i的值为9

然后另外一个线程执行了这句：
  
for(int i=0;i<vector.size();i++)
    vector.remove(i);
　
 将下标为9的元素删除了。那么通过get方法访问下标为9的元素肯定就会出问题了。

因此为了保证线程安全，必须在方法调用端做额外的同步措施。

### 阻塞队列
#### 几种主要的阻塞队列

　　自从Java 1.5之后，在java.util.concurrent包下提供了若干个阻塞队列，主要有以下几个：

　　ArrayBlockingQueue：基于数组实现的一个阻塞队列，在创建ArrayBlockingQueue对象时必须制定容量大小。并且可以指定公平性与非公平性，默认情况下为非公平的，即不保证等待时间最长的队列最优先能够访问队列。

　　LinkedBlockingQueue：基于链表实现的一个阻塞队列，在创建LinkedBlockingQueue对象时如果不指定容量大小，则默认大小为Integer.MAX_VALUE。

　　PriorityBlockingQueue：以上2种队列都是先进先出队列，而PriorityBlockingQueue却不是，它会按照元素的优先级对元素进行排序，按照优先级顺序出队，每次出队的元素都是优先级最高的元素。注意，此阻塞队列为无界阻塞队列，即容量没有上限（通过源码就可以知道，它没有容器满的信号标志），前面2种都是有界队列。

　　DelayQueue：基于PriorityQueue，一种延时阻塞队列，DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue也是一个无界队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。
  
#### 阻塞队列的实现原理

　　以ArrayBlockingQueue为例，其他阻塞队列实现原理可能和ArrayBlockingQueue有一些差别，但是大体思路应该类似，有兴趣的朋友可自行查看其他阻塞队列的实现源码。

　　首先看一下ArrayBlockingQueue类中的几个成员变量：
 ```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
implements BlockingQueue<E>, java.io.Serializable {
 
private static final long serialVersionUID = -817911632652898426L;
 
/** The queued items  */
private final E[] items;
/** items index for next take, poll or remove */
private int takeIndex;
/** items index for next put, offer, or add. */
private int putIndex;
/** Number of items in the queue */
private int count;
 
/*
* Concurrency control uses the classic two-condition algorithm
* found in any textbook.
*/
 
/** Main lock guarding all access */
private final ReentrantLock lock;
/** Condition for waiting takes */
private final Condition notEmpty;
/** Condition for waiting puts */
private final Condition notFull;
}
```
 　　可以看出，ArrayBlockingQueue中用来存储元素的实际上是一个数组，takeIndex和putIndex分别表示队首元素和队尾元素的下标，count表示队列中元素的个数。

　　lock是一个可重入锁，notEmpty和notFull是等待条件。

　　下面看一下ArrayBlockingQueue的构造器，构造器有三个重载版本：
 ```java
public ArrayBlockingQueue(int capacity) {
}
public ArrayBlockingQueue(int capacity, boolean fair) {
 
}
public ArrayBlockingQueue(int capacity, boolean fair,
                          Collection<? extends E> c) {
}
```
 　　第一个构造器只有一个参数用来指定容量，第二个构造器可以指定容量和公平性，第三个构造器可以指定容量、公平性以及用另外一个集合进行初始化。

　　然后看它的两个关键方法的实现：put()和take()：
 ```java
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    final E[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        try {
            while (count == items.length)
                notFull.await();
        } catch (InterruptedException ie) {
            notFull.signal(); // propagate to non-interrupted thread
            throw ie;
        }
        insert(e);
    } finally {
        lock.unlock();
    }
}
```
 　　从put方法的实现可以看出，它先获取了锁，并且获取的是可中断锁，然后判断当前元素个数是否等于数组的长度，如果相等，则调用notFull.await()进行等待，如果捕获到中断异常，则唤醒线程并抛出异常。

　　当被其他线程唤醒时，通过insert(e)方法插入元素，最后解锁。

　　我们看一下insert方法的实现：
 ```java
private void insert(E x) {
    items[putIndex] = x;
    putIndex = inc(putIndex);
    ++count;
    notEmpty.signal();
}
```
 　　它是一个private方法，插入成功后，通过notEmpty唤醒正在等待取元素的线程。

　　下面是take()方法的实现：
 ```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        try {
            while (count == 0)
                notEmpty.await();
        } catch (InterruptedException ie) {
            notEmpty.signal(); // propagate to non-interrupted thread
            throw ie;
        }
        E x = extract();
        return x;
    } finally {
        lock.unlock();
    }
}
```
 　　跟put方法实现很类似，只不过put方法等待的是notFull信号，而take方法等待的是notEmpty信号。在take方法中，如果可以取元素，则通过extract方法取得元素，下面是extract方法的实现：
 ```java
private E extract() {
    final E[] items = this.items;
    E x = items[takeIndex];
    items[takeIndex] = null;
    takeIndex = inc(takeIndex);
    --count;
    notFull.signal();
    return x;
}
```
 　　跟insert方法也很类似。

　　其实从这里大家应该明白了阻塞队列的实现原理，事实它和我们用Object.wait()、Object.notify()和非阻塞队列实现生产者-消费者的思路类似，只不过它把这些工作一起集成到了阻塞队列中实现。

### 并发容器

　　JDK5中添加了新的concurrent包，相对同步容器而言，并发容器通过一些机制改进了并发性能。因为同步容器将所有对容器状态的访问都串行化了，这样保证了线程的安全性，所以这种方法的代价就是严重降低了并发性，当多个线程竞争容器时，吞吐量严重降低。因此Java5.0开始针对多线程并发访问设计，提供了并发性能较好的并发容器，引入了java.util.concurrent包。与Vector和Hashtable、

Collections.synchronizedXxx()同步容器等相比，util.concurrent中引入的并发容器主要解决了两个问题： 
1）根据具体场景进行设计，尽量避免synchronized，提供并发性。 
2）定义了一些并发安全的复合操作，并且保证并发环境下的迭代操作不会出错。

　　util.concurrent中容器在迭代时，可以不封装在synchronized中，可以保证不抛异常，但是未必每次看到的都是"最新的、当前的"数据。

　　下面是对并发容器的简单介绍：

　　ConcurrentHashMap代替同步的Map（Collections.synchronized（new HashMap()）），众所周知，HashMap是根据散列值分段存储的，同步Map在同步的时候锁住了所有的段，而ConcurrentHashMap加锁的时候根据散列值锁住了散列值锁对应的那段，因此提高了并发性能。ConcurrentHashMap也增加了对常用复合操作的支持，比如"若没有则添加"：putIfAbsent()，替换：replace()。这2个操作都是原子操作。

　　CopyOnWriteArrayList和CopyOnWriteArraySet分别代替List和Set，主要是在遍历操作为主的情况下来代替同步的List和同步的Set，这也就是上面所述的思路：迭代过程要保证不出错，除了加锁，另外一种方法就是"克隆"容器对象。

　　ConcurrentLinkedQuerue是一个先进先出的队列。它是非阻塞队列。

 　  ConcurrentSkipListMap可以在高效并发中替代SoredMap（例如用Collections.synchronzedMap包装的TreeMap）。ConcurrentHashMap可以做到读取数据不加锁，并且其内部的结构可以让其在进行写操作的时候能够将锁的粒度保持地尽量地小，不用对整个ConcurrentHashMap加锁。

　　ConcurrentSkipListSet可以在高效并发中替代SoredSet（例如用Collections.synchronzedSet包装的TreeMap）。


### JAVA线程调度与切换

#### RxJava
subscribeOn()和observeOn()都是用来切换线程用的

subscribeOn()主要改变的是订阅的线程，即call()执行的线程;
observeOn()主要改变的是发送的线程，即onNext()执行的线程。

### 参考资

[1.Java并发编程](http://www.cnblogs.com/dolphin0520/category/602384.html)



  [1]: ./images/1482723883361.jpg "1482723883361.jpg"
  [2]: ./images/1482999622348.jpg "1482999622348.jpg"