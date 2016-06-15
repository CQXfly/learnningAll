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


### 定时器
* NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:4.0
target:self
selector:@selector(backgroundThreadFire:) userInfo:nil
repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timerforMode:NSDefaultRunLoopMode];

### runloop observe

```
源是在合适的同步或异步事件发生时触发，而run loop观察者则是在run loop本身运行的特定时候触发。你可以使用run loop观察者来为处理某一特定事件或是进入休眠的线程做准备。你可以将run loop观察者和以下事件关联：
1.  Runloop入口
2.  Runloop何时处理一个定时器
3.  Runloop何时处理一个输入源
4.  Runloop何时进入睡眠状态
5.  Runloop何时被唤醒，但在唤醒之前要处理的事件
6.  Runloop终止
```

```
- (void)addObserverToCurrentRunloop
{
// The application uses garbage collection, so noautorelease pool is needed.
NSRunLoop*myRunLoop = [NSRunLoop currentRunLoop];

// Create a run loop observer and attach it to the runloop.
CFRunLoopObserverContext  context = {0, self, NULL, NULL, NULL};
CFRunLoopObserverRef    observer =CFRunLoopObserverCreate(kCFAllocatorDefault,
kCFRunLoopBeforeTimers, YES, 0, &myRunLoopObserver, &context);

if (observer)
{
CFRunLoopRef    cfLoop = [myRunLoop getCFRunLoop];
CFRunLoopAddObserver(cfLoop, observer, kCFRunLoopDefaultMode);
}
}
```

```
每次运行run loop，你线程的run loop对会自动处理之前未处理的消息，并通知相关的观察者。具体的顺序如下：
通知观察者run loop已经启动
通知观察者任何即将要开始的定时器
通知观察者任何即将启动的非基于端口的源
启动任何准备好的非基于端口的源
如果基于端口的源准备好并处于等待状态，立即启动；并进入步骤9。
通知观察者线程进入休眠
将线程置于休眠直到任一下面的事件发生：
某一事件到达基于端口的源
定时器启动
Run loop设置的时间已经超时
run loop被显式唤醒
通知观察者线程将被唤醒。
处理未处理的事件
如果用户定义的定时器启动，处理定时器事件并重启run loop。进入步骤2
如果输入源启动，传递相应的消息
如果run loop被显式唤醒而且时间还没超时，重启run loop。进入步骤2
通知观察者run loop结束。
```
