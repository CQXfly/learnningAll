#first learn runloop 
## 在主线程中添加代码

`
while(1){
NSRunLoop * runLoop = [NsRunLoop currentRunLoop];
[runloop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
}
`

-------------------------
h1
zh
