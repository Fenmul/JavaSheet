# 并发进阶01_Java 线程模型

synchronized 关键字在 1.6 以前是使用的操作系统中的互斥锁也就是重量级锁，在 1.6 以后是使用的 JDK 实现的轻量锁。


linux 中创建线程
```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, 
                   void *(*start_routine) (void *), void *arg);
```
为什么 Java 启动线程是使用 start 方法最终执行的方法却是 run 方法呢？
start() -> start0 (native 方法) -> c -> pthread_create 创建线程（java_start 第三个参数）-> java_start JNI反射回调 java 中的 run 方法。


所以 Java 中的线程和linux 中的线程是一一对应的关系，run 方法中的内容才是线程真正执行的内容。


