
## CFRunloop源码
```
`int32_t __CFRunLoopRun()
{
//通知即将进入runloop
__CFRunLoopDoObservers(KCFRunLoopEntry);

do
{
// 通知将要处理timer和source
__CFRunLoopDoObservers(kCFRunLoopBeforeTimers);
__CFRunLoopDoObservers(kCFRunLoopBeforeSources);

__CFRunLoopDoBlocks();  //处理非延迟的主线程调用
__CFRunLoopDoSource0(); //处理UIEvent事件

//GCD dispatch main queue
CheckIfExistMessagesInMainDispatchQueue();

// 即将进入休眠
__CFRunLoopDoObservers(kCFRunLoopBeforeWaiting);

// 等待内核mach_msg事件
mach_port_t wakeUpPort = SleepAndWaitForWakingUpPorts();

// Zzz...

// 从等待中醒来
__CFRunLoopDoObservers(kCFRunLoopAfterWaiting);

// 处理因timer的唤醒
if (wakeUpPort == timerPort)
__CFRunLoopDoTimers();

// 处理异步方法唤醒,如dispatch_async
else if (wakeUpPort == mainDispatchQueuePort)
__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__()

// UI刷新,动画显示
else
__CFRunLoopDoSource1();

// 再次确保是否有同步的方法需要调用
__CFRunLoopDoBlocks();

} while (!stop && !timeout);

//通知即将退出runloop
__CFRunLoopDoObservers(CFRunLoopExit);
}
```
`NSRunLoop调用方法主要就是在kCFRunLoopBeforeSources和kCFRunLoopBeforeWaiting之间,还有kCFRunLoopAfterWaiting之后,也就是如果我们发现这两个时间内耗时太长,那么就可以判定出此时主线程卡顿
