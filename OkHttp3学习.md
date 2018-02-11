Android在6.0之后取消了httpclient官方包支持，在4.4之后httpurlconection是用okhttp实现的，这个开源库的控件的鼎鼎大名就不用多说了，那它有什么优点呢，怎么使用呢，还有简单的源码分析是怎么样的
## 优点
1、支持Http2/SPDY；
2、默认启用长连接，使用连接池管理，支持Cache(目前仅支持GET请求的缓存)；
3、路由节点管理，提升访问速度；
4、透明的Gzip处理，节省网络流量。
5、灵活的拦截器，行为类似Java EE的Filter或者函数编程中的Map。
## 主框架分析
我们都知道在OkHttp3中，其灵活性，很大程度上体现在，我们可以intercept其任意一个环节，而这个优势便是okhttp3整个请求响应架构体系的精髓所在：
a、在okhttp3中，每一个请求任务都封装为一个call，其实现为RealCall。
b、而所有的策略几乎都可以通过OkHttpClient传入
c、所有全局策略与数据，除了存储在允许上层访问的OkHttpClient实例外，还有一部分是存储在只允许包可见的Internal.instance中（比如连接池、路由黑名单等）
d、okhttp中用户看传入的interceptor分为两类，一类是全局interceptor，该类interceptor在请求之前最早被调用，另一类为非网页请求的networkInterceptor，这类interceptor只有在飞网页请求中会被调用，并且是在组装完成请求之后，正在发起请求之前被调用
e、整个请求过程通过递归RealInterceptorChain#proceed来实现，在每个interceptor中调用递归到下一个interceptor来完成整个请求流程，并且在递归回到当前iterceptor后完成相应处理
f、在异步请求中，我们通过Callback来获得简单清晰的请求回调（onFailure、onResponse）
g、但是在OkHttpClient中，我们可以传入EventListener的工厂方法，为每一个请求创建一个EventLinster，来接受非常细的事件回调
[https://blog.dreamtobe.cn/img/okhttp3-call-flowchart.png](请求调用流程)

