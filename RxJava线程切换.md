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
