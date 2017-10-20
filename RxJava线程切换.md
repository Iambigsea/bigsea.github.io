    RxJava在安卓开发中常用的情景就是在子线程中获取数据,然后再主线程中更新内容.代码示例应该是这样的
### 基本使用
```Java
private void scheduleThread() {
        Log.d(TAG, "Main thread name : " + Thread.currentThread().getId());
        Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                Log.d(TAG, "Observable thread name : " + Thread.currentThread().getId());
                subscriber.onNext("test");
                subscriber.onCompleted();
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<String>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(String s) {
                        Log.d(TAG, "Subscriber thread name : " + Thread.currentThread().getId());
                    }
                });
    }
```
输出结果是这样的
```Java
Main thread name : 1
Observable thread name : 673
Subscriber thread name : 1
```
可以看到我们在Observable.OnSubscrib.call()方法中我们由主线程切到了子线程中处理的,Subscriber.onNext()是切回到了主线程中
那么他到底是怎么完成这个切换的呢?这个就是我们今天要做的事情
```Java
    Observable.create(new Observable.OnSubscribe<>()).subscribe(new Subscriber())
```
在上一章已经讲过,这里就不赘言了,直接来到关键的两个方法
```Java
    subscribeOn(Schedulers.io()) 
    observeOn(AndroidSchedulers.mainThread()),
```
我们先看下大概的结构
### 类图

