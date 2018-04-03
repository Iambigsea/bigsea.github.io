## bindlr驱动层
#### binder机制简介
android进程间通信的方式有多种，比如管道，socket（zygote启动进程），共享内存，信号量，消息队列等<br>
Binder相对于其他传统的ipc方式更加适合android，原因有三点：
* binder是基于c/s架构的，这一点更加符合安卓的系统架构
* 性能上更加有优势，管道，消息队列，socket都需要拷贝两次内存，binder只需要拷贝一次，对于底层的ipc形式，少拷贝一次对性能的影响是非常大的
* 安全性更好：对于传统的ipc是无法识别到对方的身份标识，（uid、pid），而binder机制是跟随调用过程而自动传递的，server端很容易知道client的身份，非常便于做安全检查
#### 整体架构
![](http://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2017-01-15-AndroidAnatomy_Binder/Binder_Architecture.png)
