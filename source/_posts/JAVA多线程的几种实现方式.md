---
title: JAVA多线程的几种实现方式
date: 2016-11-22 18:28:48
tags: [Android,多线程]
categories: [JAVA]
---

### JAVA多线程的几种实现方式
@(技术学习)[多线程, 线程池]

[TOC]

#### 继承Thread类，并重写run函数
（1）定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体。

（2）创建Thread子类的实例，即创建了线程对象。

（3）调用线程对象的start()方法来启动该线程。
#### 实现Runnable接口，并写run函数
（1）定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。

（2）创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。

（3）调用线程对象的start()方法来启动该线程。
#### 实现Callable接口，并重写call函数
（1）创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。

（2）创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。

（3）使用FutureTask对象作为Thread对象的target创建并启动新线程。

（4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值


#### Thread、Runnable、Callable创建线程方式的对比

采用实现Runnable、Callable接口的方式创见多线程时，优势是：
- 线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。
- 在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。

劣势是：
- 编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

使用继承Thread类的方式创建多线程时优势是：
- 编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。

劣势是：
- 线程类已经继承了Thread类，所以不能再继承其他父类。

#### 线程池
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

###### newCachedThreadPool
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。示例代码如下：

```
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
```
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
```
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
```
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
```
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
结果依次输出，相当于顺序执行各个任务。