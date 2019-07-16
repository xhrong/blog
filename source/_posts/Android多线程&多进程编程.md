---
title: Android多线程&多进程编程
tags: [线程,进程]
grammar_cjkRuby: true
categories: [Android]
date: 2019-07-14
---

### 跨线程通信

#### Pipes

管道流（pipeStream）是种特殊的流，用于在不同线程间**单向**传送数据，一个线程一端发送数据到管道，另外一个线程从输入管道读取

管道流是Java IO提供的进程间通信方案，不是Android平台专有的。

管道流有两种：

 - 字符流：PipedReader、PipedWriter
 - 字节流：PipedInputStrean、PipedOutputStram 

```java
package com.iflytek.timesdemo;


import java.io.IOException;
import java.io.PipedReader;
import java.io.PipedWriter;

public class PipedWriteTest {

    public static class ThreadPipedReaderA extends Thread {

        PipedReader pipedReader;

        public ThreadPipedReaderA(PipedReader pipedReader) {
            this.pipedReader = pipedReader;
        }

        @Override
        public void run() {
            try {
                char[] chararray = new char[20];
                int readLenth = pipedReader.read(chararray);
                while (readLenth != -1) {
                    String str = new String(chararray, 0, readLenth);
                    System.out.println("读到的数据为：" + str);
                    readLenth = pipedReader.read(chararray);
                }
                pipedReader.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }

    public static class ThradPipedWriteB extends Thread {

        PipedWriter pipedWriter;
        
        public ThradPipedWriteB(PipedWriter pipedWriter) {
            this.pipedWriter = pipedWriter;
        }

        @Override
        public void run() {
            try {
                for (int i = 0; i <= 10; i++) {
                    pipedWriter.write("发送数据" + i);
                }
                pipedWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws IOException {
        PipedReader pipedReader = new PipedReader();
        PipedWriter pipedWriter = new PipedWriter();

        pipedReader.connect(pipedWriter);// 做个链接才能通讯
        ThreadPipedReaderA a = new ThreadPipedReaderA(pipedReader);
        ThradPipedWriteB b = new ThradPipedWriteB(pipedWriter);
        a.start();
        b.start();
    }
}
```

#### Shared Memory

Shared Memory即共享变量，要求是final变量，用的较多，且较简单。从略。

![enter description here](./images/1563192614362.png)

锁机制也是共享变量的一种。

![enter description here](./images/1563192647112.png)

#### BlockingQueue
BlockingQueue队列和平常队列一样都可以用来作为存储数据的容器，但有时候在线程当中涉及到数据存储的时候就会出现问题，BlockingQueue是空的话，如果一个线程要从BlockingQueue里取数据的时候，该线程将会被阻断，并进入等待状态，直到BlockingQueue里面有数据存入了后，就会唤醒线程进行数据的去除。若BlockingQueue是满的，如果一个线程要将数据存入BlockQueue，该线程将会被阻断，并进入等待状态，直到BlcokQueue里面的数据被取出有空间后，线程被唤醒后在将数据存入

BlockingQueue 方法以四种形式出现，对于不能立即满足但可能在将来某一时刻可以满足的操作，这四种形式的处理方式不同：第一种是抛出一个异常，第二种是返回一个特殊值（null 或 false，具体取决于操作），第三种是在操作可以成功前，无限期地阻塞当前线程，第四种是在放弃前只在给定的最大时间限制内阻塞。下表中总结了这些方法：
 
 ![enter description here](./images/1563192091915.png)

- add(anObject):把anObject加到BlockingQueue里,即如果BlockingQueue可以容纳,则返回true,否则报异常 
- offer(anObject):表示如果可能的话,将anObject加到BlockingQueue里,即如果BlockingQueue可以容纳,则返回true,否则返回false. 
- put(anObject):把anObject加到BlockingQueue里,如果BlockQueue没有空间,则调用此方法的线程被阻断直到BlockingQueue里面有空间再继续. 
- poll(time):取走BlockingQueue里排在首位的对象,若不能立即取出,则可以等time参数规定的时间,取不到时返回null 
- take():取走BlockingQueue里排在首位的对象,若BlockingQueue为空,阻断进入等待状态直到Blocking有新的对象被加入为止 