### 源码
看过类图,我们直接来看源码
直接点击线程切换的方法
```Java
    .subscribeOn(Schedulers.io())
```
调到RxJava的control类Observable中
```Java
    public final Observable<T> subscribeOn(Scheduler scheduler) {
        return subscribeOn(scheduler, !(this.onSubscribe instanceof OnSubscribeCreate));
    }
```
这里接收的是我们的Scheduler对象Schedulers.io(),先不管他继续看到里面的方法
```Java
    public final Observable<T> subscribeOn(Scheduler scheduler, boolean requestOn) {
        if (this instanceof ScalarSynchronousObservable) {
            return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
        }
        return unsafeCreate(new OperatorSubscribeOn<T>(this, scheduler, requestOn));
    }
```
这里会判断下我们的scheduler是不是ScalarSynchronousObservable对象,我们可以先看看是不是这个对象,这个是看类型的,我们看下它的父类和实现了这个接口没有就可以直接点进去
```Java
    Schedulers.io()
```
这里调用的是Schedulers里面的静态方法
```Java
    public static Scheduler io() {
        return RxJavaHooks.onIOScheduler(getInstance().ioScheduler);
    }
```
查看onIOScheduler()方法
```Java
    public static Scheduler onIOScheduler(Scheduler scheduler) {
        Func1<Scheduler, Scheduler> f = onIOScheduler;
        if (f != null) {
            return f.call(scheduler);
        }
        return scheduler;
    }
```
这个onIOScheduler是否为空,这个对象是在类RxJavaHooks里面的,查看了下,这里面并没有对这个对象赋值,所以我们的Scheduler是通过getInstance().ioScheduler来创建的,这里说下RxJavaHooks,RxJavaPlugins这两个类,RxJavaPlugins是生成这个配置的地方,通过cas类防止多线程的问题,生成各种全局的Hook钩子,然后RxJavaHooks是装饰一下,装饰RxJavaPlugins的方法,生成各个具体的Hook的工具类方法,这个选不管,我们继续之前的话题,我们看看getInstance().ioScheduler到底是什么东东
```Java
private Schedulers() {
        @SuppressWarnings("deprecation")
        RxJavaSchedulersHook hook = RxJavaPlugins.getInstance().getSchedulersHook();

        Scheduler c = hook.getComputationScheduler();
        if (c != null) {
            computationScheduler = c;
        } else {
            computationScheduler = RxJavaSchedulersHook.createComputationScheduler();
        }

        Scheduler io = hook.getIOScheduler();
        if (io != null) {
            ioScheduler = io;
        } else {
            ioScheduler = RxJavaSchedulersHook.createIoScheduler();
        }

        Scheduler nt = hook.getNewThreadScheduler();
        if (nt != null) {
            newThreadScheduler = nt;
        } else {
            newThreadScheduler = RxJavaSchedulersHook.createNewThreadScheduler();
        }
    }
```
在Schedulers的构造函数中,对他进行赋值了,我们仔细看看它里面的方法,这里的RxJavaSchedulersHook又是个什么东东
```Java
public RxJavaSchedulersHook getSchedulersHook() {
        if (schedulersHook.get() == null) {
            // check for an implementation from System.getProperty first
            Object impl = getPluginImplementationViaProperty(RxJavaSchedulersHook.class, System.getProperties());
            if (impl == null) {
                // nothing set via properties so initialize with default
                schedulersHook.compareAndSet(null, RxJavaSchedulersHook.getDefaultInstance());
                // we don't return from here but call get() again in case of thread-race so the winner will always get returned
            } else {
                // we received an implementation from the system property so use it
                schedulersHook.compareAndSet(null, (RxJavaSchedulersHook) impl);
            }
        }
        return schedulersHook.get();
    }
```
这里就是我们说的生成各个hook的方法的类,这里是没有写配置的,所以Object impl为空,所以我们的RxJavaSchedulersHook对象是通过
```Java
    RxJavaSchedulersHook.getDefaultInstance()
```
来生成的
```Java
    public static RxJavaSchedulersHook getDefaultInstance() {
        return DEFAULT_INSTANCE;
    }
    
    private final static RxJavaSchedulersHook DEFAULT_INSTANCE = new RxJavaSchedulersHook();

```
点进去看,其实也没什么东西,绕了半天,其实就是new 的一个RxJavaSchedulersHook对象,尴尬,好,我们回去Schedulers的构造函数
```Java
        Scheduler io = hook.getIOScheduler();
        if (io != null) {
            ioScheduler = io;
        } else {
            ioScheduler = RxJavaSchedulersHook.createIoScheduler();
        }
```
```Java
    public Scheduler getIOScheduler() {
        return null;
    }
```
hook.getIOScheduler()返回的对象也是null的,所以就走到了下面这句
```Java
    public static Scheduler createIoScheduler() {
        return createIoScheduler(new RxThreadFactory("RxIoScheduler-"));
    }
```
```Java
    public static Scheduler createIoScheduler(ThreadFactory threadFactory) {
        if (threadFactory == null) {
            throw new NullPointerException("threadFactory == null");
        }
        return new CachedThreadScheduler(threadFactory);
    }
```
好了,真相大白,我们的scheduler其实就是这个CachedThreadScheduler,我们看看它的父类和实现对象
```Java
   public final class CachedThreadScheduler extends Scheduler implements SchedulerLifecycle
```
好吧,它并不是ScalarSynchronousObservable对象,我们回到之前的代码
```Java
    public final Observable<T> subscribeOn(Scheduler scheduler, boolean requestOn) {
        if (this instanceof ScalarSynchronousObservable) {
            return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
        }
        return unsafeCreate(new OperatorSubscribeOn<T>(this, scheduler, requestOn));
    }
```
很清晰了,这里走的是unsafeCreate方法,继续进去
```Java
    public static <T> Observable<T> unsafeCreate(OnSubscribe<T> f) {
        return new Observable<T>(RxJavaHooks.onCreate(f));
    }
```
这里就是返回一个new Observable对象,里面的f就是我们上面传过来的new OperatorSubscribeOn<T>(this, scheduler, requestOn),这里我们先不管这个是干嘛的,我们直接往后看,上节我们说过了基本的简单用法的流程
 ```Java
 Subscriber<String> subscriber = new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(TAG, s);
            }
        };
        Observable.OnSubscribe  onSubscribe= new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("hello RxJava");
                subscriber.onCompleted();
            }
        };
	onSubscribe.call(subscriber);
 ```
 我们这里的Subscriber已经知道了,那么到底哪个是Observable.OnSubscribe呢?答案就在这里
 ```Java
     public final Observable<T> subscribeOn(Scheduler scheduler, boolean requestOn) {
        if (this instanceof ScalarSynchronousObservable) {
            return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
        }
        return unsafeCreate(new OperatorSubscribeOn<T>(this, scheduler, requestOn));
    }
 ```
 就是这个OperatorSubscribeOn对象,this就是我们一开始创建的,做具体事情的Observable.OnSubscribe对象,我们继续去看它的庐山真面目,因为我们已经知道是调用它的call方法来执行的,所以我们直接看它的call方法即可
 ```Java
 public final class OperatorSubscribeOn<T> implements OnSubscribe<T> {

    final Scheduler scheduler;
    final Observable<T> source;
    final boolean requestOn;

    public OperatorSubscribeOn(Observable<T> source, Scheduler scheduler, boolean requestOn) {
        this.scheduler = scheduler;
        this.source = source;
        this.requestOn = requestOn;
    }

    @Override
    public void call(final Subscriber<? super T> subscriber) {
        final Worker inner = scheduler.createWorker();

        SubscribeOnSubscriber<T> parent = new SubscribeOnSubscriber<T>(subscriber, requestOn, inner, source);
        subscriber.add(parent);
        subscriber.add(inner);

        inner.schedule(parent);
    }
    ...
    }
 ```
 来仔细的look一下,这个scheduler就是我们之前找的CachedThreadScheduler对象,来看下它的createWorker()方法
 ```Java
     @Override
    public Worker createWorker() {
        return new EventLoopWorker(pool.get());
    }
 ```
