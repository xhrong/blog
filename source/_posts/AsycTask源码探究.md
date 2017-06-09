---
title: AsycTask源码探究 
tags: [Android,AsycTask]
grammar_cjkRuby: true
categories: [Android]
date: 2017-01-08
---


```java
    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     */
    public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                Result result = doInBackground(mParams);
                Binder.flushPendingCommands();
                return postResult(result);
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
```

mWorker是个Callable对象，用来在子线程中执行doInBackground()内的耗时操作

mFuture是FutureTask对象，通过重写done()方法，获取doInBackground()的执行结果。并通过  postResultIfNotInvoked(get())将执行结果发送给主线程。

注意，done()方法仍然是在子线程中执行的，所以不会其内部调用get()方法，不会阻塞主线程


 ```java
     private void postResultIfNotInvoked(Result result) {
        final boolean wasTaskInvoked = mTaskInvoked.get();
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }

    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
	
	    private static Handler getHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler();
            }
            return sHandler;
        }
    }
	
	    private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
```
	
postResultIfNotInvoked() 实质上是利用Android的Handler机制进行线程切换。getHandler()所得到的Handler对象关联的是主线程的消息队列。progress信息也是通过这个Handler传给主线程的。

```
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
	
    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
AsycTask采用了重新定义的SerialExecutor类，虽然采用了线程池，但是由于总是run()之后调用scheduleNext()，导致任务也单线程排队的方式执行。而且这种排队是全局性的。所以AsycTask并不适合执行长时间的任务，否则会导致后续任务长时间排队，无法及时获取结果。


Tips：

1、AsycTask默认采用单线程排队的方式执行，不适合执行长时间的任务，否则会导致后续任务长时间排队，无法及时获取结果。（当然可以进行配置的）

	