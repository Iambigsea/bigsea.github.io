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
Observable.create(new Observable.OnSubscribe<>()).subscribe(new Subscriber())在上一章已经讲过,这里就不赘言了,直接来到关键的两个方法subscribeOn(Schedulers.io())  和 .observeOn(AndroidSchedulers.mainThread()),我们先看下大概的结构
### 类图
