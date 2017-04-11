RxJava解析1
=========
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

![image](https://raw.githubusercontent.com/Iambigsea/bigsea.github.io/master/subscribe.png)

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
	
好吧，什么都没干，就是直接吧你传进来的obSubscribe对象给返回给你，不过这个是个抽象类，看看我们使用的hook对象有没有去复写这个方法，这里hook是通过

        static final RxJavaObservableExecutionHook hook = RxJavaPlugins.getInstance().getObservableExecutionHook();

来获取到的，点击去看看

	public class RxJavaPlugins {
		private final static RxJavaPlugins INSTANCE = new RxJavaPlugins();
		private final AtomicReference<RxJavaObservableExecutionHook> observableExecutionHook = new AtomicReference<RxJavaObservableExecutionHook>();
		public static RxJavaPlugins getInstance() {
			return INSTANCE;
		}
		RxJavaPlugins() {
		}
		......
		public RxJavaObservableExecutionHook getObservableExecutionHook() {
			if (observableExecutionHook.get() == null) {
				// check for an implementation from System.getProperty first
				Object impl = getPluginImplementationViaProperty(RxJavaObservableExecutionHook.class, System.getProperties());
				if (impl == null) {
					observableExecutionHook.compareAndSet(null, RxJavaObservableExecutionHookDefault.getInstance());
				} else {
					observableExecutionHook.compareAndSet(null, (RxJavaObservableExecutionHook) impl);
				}
			}
			return observableExecutionHook.get();
		}
		public void registerObservableExecutionHook(RxJavaObservableExecutionHook impl) {
			if (!observableExecutionHook.compareAndSet(null, impl)) {
				throw new IllegalStateException("Another strategy was already registered: " + observableExecutionHook.get());
			}
		}
		static Object getPluginImplementationViaProperty(Class<?> pluginClass, Properties props) {
			final String classSimpleName = pluginClass.getSimpleName();     
			final String pluginPrefix = "rxjava.plugin.";        
			String defaultKey = pluginPrefix + classSimpleName + ".implementation";
			String implementingClass = props.getProperty(defaultKey);
			if (implementingClass == null) {
				final String classSuffix = ".class";
				final String implSuffix = ".impl";    
				for (Map.Entry<Object, Object> e : props.entrySet()) {
					String key = e.getKey().toString();
					if (key.startsWith(pluginPrefix) && key.endsWith(classSuffix)) {
						String value = e.getValue().toString();                    
						if (classSimpleName.equals(value)) {
							String index = key.substring(0, key.length() - classSuffix.length()).substring(pluginPrefix.length());                        
							String implKey = pluginPrefix + index + implSuffix;                        
							implementingClass = props.getProperty(implKey);                        
							if (implementingClass == null) {
								throw new RuntimeException("Implementing class declaration for " + classSimpleName + " missing: " + implKey);
							}                        
							break;
						}
					}
				}
			}
			if (implementingClass != null) {
				try {
					Class<?> cls = Class.forName(implementingClass);
					// narrow the scope (cast) to the type we're expecting
					cls = cls.asSubclass(pluginClass);
					return cls.newInstance();
				} catch (ClassCastException e) {
					throw new RuntimeException(classSimpleName + " implementation is not an instance of " + classSimpleName + ": " + implementingClass);
				} catch (ClassNotFoundException e) {
					throw new RuntimeException(classSimpleName + " implementation class not found: " + implementingClass, e);
				} catch (InstantiationException e) {
					throw new RuntimeException(classSimpleName + " implementation not able to be instantiated: " + implementingClass, e);
				} catch (IllegalAccessException e) {
					throw new RuntimeException(classSimpleName + " implementation not able to be accessed: " + implementingClass, e);
				}
			}
			return null;
		}
		......
	}

RxJavaPlugins.getInstance()就是获取RxJavaPlugins的单例对象，然后再调用getObservableExecutionHook()方法来获取到RxJavaObservableExecutionHook对象，observableExecutionHook是AtomicReference<RxJavaObservableExecutionHook>类型的，那这个AtomicReference是什么鬼呢，它其实就是一个保持数据在多线程之间同步的，就是说无论有几个线程去获取或设置它，获取到的都是唯一一个对象，不会又数据不同步的问题。if (observableExecutionHook.get() == null) {一开始肯定是为空的，因为没设置过，然后Object impl = getPluginImplementationViaProperty(RxJavaObservableExecutionHook.class, System.getProperties());我们也没有去配置RxJavaObservableExecutionHook，所以这个impl肯定是为空的，所以会去执行observableExecutionHook.compareAndSet(null, RxJavaObservableExecutionHookDefault.getInstance())方法compareAndSet(null, RxJavaObservableExecutionHookDefault.getInstance())的意思就是如果impl为null，就把RxJavaObservableExecutionHookDefault.getInstance()设置到
observableExecutionHook里面，所以我们的hook对象就是通过RxJavaObservableExecutionHookDefault.getInstance()来获取到的，再点进去看看

	class RxJavaObservableExecutionHookDefault extends RxJavaObservableExecutionHook {

		private static RxJavaObservableExecutionHookDefault INSTANCE = new RxJavaObservableExecutionHookDefault();

		public static RxJavaObservableExecutionHook getInstance() {
			return INSTANCE;
		}

	}

它就是继承RxJavaObservableExecutionHook抽象类，然后什么都没干，就提供了一个静态方法获取这个对象，其实就是一个RxJavaObservableExecutionHook的单利写法，所以我们的hook对象就是RxJavaObservableExecutionHook对象，所以我们之前调用的方法


        public abstract class RxJavaObservableExecutionHook {
	        ......
                public <T> OnSubscribe<T> onSubscribeStart(Observable<? extends T> observableInstance, final OnSubscribe<T> onSubscribe) {
                // pass-thru by default
                return onSubscribe;
                }
	        ......
        }
	
就是这个方法，这个方法就是什么都没干，就把你传进来的onSubscribe给返回了给你

	hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber);

所以这里就是调用了observable.onSubscribe的call方法，这个onSubscribe就是

        Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("hello RxJava");
                subscriber.onCompleted();
            }
        })

就是我们new的这个OnSubscribe，而hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber)的call里面的subscriber就是我们

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
	
的这个玩意，所以


        Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("hello RxJava");
                subscriber.onCompleted();
            }
        })

我们的"hello RxJava"就通过subscriber.onNext("hello RxJava");跑到我们的

            @Override
            public void onNext(String s) {
                Log.i(TAG,s);
            }

里面了，简化一下就是这样

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


o了，就到这里