好的,它返回了一个EventLoopWorker,其他的先不看,先到OperatorSubscribeOn.cal方法里面继续看,这里new SubscribeOnSubscriber对象然后调用了subscriber的add方法,把新建的2个对象add了进去
```Java
    public final void add(Subscription s) {
        subscriptions.add(s);
    }
   ...
    @Override
    public final void unsubscribe() {
        subscriptions.unsubscribe();
    }
    ...
    @Override
    public void unsubscribe() {
        if (!unsubscribed) {
            List<Subscription> list;
            synchronized (this) {
                if (unsubscribed) {
                    return;
                }
                unsubscribed = true;
                list = subscriptions;
                subscriptions = null;
            }
            // we will only get here once
            unsubscribeFromAll(list);
        }
    }
```
这个也没什么,我把主要的三个方法列出来了,其实就是把它放到一个专门存放这些Subscription的集合里面,方便在取消订阅的时候统一取消,继续看会之前的
 ```Java
 inner.schedule(parent);
 ```
 它调用了Worker inner的schedule方法,这个inner就是我们前面说的new的EventLoopWorker对象,所以调用的是EventLoopWorker的schedule方法
 ```Java		
    @Override
    public Subscription schedule(Action0 action) {
        return schedule(action, 0, null);
    }

    @Override
    public Subscription schedule(final Action0 action, long delayTime, TimeUnit unit) {
        if (innerSubscription.isUnsubscribed()) {
            // don't schedule, we are unsubscribed
            return Subscriptions.unsubscribed();
        }

        ScheduledAction s = threadWorker.scheduleActual(new Action0() {
            @Override
            public void call() {
                if (isUnsubscribed()) {
                    return;
                }
                action.call();
            }
        }, delayTime, unit);
        innerSubscription.add(s);
        s.addParent(innerSubscription);
        return s;
    }	
 ```
