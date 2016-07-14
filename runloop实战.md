### 学习runloop这么久了 该实战演练一下了
#### 1 . _对于下载任务比较多的界面可以使用_

对于列表多图下载 本身这些下载任务都放在一个runloop中 可以拆分至多个runloop
```
- (void)addTask:(QXRunLoopTaskManagerUnit)unit withKey:(id)key {
    [self.tasks addObject:unit];
    [self.tasksKeys addObject:key];
    
    if (self.tasks.count > self.maxQueueCount) {
        [self.tasks removeObjectAtIndex:0];
        [self.tasksKeys removeObjectAtIndex:0];
    }
}
```
QXRunLoopTaskManagerUnit block 是这样的
```
BOOL(^QXRunLoopTaskManagerUnit)(void)

```

该方法是将该block存入我们的tasks池中 同时配对一个key 根据key去执行对应的blocl

#下面的方法是用来注册这个runloop#
```

+ (void)_registerRunLoopTaskAsMainRunloopObserver:(QXRunloopTaskManager *) manager {
    static CFRunLoopObserverRef defaultModeObsever;
   _registerObserver(kCFRunLoopBeforeWaiting, defaultModeObsever, NSIntegerMax - 999, kCFRunLoopDefaultMode, (__bridge void *)manager, &_defaultModeRunLoopTaskManagerCallback);
 }
```

```
tatic void _registerObserver(CFOptionFlags activities, CFRunLoopObserverRef observer, CFIndex order, CFStringRef mode, void *info, CFRunLoopObserverCallBack callback) {
    CFRunLoopRef runLoop = CFRunLoopGetCurrent();
    CFRunLoopObserverContext context = {
        0,
        info,
        &CFRetain,
        &CFRelease,
        NULL
    };
    observer = CFRunLoopObserverCreate(     NULL,
                                       activities,
                                       YES,
                                       order,
                                       callback,
                                       &context);
    CFRunLoopAddObserver(runLoop, observer, mode);
    CFRelease(observer);
}

static void _runLoopTaskManagerCallback(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info)
{
    QXRunloopTaskManager *manager = (__bridge QXRunloopTaskManager *)info;
    if (manager.tasks.count == 0) {
        return;
    }
    BOOL result = NO;
    while (result == NO && manager.tasks.count) {
        QXRunLoopTaskManagerUnit unit  = manager.tasks.firstObject;
        result = unit();
        [manager.tasks removeObjectAtIndex:0];
        [manager.tasksKeys removeObjectAtIndex:0];
    }
}

```

注册函数中传入一个函数指针
_defaultModeRunLoopTaskManagerCallback 
```
static void _defaultModeRunLoopTaskManagerCallback(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info) {
    _runLoopTaskManagerCallback(observer, activity, info);
}
```
这个函数是runloop的回掉 在这个回掉中执行taskmanagercallback

核心代码就是这些 以后会继续添加runloop相关的使用
