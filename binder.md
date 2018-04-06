# [文章来之大神博客](http://qiangbo.space/)
## bindlr驱动层
#### binder机制简介
android进程间通信的方式有多种，比如管道，socket（zygote启动进程），共享内存，信号量，消息队列等<br>
Binder相对于其他传统的ipc方式更加适合android，原因有三点：
* binder是基于c/s架构的，这一点更加符合安卓的系统架构
* 性能上更加有优势，管道，消息队列，socket都需要拷贝两次内存，binder只需要拷贝一次，对于底层的ipc形式，少拷贝一次对性能的影响是非常大的
* 安全性更好：对于传统的ipc是无法识别到对方的身份标识，（uid、pid），而binder机制是跟随调用过程而自动传递的，server端很容易知道client的身份，非常便于做安全检查
#### 整体架构
![](http://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2017-01-15-AndroidAnatomy_Binder/Binder_Architecture.png)
从图中可以看出，binder的实现可以分为这么几层：
* Framwork 层
    * java部分
    * jni部分
    * c++部分
* 驱动层
驱动层位于linux内核中，它提供了最底层的数据传递、对象标识、线程管理、调用过程控制等功能。驱动层是整个binder机制的核心。<bar>
Framework层以驱动层为基础提供了应用开发的基础设施。<br>
Framework层即包括了c++实现的部分，又包括了java实现的部分。为了能将c++的实现复现到Java端，中间通过JNI进行衔接。
一个client调用server的请求的调用过程如下图所示：
![](http://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2017-01-15-AndroidAnatomy_Binder/binder_layer.png) 
对于网络协议有所了解的读者会发现，这个数据的传递过程和网络协议是如此相似。
#### 初始ServiceManager
ServiceManger是Binder框架中负责管理各个Service的。每个公开对外提供服务的service都需要注册到ServiceManager中（通过addService），注册的时候需要指定一个唯一的id（其实就是一个字符串）。<br>
Clinet要多server发出请求，就必须知道服务端的id。client需要先更具server的id通过ServiceManager拿到Server的引用（通过getService），整个过程如下图所示
![](http://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2017-01-15-AndroidAnatomy_Binder/binder_servicemanager.png) 
### 驱动层
驱动层中有3个重要的函数binder_open,binder_mmap,binder_ioctl,这是因为，需要使用binder的进程，几乎总是通过binder_open来打开binder设备，然后通过binder_mmap来进行内存映射，在这之后，通过ioctl来进行实际操作。client对于server端的请求，以及Server对client请求结果的返回，都是通过ioctl来完成的。这里提到的流程图如下所示：
![](http://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2017-01-15-AndroidAnatomy_Binder/binder_driver_interface.png)
### 主要结构
binder驱动中的结构体可以分为两类：<br>
一类是与用户空间共用的，这类结构体在Binder通信协议过程中会用到。因此，这些结构体定义在binder.h中，包括：<br>
这其中，binder_write_read和binder_transaction_data这两个结构体最为重要，它们存储了IPC调用过程中的数据<br>
还要一类结构体是Binder驱动内部使用的，