BlockingQueue有四个具体的实现类,根据不同需求,选择不同的实现类 ：
 - ArrayBlockingQueue:规定大小的BlockingQueue,其构造函数必须带一个int参数来指明其大小.其所含的对象是以FIFO(先入先出)顺序排序的. 
- LinkedBlockingQueue:大小不定的BlockingQueue,若其构造函数带一个规定大小的参数,生成的 BlockingQueue有大小限制,若不带大小参数,所生成的BlockingQueue的大小由Integer.MAX_VALUE来决定.其所含 的对象是以FIFO(先入先出)顺序排序的 
- PriorityBlockingQueue:类似于LinkedBlockQueue,但其所含对象的排序不是FIFO,而是依据对象的自然排序顺序或者是构造函数的Comparator决定的顺序. 
- SynchronousQueue:特殊的BlockingQueue,对其的操作必须是放和取交替完成的. 

LinkedBlockingQueue和ArrayBlockingQueue比较起来,它们背后所用的数据结构不一样,导致 LinkedBlockingQueue的数据吞吐量要大于ArrayBlockingQueue,但在线程数量很大时其性能的可预见性低于 ArrayBlockingQueue.     

```java
 class Producer implements Runnable {
   private final BlockingQueue queue;
   Producer(BlockingQueue q) { queue = q; }
   public void run() {
     try {
       while (true) { queue.put(produce()); }
     } catch (InterruptedException ex) { ... handle ...}
   }
   Object produce() { ... }
 }

 class Consumer implements Runnable {
   private final BlockingQueue queue;
   Consumer(BlockingQueue q) { queue = q; }
   public void run() {
     try {
       while (true) { consume(queue.take()); }
     } catch (InterruptedException ex) { ... handle ...}
   }
   void consume(Object x) { ... }
 }

 class Setup {
   void main() {
     BlockingQueue q = new SomeQueueImplementation();
     Producer p = new Producer(q);
     Consumer c1 = new Consumer(q);
     Consumer c2 = new Consumer(q);
     new Thread(p).start();
     new Thread(c1).start();
     new Thread(c2).start();
   }
 }
```

#### Handler
Handler机制比较常见，从略

![enter description here](./images/1563192334868.png)

补充：IdleHandler

在android的MessageQueue中有一个static的接口IdleHandler，这个接口用于在MessageQueue中没有可处理的Message的时候回调，这样就可以在UI线程中处理完所有的view事务之后，回调一些额外的操作而不会block UI线程。
```java
mHandler.getLooper().getQueue().addIdleHandler(new MessageQueue.IdleHandler() {
            @Override
            public boolean queueIdle() {
                Log.e(TAG, "----I am idle");
                return false;
            }
        });
```
#### UI Thread
在线程中更新UI线程，还有以下几种常用方案：

- new Handler(Looper.getMainLooper()).post(task);

- runOnUiThread(new Runnable() { ... });

- myView.post(new Runnable());

### 跨进程通信

#### Binder

![enter description here](./images/1563193110312.png)

![enter description here](./images/1563193196421.png)

Binder通信的四个角色：
- Client进程：使用服务的进程。
- Server进程：提供服务的进程。
- ServiceManager进程：ServiceManager的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。
- Binder驱动：驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。

#### AIDL

![enter description here](./images/1563192999715.png)

AIDL实现过程是：定义接口，实现Sever，实现Client。从略。

补充：
- AIDL客户端调用**默认是阻塞**的，即如果不做线程处理，可能会发生ANR。
- oneway 修饰可将AIDL调用变成非阻塞的回调方式，但是有一定的限制：接口方法只能返回void，不能有 in 或者 out 修饰参数

