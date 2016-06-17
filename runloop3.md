#i've seen a article by foreigner whose is cococa samyrai [http://cocoasamurai.blogspot.jp/search/label/NSRunLoop]

## osspinlock and pthread_mutex_t nearly similarity 

## **this article was written when 2011 but now is 2016 haha

* first introduced a code about cw_each by block 
```
/**
 Ruby inspired iterator for NSArray in Objective-C
 */
-(NSArray *)cw_each:(void (^)(id obj))block
{
    for(id object in self){
        block(object);
    }
 
    return self;
}
```
* the author written a dispatch_group_t block to find concurently task 异步并行执行遍历数组执行任务  
```

// only when all task complete it will return with dispatch_group_wait
-(NSArray *)cw_eachConcurrentlyWithBlock:(void (^)(id obj))block
{
    dispatch_group_t group = dispatch_group_create();
 
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
 
    for(id object in self){
  
        dispatch_group_async(group, queue, ^{
            block(object);
        });
    }
 
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    dispatch_release(group);//arc 下不需要release 
 
    return self;
}
```
## intro to threading 
![image](http://lh6.ggpht.com/colindw/R_-8utbctsI/AAAAAAAAAaQ/wwga7D1mZrM/thread%20memory%20layout.png?imgmax=800)

## why & when you should & shouldn't use multithread
* file/io/networking operations
* api's that explicitly state they will lock the calling thread and create their own thread
* any coompartmental / modular task that's at least 10ms

* __ do not refresh main ui in other thread __

## here i will tell you what it will cost when you create a thread
* Kernel Data Structures	1 KB // 内核数据结构
* Stack Space	512 KB 2nd*/8MB Main // 栈空间
* Creation Time	90 microseconds。    // 创建需要时间。微秒
* Mutex Acquisition Time	0.2 microseconds // 
* Atomic Compare & Swap	0.05 microseconds


