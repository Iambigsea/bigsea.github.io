RxJava解析
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
这里先把类图xie