```java

import com.iflytek.aidl.IAsynchronousCallback;
interface ISynchronous {
     oneway void getThreadNameSlow(long sleep,IAsynchronousCallback callback);
}


interface IAsynchronousCallback {
    void handleResult(String name);
}


public class AIDLChannel extends ISynchronous.Stub {

    @Override
    public void getThreadNameSlow(long sleep,IAsynchronousCallback callback) throws RemoteException {
        SystemClock.sleep(sleep);

        if(callback !=null){
            callback.handleResult(Thread.currentThread().getName());
        }
    }
}
```


#### Messenger

Messenger 对 AIDL 进行了封装，也就是对 Binder 的封装，我们可以使用它的实现来完成基于消息的跨进程通信，就和使用 Handler 一样简单。

使用步骤：
- 客户端创建一个 Messenger，传递一个 Handler 处理消息
服务端也一样
- 如果需要双向通信，给 Message 设置一个用于回信的 Messenger 即可：message.replyTo = mClientMessenger;

-客户端在调用send()方法之后，就会走 Binder 跨进程通信机制 ，最后到服务端的 Handler 中得到处理。

```java

public class MessengerService extends Service {

    private static final int RECEIVE_MESSAGE_CODE = 0x0001;

    private static final int SEND_MESSAGE_CODE = 0x0002;

    //clientMessenger表示的是客户端的Messenger，可以通过来自于客户端的Message的replyTo属性获得，
    //其内部指向了客户端的ClientHandler实例，可以用clientMessenger向客户端发送消息
    private Messenger clientMessenger = null;

    //serviceMessenger是Service自身的Messenger，其内部指向了ServiceHandler的实例
    //客户端可以通过IBinder构建Service端的Messenger，从而向Service发送消息，
    //并由ServiceHandler接收并处理来自于客户端的消息
    private Messenger serviceMessenger = new Messenger(new ServiceHandler());

    //MyService用ServiceHandler接收并处理来自于客户端的消息
    private class ServiceHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            Log.i("DemoLog", "ServiceHandler -> handleMessage");
            if(msg.what == RECEIVE_MESSAGE_CODE){
                Bundle data = msg.getData();
                if(data != null){
                    String str = data.getString("msg");
                    Log.i("DemoLog", "MyService收到客户端如下信息: " + str);
                }
                //通过Message的replyTo获取到客户端自身的Messenger，
                //Service可以通过它向客户端发送消息
                clientMessenger = msg.replyTo;
                if(clientMessenger != null){
                    Log.i("DemoLog", "MyService向客户端回信");
                    Message msgToClient = Message.obtain();
                    msgToClient.what = SEND_MESSAGE_CODE;
                    //可以通过Bundle发送跨进程的信息
                    Bundle bundle = new Bundle();
                    bundle.putString("msg", "你好，客户端，我是MyService");
                    msgToClient.setData(bundle);
                    try{
                        clientMessenger.send(msgToClient);
                    }catch (RemoteException e){
                        e.printStackTrace();
                        Log.e("DemoLog", "MyService向客户端发送信息失败: " + e.getMessage());
                    }
                }
            }
        }
    }

    @Override
    public void onCreate() {
        Log.i("DemoLog", "MyService -> onCreate");
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent) {
        Log.i("DemoLog", "MyServivce -> onBind");
        //获取Service自身Messenger所对应的IBinder，并将其发送共享给所有客户端
        return serviceMessenger.getBinder();
    }

    @Override
    public void onDestroy() {
        Log.i("DemoLog", "MyService -> onDestroy");
        clientMessenger = null;
        super.onDestroy();
    }
}


public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    private static final int SEND_MESSAGE_CODE = 0x0001;

    private static final int RECEIVE_MESSAGE_CODE = 0x0002;

    private boolean isBound = false;

    //用于启动MyService的Intent对应的action
    private final String SERVICE_ACTION = "com.ispring2.action.MYSERVICE";

    //serviceMessenger表示的是Service端的Messenger，其内部指向了MyService的ServiceHandler实例
    //可以用serviceMessenger向MyService发送消息
    private Messenger serviceMessenger = null;

    //clientMessenger是客户端自身的Messenger，内部指向了ClientHandler的实例
    //MyService可以通过Message的replyTo得到clientMessenger，从而MyService可以向客户端发送消息，
    //并由ClientHandler接收并处理来自于Service的消息
    private Messenger clientMessenger = new Messenger(new ClientHandler());

    //客户端用ClientHandler接收并处理来自于Service的消息
    private class ClientHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            Log.i("DemoLog", "ClientHandler -> handleMessage");
            if(msg.what == RECEIVE_MESSAGE_CODE){
                Bundle data = msg.getData();
                if(data != null){
                    String str = data.getString("msg");
                    Log.i("DemoLog", "客户端收到Service的消息: " + str);
                }
            }
        }
    }

    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder binder) {
            //客户端与Service建立连接
            Log.i("DemoLog", "客户端 onServiceConnected");

            //我们可以通过从Service的onBind方法中返回的IBinder初始化一个指向Service端的Messenger
            serviceMessenger = new Messenger(binder);
            isBound = true;

            Message msg = Message.obtain();
            msg.what = SEND_MESSAGE_CODE;

            //此处跨进程Message通信不能将msg.obj设置为non-Parcelable的对象，应该使用Bundle
            //msg.obj = "你好，MyService，我是客户端";
            Bundle data = new Bundle();
            data.putString("msg", "你好，MyService，我是客户端");
            msg.setData(data);

            //需要将Message的replyTo设置为客户端的clientMessenger，
            //以便Service可以通过它向客户端发送消息
            msg.replyTo = clientMessenger;
            try {
                Log.i("DemoLog", "客户端向service发送信息");
                serviceMessenger.send(msg);
            } catch (RemoteException e) {
                e.printStackTrace();
                Log.i("DemoLog", "客户端向service发送消息失败: " + e.getMessage());
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            //客户端与Service失去连接
            serviceMessenger = null;
            isBound = false;
            Log.i("DemoLog", "客户端 onServiceDisconnected");
        }
    };


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        bindAidl();
    }


    private void  bindAidl(){

        Intent intent = new Intent();
        intent.setComponent(new ComponentName("com.iflytek.datashareservice", "com.iflytek.datashareservice.service.MessengerService"));
        getApplicationContext().bindService(intent, conn, Context.BIND_AUTO_CREATE);
    }
}

```

