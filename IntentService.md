**Android中I****ntentService****与Service的区别** 


**Android中的Service是用于后台服务的，当应用程序被挂到后台的时候，问了保证应用某些组件仍然可以工作而引入了Service这个概念，那么这里面要强调的是Service不是独立的进程，也不是独立的线程，它是依赖于应用程序的主线程的，也就是说，在更多时候不建议在Service中编写耗时的逻辑和操作，否则会引起ANR。

**那么我们当我们编写的耗时逻辑，不得不被service来管理的时候，就需要引入IntentService，IntentService是继承Service的，那么它包含了Service的全部特性，当然也包含service的生命周期，那么与service不同的是，IntentService在执行onCreate操作的时候，内部开了一个线程，去你执行你的耗时操作。

1. Service中提供了一个方法：
   `    public int onStartCommand(Intent intent, int flags, int startId) {  
         onStart(intent, startId);  
         return mStartCompatibility ? START_STICKY_COMPATIBILITY : START_STICKY;  
     }  `
1. 这个方法的具体含义是，当你的需要这个service启动的时候，或者调用这个servcie的时候，那么这个方法首先是要被回调的。
1. 同时IntentService中提供了这么一个方法：
   `    protected abstract void onHandleIntent(Intent intent);  `
1. 这是一个抽象方法，也就是说具体的实现需要被延伸到子类。
2. 上面提到过IntentService是继承Service的，那么这个子类也肯定继承service，那么onHandleIntent()方法是什么时候被调用的呢？让我们具体看IntentService的内部实现：
 
    ` public void onCreate() { ` 
   ` // TODO: It would be nice to have an option to hold a partial wakelock  ` 
   `  // during processing, and to have a static startService(Context, Intent) `  
    ` // method that would launch the service & hand off a wakelock.  ` 
  
    ` super.onCreate(); `  
   `  HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");  ` 
   `  thread.start();  ` 
  
   	`  mServiceLooper = thread.getLooper();  ` 
   `  mServiceHandler = new ServiceHandler(mServiceLooper);  ` 

 	  }
1. 在这里我们可以清楚的看到其实IntentService在执行onCreate的方法的时候，其实开了一个线程HandlerThread,并获得了当前线程队列管理的looper，并且在onStart的时候，把消息置入了消息队列
2. 在消息被handler接受并且回调的时候，执行了onHandlerIntent方法，该方法的实现是子类去做的。

**结论：**

	IntentService是通过Handler looper message的方式实现了一个多线程的操作，同时耗时操作也可以被这个线程管理和执行，同时不会产生ANR的情况。