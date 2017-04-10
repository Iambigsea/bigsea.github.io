RxJava解析
========
首先来看一下最基本的使用方式

        Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("hello RxJava");
                subscriber.onCompleted();
            }
        }).subscribe(new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(TAG,s);
            }
        });

写成这样的话就可以把字符串“hello RxJava”转到onNext打印出来。OnSubscribe里面的"hello RxJava"怎么会跑到Subscriber的onnext方法的参数s里面的呢？
这里先把类图上传上去

这里大概能看到一些类的关系，我们来看看具体源码
首先是Observable.create()

        Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("hello RxJava");
                subscriber.onCompleted();
            }
        })
点击看看到的源码是这样的

        public class Observable<T> {

            final OnSubscribe<T> onSubscribe;

                protected Observable(OnSubscribe<T> f) {
                this.onSubscribe = f;
            }

                public static <T> Observable<T> create(OnSubscribe<T> f) {
                return new Observable<T>(hook.onCreate(f));
            }
	        .......
        }
这里把多余的都删了。这里是通过调用Obserable的静态方法create来创建一个Obserable对象，这里把创建的OnSubscribe对象作为observable的构造函数参数传进去，并把值赋给成员变量onSubscribe。这里就创建Observable对象并给它的成员变量赋值onSubscribe赋值。
再来看下subscribe()方法

        subscribe(new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(TAG,s);
            }
        })
点进去看源码是这样的

	public class Observable<T> {

		static final RxJavaObservableExecutionHook hook = RxJavaPlugins.getInstance().getObservableExecutionHook();

		......
		public final Subscription subscribe(Subscriber<? super T> subscriber) {
			return Observable.subscribe(subscriber, this);
		}
		
		private static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {
			if (subscriber == null) {
				throw new IllegalArgumentException("observer can not be null");
			}
			if (observable.onSubscribe == null) {
				throw new IllegalStateException("onSubscribe function can not be null.");
			}
			subscriber.onStart();
			if (!(subscriber instanceof SafeSubscriber)) {
				// assign to `observer` so we return the protected version
				subscriber = new SafeSubscriber<T>(subscriber);
			}
			try {
				hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber);
				return hook.onSubscribeReturn(subscriber);
			} catch (Throwable e) {
				Exceptions.throwIfFatal(e);
				try {
					subscriber.onError(hook.onSubscribeError(e));
				} catch (Throwable e2) {
					Exceptions.throwIfFatal(e2);
					RuntimeException r = new RuntimeException("Error occurred attempting to subscribe [" + e.getMessage() + "] and then again while trying to pass to onError.", e2);
					hook.onSubscribeError(r);
					throw r;
				}
				return Subscriptions.unsubscribed();
			}
		}
		.......
	}
subscribe(Subscriber<? super T> subscriber)去调用重载方法Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable)，把传进来的subscriber和上一步创建的Observable对象做为参数穿进去，这个各种非法校验，合法之后了就调用subscribe的空模板方法onStart(),然后调用hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber);这个hook是什么鬼，它的onSubscribeStart()是干嘛的呢，不用慌，点进去see see

        public abstract class RxJavaObservableExecutionHook {
	        ......
                public <T> OnSubscribe<T> onSubscribeStart(Observable<? extends T> observableInstance, final OnSubscribe<T> onSubscribe) {
                // pass-thru by default
                return onSubscribe;
                }
	        ......
        }
好吧，什么都没干，就是直接吧你传进来的obSubscribe对象给返回给你，不过这个是个抽象类，看看我们使用的hook对象有没有去复写这个方法

        static final RxJavaObservableExecutionHook hook = RxJavaPlugins.getInstance().getObservableExecutionHook();