#### Other
Android平台下，常用跨进程通信还可以采用以下方案：
 - Service 
 - startActivityForResult() 
 - ContentProvider
 - Shared File

### Android异步编程方案

![enter description here](./images/1563263832984.png)

#### Thread
常用从略。特别说明一下Runnable和Callable的区别：

Runnable接口
```java
public interface Runnable {
    void run();
}
```

Callable接口
```java
public interface Callable<V> {
    V call() throws Exception;
}
```

- Runnable执行方法是run(),Callable是call()
- 实现Runnable接口的任务线程无返回值；实现Callable接口的任务线程能返回执行结果
- call方法可以抛出异常，run方法若有异常只能在内部消化
- Callable接口支持返回执行结果，需要调用FutureTask.get()方法实现，此方法会阻塞主线程直到获取结果；当不调用此方法时，主线程不会阻塞！ 
- 如果线程出现异常，Future.get()会抛出throws InterruptedException或者ExecutionException；如果线程已经取消，会抛出CancellationException


#### HandlerThread

HandlerThread的本质：继承Thread类 & 封装Handler类

HandlerThread的使用步骤：
```java
// 步骤1：创建HandlerThread实例对象
// 传入参数 = 线程名字，作用 = 标记该线程
   HandlerThread mHandlerThread = new HandlerThread("handlerThread");

// 步骤2：启动线程
   mHandlerThread.start();

// 步骤3：创建工作线程Handler & 复写handleMessage（）
// 作用：关联HandlerThread的Looper对象、实现消息处理操作 & 与其他线程进行通信
// 注：消息处理操作（HandlerMessage（））的执行线程 = mHandlerThread所创建的工作线程中执行
  Handler workHandler = new Handler( handlerThread.getLooper() ) {
            @Override
            public boolean handleMessage(Message msg) {
                ...//消息处理
                return true;
            }
        });

// 步骤4：使用工作线程Handler向工作线程的消息队列发送消息
// 在工作线程中，当消息循环时取出对应消息 & 在工作线程执行相关操作
  // a. 定义要发送的消息
  Message msg = Message.obtain();
  msg.what = 2; //消息的标识
  msg.obj = "B"; // 消息的存放
  // b. 通过Handler发送消息到其绑定的消息队列
  workHandler.sendMessage(msg);

// 步骤5：结束线程，即停止线程的消息循环
  mHandlerThread.quit();
  
```

