#NSProgress
## 首先应该介绍下这个类 
* 苹果公司在 iOS 7 and OS X 10.9引入NSProgress类,目标是建立一个标准的机制用来报告长时间运行的任务的进度。NSProgress引入之后，其最重要的作用是可以在一个app的多个不需要紧耦合的模块之间产生进度报告。举个例子，一个运行在后台队列中的图片操作，这个操作应该能够把它的进度通知给一个视图控制器 （并且这个视图控制器也可以暂停或者终止该操作），甚至两个对象不可能持有对方的引用。

* 在发布说明中，苹果公司阐述了关于NSProgress四个主要的设计目的：
. 松耦合
. 组合性
. 重用性
. 可用性

* NSProgress对象活跃在一个层次结构中，该结构不同于UIKit下的视图层次结构。在该层次树的根部，UI图层可以建立一个进度对象而不论它何时想要监控任务的进度。借助于这样的对象（也就是所谓的在调用完成任务的方法之前的当前进度），这个对象将会自动成为任何被低级代码建立的子进度实例的父对象。在工作进行时，子进度会更新，同时更新也会被传递给父进度。因此，借助观察根进度对象（通过KVO）,UI 层能够显示子进度的组合进度。并且，还可以让根进度对象终止或者暂停，该UI 图层也有能力通过进度层次结构和执行代码交互。
 
和视图层次结构不同，不存在某种公共API，其可以从父到子或者从子到父贯穿进度层次结构。进度对象不需要考虑他们在层级树中的位置，以及他们是否有一个实际上的父或者子。事件传递和整体进度计算完全在后台完成。

* “当前进度”的思想（每个线程可以有其自己的当前进度）极大的解放了开发者，从而不用在不同的代码层中前后传递NSProgress（例如就像我们经常用NSError对象做的事情）。 这个设计考虑到了一个事实，就是显示进度的代码（就是UI）经常是从做实际执行的代码中被剥离出来的多个层次。另一方面，看起就像代码的风格。当前进度是基于一个线程自身的全局变量，有时候开发者一般会被告知要避免它。 

# 说重点  AFN 源码学习
## AFNURLSessionManager 类中持有nsprogress uploadProgress 和 downloadProgress 
在该类中有setupProgressForTask:(NSURLSessionTask*)task 方法中 对task进行了监听 监听 
```
[task addObserver:self
           forKeyPath:NSStringFromSelector(@selector(countOfBytesReceived))
              options:NSKeyValueObservingOptionNew
              context:NULL];
    [task addObserver:self
           forKeyPath:NSStringFromSelector(@selector(countOfBytesExpectedToReceive))
              options:NSKeyValueObservingOptionNew
              context:NULL];

    [task addObserver:self
           forKeyPath:NSStringFromSelector(@selector(countOfBytesSent))
              options:NSKeyValueObservingOptionNew
              context:NULL];
    [task addObserver:self
           forKeyPath:NSStringFromSelector(@selector(countOfBytesExpectedToSend))
              options:NSKeyValueObservingOptionNew
              context:NULL];

```

同时也对持有的nsprogress进行了监听
```

    [self.downloadProgress addObserver:self
                            forKeyPath:NSStringFromSelector(@selector(fractionCompleted))
                               options:NSKeyValueObservingOptionNew
                               context:NULL];
    [self.uploadProgress addObserver:self
                          forKeyPath:NSStringFromSelector(@selector(fractionCompleted))
                             options:NSKeyValueObservingOptionNew
                             context:NULL];

```
kvo监听回调中做了处理 很明显是针对于nsprogress 
```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context {
    if ([object isKindOfClass:[NSURLSessionTask class]] || [object isKindOfClass:[NSURLSessionDownloadTask class]]) {
        if ([keyPath isEqualToString:NSStringFromSelector(@selector(countOfBytesReceived))]) {
            self.downloadProgress.completedUnitCount = [change[NSKeyValueChangeNewKey] longLongValue];
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(countOfBytesExpectedToReceive))]) {
            self.downloadProgress.totalUnitCount = [change[NSKeyValueChangeNewKey] longLongValue];
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(countOfBytesSent))]) {
            self.uploadProgress.completedUnitCount = [change[NSKeyValueChangeNewKey] longLongValue];
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(countOfBytesExpectedToSend))]) {
            self.uploadProgress.totalUnitCount = [change[NSKeyValueChangeNewKey] longLongValue];
        }
    }
    else if ([object isEqual:self.downloadProgress]) {
        if (self.downloadProgressBlock) {
            self.downloadProgressBlock(object);
        }
    }
    else if ([object isEqual:self.uploadProgress]) {
        if (self.uploadProgressBlock) {
            self.uploadProgressBlock(object);
        }
    }
}
```
kvo 回调中处理NSURLSessionTask 以及 downloadTask实例 然后对应方法进行处理

## 这里就到了 NSURLSessionTaskDelegate 中 
- (void)URLSession:(__unused NSURLSession*)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error 