去除其他合法校验的东西,其实就核心代码就是这个
```Java
    ScheduledAction s = threadWorker.scheduleActual(new Action0() {
        @Override
        public void call() {
            if (isUnsubscribed()) {
               return;
            }
            action.call();
         }
    }, delayTime, unit);
```
其实就是调用threadWorker.scheduleActual()方法,这个threadWorker又是什么呢
```Java
    static final class EventLoopWorker extends Worker implements Action0 {
        private final CompositeSubscription innerSubscription = new CompositeSubscription();
        private final CachedWorkerPool pool;
        private final ThreadWorker threadWorker;
        final AtomicBoolean once;

        EventLoopWorker(CachedWorkerPool pool) {
            this.pool = pool;
            this.once = new AtomicBoolean();
            this.threadWorker = pool.get();
        }
	
	@Override
	public Worker createWorker() {
	    return new EventLoopWorker(pool.get());
	}
```
threadWorker是通过CachedWorkerPool.get()获取到的,这个pool就是我们CachedThreadScheduler的一个成员变量,我们来找找它
```Java
    public CachedThreadScheduler(ThreadFactory threadFactory) {
        this.threadFactory = threadFactory;
        this.pool = new AtomicReference<CachedWorkerPool>(NONE);
        start();
    }
```
这个pool是在我们创建CachedThreadScheduler对象时,在它的成员变量里面赋值的,是一个保证在多线程运行中保证数据同步的类AtomicReference的
类包裹的CachedWorkerPool类,一开始的值为NONE,这个null是在静态代码块里面赋值的
```Java
    static {
	......
        NONE = new CachedWorkerPool(null, 0, null);
        NONE.shutdown();
        ......
    }	
```
是一个CachedWorkerPool对象,先不看这个对象是干嘛的,因为这个变量的命名为NONE,所以应该是空值的默认值,回到前面CachedThreadScheduler的构造函数,里面在创建了pool之后还调用了start()方法,去看看它做了什么
```Java
    @Override
    public void start() {
        CachedWorkerPool update =
            new CachedWorkerPool(threadFactory, KEEP_ALIVE_TIME, KEEP_ALIVE_UNIT);
        if (!pool.compareAndSet(NONE, update)) {
            update.shutdown();
        }
    }	
```
没错,这里的start()就是对pool对象赋值了,通过cas方法来进行赋值,如果赋值不成功则废弃掉这个update对象
```Java
    void shutdown() {
        try {
            if (evictorTask != null) {
                evictorTask.cancel(true);
            }
            if (evictorService != null) {
                evictorService.shutdownNow();
            }
        } finally {
            allWorkers.unsubscribe();
        }
    }	
```
取消evictorTask,停止evictorService,取消订阅allWorkers里面的所有订阅者.好的,回到主路线,我们新建的update对象即是我们的pool对象,里面的threadWork就是通过这个pool.get()来获取的,我们来看这个这个pool到底是何方神圣
```Java
static final class CachedWorkerPool {
        private final ThreadFactory threadFactory;
        private final long keepAliveTime;
        private final ConcurrentLinkedQueue<ThreadWorker> expiringWorkerQueue;
        private final CompositeSubscription allWorkers;
        private final ScheduledExecutorService evictorService;
        private final Future<?> evictorTask;

        CachedWorkerPool(final ThreadFactory threadFactory, long keepAliveTime, TimeUnit unit) {
            this.threadFactory = threadFactory;
            this.keepAliveTime = unit != null ? unit.toNanos(keepAliveTime) : 0L;
            this.expiringWorkerQueue = new ConcurrentLinkedQueue<ThreadWorker>();
            this.allWorkers = new CompositeSubscription();

            ScheduledExecutorService evictor = null;
            Future<?> task = null;
            if (unit != null) {
                evictor = Executors.newScheduledThreadPool(1, new ThreadFactory() {
                    @Override public Thread newThread(Runnable r) {
                        Thread thread = threadFactory.newThread(r);
                        thread.setName(thread.getName() + " (Evictor)");
                        return thread;
                    }
                });
                NewThreadWorker.tryEnableCancelPolicy(evictor);
                task = evictor.scheduleWithFixedDelay(
                        new Runnable() {
                            @Override
                            public void run() {
                                evictExpiredWorkers();
                            }
                        }, this.keepAliveTime, this.keepAliveTime, TimeUnit.NANOSECONDS
                );
            }
            evictorService = evictor;
            evictorTask = task;
        }

        ThreadWorker get() {
            if (allWorkers.isUnsubscribed()) {
                return SHUTDOWN_THREADWORKER;
            }
            while (!expiringWorkerQueue.isEmpty()) {
                ThreadWorker threadWorker = expiringWorkerQueue.poll();
                if (threadWorker != null) {
                    return threadWorker;
                }
            }

            // No cached worker found, so create a new one.
            ThreadWorker w = new ThreadWorker(threadFactory);
            allWorkers.add(w);
            return w;
        }	
```
先看看构造函数,创建了一个周期性执行的线程池,在NONE的时候我们传入的threadFactory和unit为null,所以不会线程次和Future也为空,我们创建真正创建的pool对象最大单个线程可存活时间为60秒,task这个是evictor.scheduleWithFixedDelay()返回的一个Future的对象,里面的方法我们去看看
```Java
    void evictExpiredWorkers() {
            if (!expiringWorkerQueue.isEmpty()) {
                long currentTimestamp = now();

                for (ThreadWorker threadWorker : expiringWorkerQueue) {
                    if (threadWorker.getExpirationTime() <= currentTimestamp) {
                        if (expiringWorkerQueue.remove(threadWorker)) {
                            allWorkers.remove(threadWorker);
                        }
                    } else {
                        // Queue is ordered with the worker that will expire first in the beginning, so when we
                        // find a non-expired worker we can stop evicting.
                        break;
                    }
                }
            }
        }
```
就是用来清空expiringWorkerQueue队列和allWorkers队列,在看看get()方法,因为我们没有对expiringWorkerQueue进行add操作,所以是isEmpty为ture,就直接走到下一句创建ThreadWorker,并返回给调用方,这个w就是我们通过pool.get获取的ThreadWorker,好了ThreadWorker找到了,我们回去EventLoopWorker中,就是 threadWorker.scheduleActual(new Action0()),好的,我们进去ThreadWorker.scheduleActual()去看看
```Java
    public ScheduledAction scheduleActual(final Action0 action, long delayTime, TimeUnit unit) {
        Action0 decoratedAction = RxJavaHooks.onScheduledAction(action);
        ScheduledAction run = new ScheduledAction(decoratedAction);
        Future<?> f;
        if (delayTime <= 0) {
            f = executor.submit(run);
        } else {
            f = executor.schedule(run, delayTime, unit);
        }
        run.add(f);

        return run;
    }
```
先看看这个executor是什么鬼
```Java
        public NewThreadWorker(ThreadFactory threadFactory) {
        ScheduledExecutorService exec = Executors.newScheduledThreadPool(1, threadFactory);
        // Java 7+: cancelled future tasks can be removed from the executor thus avoiding memory leak
        boolean cancelSupported = tryEnableCancelPolicy(exec);
        if (!cancelSupported && exec instanceof ScheduledThreadPoolExecutor) {
            registerExecutor((ScheduledThreadPoolExecutor)exec);
        }
        executor = exec;
    }
```
这个也是一个ScheduledExecutorService线程池,再回到scheduleActual()方法去看,Action0 decoratedAction对象其实还是action对象自己,前面说过了,就不进去看了,我们知道了executor是个线程池,那么这里的核心方法应该在ScheduledAction.run()里面了
```Java
    public ScheduledAction(Action0 action) {
        this.action = action;
        this.cancel = new SubscriptionList();
    }
    
    @Override
    public void run() {
        try {
            lazySet(Thread.currentThread());
            action.call();
        } catch (OnErrorNotImplementedException e) {
            signalError(new IllegalStateException("Exception thrown on Scheduler.Worker thread. Add `onError` handling.", e));
        } catch (Throwable e) {
            signalError(new IllegalStateException("Fatal Exception thrown on Scheduler.Worker thread.", e));
        } finally {
            unsubscribe();
        }
    }
```
好的,看到这里真相大白了,我们要找的线程切换,其实就是在NewThreadWorker的够着函数中创建的线程池,还有ScheduledAction线程对象,对传进来的Action进行线程切换的,再理一下这个Action
```Java
public final class OperatorSubscribeOn<T> implements OnSubscribe<T> {

    final Scheduler scheduler;
    final Observable<T> source;
    final boolean requestOn;

    public OperatorSubscribeOn(Observable<T> source, Scheduler scheduler, boolean requestOn) {
        this.scheduler = scheduler;
        this.source = source;
        this.requestOn = requestOn;
    }

    @Override
    public void call(final Subscriber<? super T> subscriber) {
        final Worker inner = scheduler.createWorker();

        SubscribeOnSubscriber<T> parent = new SubscribeOnSubscriber<T>(subscriber, requestOn, inner, source);
        subscriber.add(parent);
        subscriber.add(inner);

        inner.schedule(parent);
    }
```
source就是我们通过Observable.OnSubscribe<String>()的对象,subscriber就是subscribe(new Subscriber<String>()对象,然后整个核心的类就在这里了
```Java
 static final class SubscribeOnSubscriber<T> extends Subscriber<T> implements Action0 {

        final Subscriber<? super T> actual;

        final boolean requestOn;

        final Worker worker;

        Observable<T> source;

        Thread t;

        SubscribeOnSubscriber(Subscriber<? super T> actual, boolean requestOn, Worker worker, Observable<T> source) {
            this.actual = actual;
            this.requestOn = requestOn;
            this.worker = worker;
            this.source = source;
        }

        @Override
        public void onNext(T t) {
            actual.onNext(t);
        }

        @Override
        public void onError(Throwable e) {
            try {
                actual.onError(e);
            } finally {
                worker.unsubscribe();
            }
        }

        @Override
        public void onCompleted() {
            try {
                actual.onCompleted();
            } finally {
                worker.unsubscribe();
            }
        }

        @Override
        public void call() {
            Observable<T> src = source;
            source = null;
            t = Thread.currentThread();
            src.unsafeSubscribe(this);
        }
    }
```
我们已经知道inner.schedule(parent)是在子线程调用parent的call方法,就是SubscribeOnSubscriber的call方法,再看下
```Java
src.unsafeSubscribe(this)
```
```Java
public final Subscription unsafeSubscribe(Subscriber<? super T> subscriber) {
        try {
            // new Subscriber so onStart it
            subscriber.onStart();
            // allow the hook to intercept and/or decorate
            RxJavaHooks.onObservableStart(this, onSubscribe).call(subscriber);
            return RxJavaHooks.onObservableReturn(subscriber);
        } catch (Throwable e) {
            // special handling for certain Throwable/Error/Exception types
            Exceptions.throwIfFatal(e);
            // if an unhandled error occurs executing the onSubscribe we will propagate it
            try {
                subscriber.onError(RxJavaHooks.onObservableError(e));
            } catch (Throwable e2) {
                Exceptions.throwIfFatal(e2);
                // if this happens it means the onError itself failed (perhaps an invalid function implementation)
                // so we are unable to propagate the error correctly and will just throw
                RuntimeException r = new OnErrorFailedException("Error occurred attempting to subscribe [" + e.getMessage() + "] and then again while trying to pass to onError.", e2);
                // TODO could the hook be the cause of the error in the on error handling.
                RxJavaHooks.onObservableError(r);
                // TODO why aren't we throwing the hook's return value.
                throw r; // NOPMD
            }
            return Subscriptions.unsubscribed();
        }
    }
```
就是我们上说过的,调用传进来的subscriber的onStart(),onError()方法,还有Observable.OnSubscribe<String>()的call方法.在call方法里面,我们又调用了subscriber的onNext(),onCompelete().也就是我们对应的
```Java
 @Override
        public void onNext(T t) {
            actual.onNext(t);
        }

        @Override
        public void onError(Throwable e) {
            try {
                actual.onError(e);
            } finally {
                worker.unsubscribe();
            }
        }

        @Override
        public void onCompleted() {
            try {
                actual.onCompleted();
            } finally {
                worker.unsubscribe();
            }
        }	
```
这个action也就是我们的subscribe(new Subscriber<String>())对象.好的,所有的流程我们已经讲完,我们最后再画张图表示一下