#### Executor Framework

详见：http://xhrong.github.io/2016/11/22/JAVA%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%87%A0%E7%A7%8D%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F/

#### AsyncTask

详见：http://xhrong.github.io/2017/01/08/AsycTask%E6%BA%90%E7%A0%81%E6%8E%A2%E7%A9%B6/

#### IntentService
- IntentService 是继承自 Service 并处理异步请求的一个类，在 IntentService 内有一个工作线程来处理耗时操作。
- 当任务执行完后，IntentService 会自动停止，不需要我们去手动结束。
- 如果启动 IntentService 多次，那么每一个耗时操作会以工作队列的方式在 IntentService 的 onHandleIntent 回调方法中执行，依次去执行，使用串行的方式，执行完自动结束。

- IntentService 源码中的 onBind() 默认返回 null；不适合 bindService() 启动服务，如果你执意要 bindService() 来启动 IntentService，可能因为你想通过 Binder 或 Messenger 使得 IntentService 和 Activity 可以通信，这样那么 onHandleIntent() 不会被回调，相当于在你使用 Service 而不是 IntentService。

- IntentService 中使用的 Handler、Looper、MessageQueue 机制把消息发送到线程中去执行的，所以多次启动 IntentService 不会重新创建新的服务和新的线程，只是把消息加入消息队列中等待执行，而如果服务停止，会清除消息队列中的消息，后续的事件得不到执行。

#### AsyncQueryHandler

- AsyncQueryHandler继续于android.os.Handler，另开了一个线程来进行数据的操作
- AsyncQueryHandler异步对数据库进行增加、删除、更新、查询操作。避免在主线程中直接调用数据库相关操作导致的ANR异常。

