#first learn runloop 
## 在主线程中添加代码

* 再来说说线程。有些线程执行的任务是一条直线，起点到终点；而另一些线程要干的活则是一个圆，不断循环，直到通过某种方式将它终止。直线线程如简单的Hello World，运行打印完,它的生命周期便结束了，像昙花一现那样；圆类型的如操作系统，一直运行直到你关机。在IOS中，圆型的线程就是通过run loop不停的循环实现的。
* Run loop，正如其名，loop表示某种循环，和run放在一起就表示一直在运行着的循环。实际上，run loop和线程是紧密相连的，可以这样说run loop是为了线程而生，没有线程，它就没有存在的必要。Run loops是线程的基础架构部分，Cocoa和CoreFundation都提供了run loop对象方便配置和管理线程的run loop（以下都已Cocoa为例）。每个线程，包括程序的主线程（main thread）都有与之相应的run loop对象。
* Cocoa中的NSRunLoop类并不是线程安全的
我们不能再一个线程中去操作另外一个线程的run loop对象，那很可能会造成意想不到的后果。不过幸运的是CoreFundation中的不透明类CFRunLoopRef是线程安全的，而且两种类型的run loop完全可以混合使用。Cocoa中的NSRunLoop类可以通过实例方法：
```
- (CFRunLoopRef)getCFRunLoop;
```
获取对应的CFRunLoopRef类，来达到线程安全的目的

```
while(1){
NSRunLoop * runLoop = [NsRunLoop currentRunLoop];
[runloop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
}
```

-------------------------
h1
zh