```java
        final String[] from=new String[] { "_id","id" };
        //数据库异步查询
        new AsyncQueryHandler(getContentResolver()) {
            protected void onQueryComplete(int token, Object cookie,
                                           Cursor cursor) {
                //TestAdapter是SimpleCursorAdapter的子类
                TestAdapter adapter = new TestAdapter(MainActivity.this, cursor,
                        from);
                listView.setAdapter(adapter);
                listView.setSelection(adapter.getCount()-1);
            };

        }.startQuery(0, null, TestProvider.CONTENT_URI, from,null, null, null);
```
#### Loader
![enter description here](./images/1563263141674.png

在应用中使用加载器时，可能会涉及到多个类和接口。 下表汇总了这些类和接口：

![enter description here](./images/1563263793862.png)

上表中的类和接口是您在应用中用于实现加载器的基本组件。 并非您创建的每个加载器都要用到上述所有类和接口。但是，为了初始化加载器以及实现一个 Loader 类（如 CursorLoader），您始终需要要引用 LoaderManager。下文将为您展示如何在应用中使用这些类和

```java
public static class CursorLoaderListFragment extends ListFragment
        implements OnQueryTextListener, LoaderManager.LoaderCallbacks<Cursor> {
 
    // This is the Adapter being used to display the list's data.
    SimpleCursorAdapter mAdapter;
 
    // If non-null, this is the current filter the user has provided.
    String mCurFilter;
 
    @Override public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
 
        // Give some text to display if there is no data.  In a real
        // application this would come from a resource.
        setEmptyText("No phone numbers");
 
        // We have a menu item to show in action bar.
        setHasOptionsMenu(true);
 
        // Create an empty adapter we will use to display the loaded data.
        mAdapter = new SimpleCursorAdapter(getActivity(),
                android.R.layout.simple_list_item_2, null,
                new String[] { Contacts.DISPLAY_NAME, Contacts.CONTACT_STATUS },
                new int[] { android.R.id.text1, android.R.id.text2 }, 0);
        setListAdapter(mAdapter);
 
        // Prepare the loader.  Either re-connect with an existing one,
        // or start a new one.
        getLoaderManager().initLoader(0, null, this);
    }
 
    @Override public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        // Place an action bar item for searching.
        MenuItem item = menu.add("Search");
        item.setIcon(android.R.drawable.ic_menu_search);
        item.setShowAsAction(MenuItem.SHOW_AS_ACTION_IF_ROOM);
        SearchView sv = new SearchView(getActivity());
        sv.setOnQueryTextListener(this);
        item.setActionView(sv);
    }
 
    public boolean onQueryTextChange(String newText) {
        // Called when the action bar search text has changed.  Update
        // the search filter, and restart the loader to do a new query
        // with this filter.
        mCurFilter = !TextUtils.isEmpty(newText) ? newText : null;
        getLoaderManager().restartLoader(0, null, this);
        return true;
    }
 
    @Override public boolean onQueryTextSubmit(String query) {
        // Don't care about this.
        return true;
    }
 
    @Override public void onListItemClick(ListView l, View v, int position, long id) {
        // Insert desired behavior here.
        Log.i("FragmentComplexList", "Item clicked: " + id);
    }
 
    // These are the Contacts rows that we will retrieve.
    static final String[] CONTACTS_SUMMARY_PROJECTION = new String[] {
        Contacts._ID,
        Contacts.DISPLAY_NAME,
        Contacts.CONTACT_STATUS,
        Contacts.CONTACT_PRESENCE,
        Contacts.PHOTO_ID,
        Contacts.LOOKUP_KEY,
    };
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        // This is called when a new Loader needs to be created.  This
        // sample only has one Loader, so we don't care about the ID.
        // First, pick the base URI to use depending on whether we are
        // currently filtering.
        Uri baseUri;
        if (mCurFilter != null) {
            baseUri = Uri.withAppendedPath(Contacts.CONTENT_FILTER_URI,
                    Uri.encode(mCurFilter));
        } else {
            baseUri = Contacts.CONTENT_URI;
        }
 
        // Now create and return a CursorLoader that will take care of
        // creating a Cursor for the data being displayed.
        String select = "((" + Contacts.DISPLAY_NAME + " NOTNULL) AND ("
                + Contacts.HAS_PHONE_NUMBER + "=1) AND ("
                + Contacts.DISPLAY_NAME + " != '' ))";
        return new CursorLoader(getActivity(), baseUri,
                CONTACTS_SUMMARY_PROJECTION, select, null,
                Contacts.DISPLAY_NAME + " COLLATE LOCALIZED ASC");
    }
 
    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
        // Swap the new cursor in.  (The framework will take care of closing the
        // old cursor once we return.)
        mAdapter.swapCursor(data);
    }
 
    public void onLoaderReset(Loader<Cursor> loader) {
        // This is called when the last Cursor provided to onLoadFinished()
        // above is about to be closed.  We need to make sure we are no
        // longer using it.
        mAdapter.swapCursor(null);
    }
}
```

### 参考资源
1、https://blog.csdn.net/liuyifeng1920/article/details/53368103
2、https://developer.android.google.cn/reference/java/util/concurrent/BlockingQueue.html
3、https://www.cnblogs.com/ganchuanpu/p/7758485.html
4、https://www.cnblogs.com/makaruila/p/4869912.html
5、https://www.jianshu.com/p/9c10beaa1c95
6、https://www.jianshu.com/p/be97093783d6
7、书籍：《Efficient Anrdoid Threading》
8、https://blog.csdn.net/Wtoria/article/details/51983584