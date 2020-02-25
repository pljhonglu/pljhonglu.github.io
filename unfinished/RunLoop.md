# RunLoop

## 结构定义

### RunLoop

CFRunLoop 负责管理 RunLoop Mode，结构如下

``` objc
typedef struct __CFRunLoop * CFRunLoopRef;
struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;			/* locked for accessing mode list */
    __CFPort _wakeUpPort;			// used for CFRunLoopWakeUp 
    Boolean _unused;
    volatile _per_run_data *_perRunData;              // reset for runs of the run loop
    pthread_t _pthread;
    uint32_t _winthread;
    CFMutableSetRef _commonModes; // 字符串，记录所有标记为common的mode
    CFMutableSetRef _commonModeItems; // 所有commonMode的item(source、timer、observer)
    CFRunLoopModeRef _currentMode;
    CFMutableSetRef _modes; // CFRunLoopModeRef set
    struct _block_item *_blocks_head;
    struct _block_item *_blocks_tail;
    CFTypeRef _counterpart;
};
```

可见 CFRunLoop 里面包含了线程，若干个 mode。因此 runloop 和线程是对应的。

### RunLoop Mode

Mode 负责管理各种事件，结构如下：

``` objc
// 定义 CFRunLoopModeRef 为指向 __CFRunLoopMode 结构体的指针
typedef struct __CFRunLoopMode *CFRunLoopModeRef;

struct __CFRunLoopMode {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;	/* must have the run loop locked before locking this */
    CFStringRef _name;
    Boolean _stopped;
    char _padding[3];
    CFMutableSetRef _sources0; // source0 set ，非基于Port的，接收点击事件，触摸事件等APP 内部事件
    CFMutableSetRef _sources1; // source1 set，基于Port的，通过内核和其他线程通信，接收，分发系统事件
    CFMutableArrayRef _observers; // observer 数组
    CFMutableArrayRef _timers; // timer 数组
    CFMutableDictionaryRef _portToV1SourceMap;// source1 对应的端口号
    __CFPortSet _portSet;
    CFIndex _observerMask;
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    dispatch_source_t _timerSource;
    dispatch_queue_t _queue;
    Boolean _timerFired; // set to true by the source when a timer has fired
    Boolean _dispatchTimerArmed;
#endif
#if USE_MK_TIMER_TOO
    mach_port_t _timerPort;
    Boolean _mkTimerArmed;
#endif
#if DEPLOYMENT_TARGET_WINDOWS
    DWORD _msgQMask;
    void (*_msgPump)(void);
#endif
    uint64_t _timerSoftDeadline; /* TSR */
    uint64_t _timerHardDeadline; /* TSR */
};
```

可见 runloop mode 里面包含了source0、source1、timer、observer 各种事件，以及 port 集合。其负责管理这些事件。

Mode   |   Name   |    Description
-------|----------|------------------
Default| NSDefaultRunLoopMode kCFRunLoopDefaultMode | 用的最多的模式，大多数情况下应该使用该模式开始 RunLoop并配置 input source
Connection| NSConnectionReplyMode | Cocoa用这个模式结合 NSConnection 对象监测回应，你应该很少用这个模式
Modal| NSModalPanelRunLoopMode | Cocoa用此模式来标识用于模态面板的事件
Event tracking| NSEventTrackingRunLoopMode| Cocoa使用此模式在鼠标拖动loop和其它用户界面跟踪 loop期间限制传入事件
Common modes| NSRunLoopCommonModes kCFRunLoopCommonModes | 这是一组可配置的常用模式。将输入源与些模式相关联会与组中的每个模式相关联。默认情况下此集包括Default、Modal和Event tracking。Core Foundation只包括默认模式，你可以自己把自定义mode用CFRunLoopAddCommonMode函数加入到集合中.

### RunLoop source

#### CFRunLoopSource

RunLoop source 分为 source0 和 source1 两种。

source0 是非基于 port 的事件，主要是 APP 内部事件，如点击事件，触摸事件等。

source1 是基于Port的，通过内核和其他线程通信，接收，分发系统事件。

``` objc
typedef struct {
    CFIndex	version;
    void *	info;
    const void *(*retain)(const void *info);
    void	(*release)(const void *info);
    CFStringRef	(*copyDescription)(const void *info);
    Boolean	(*equal)(const void *info1, const void *info2);
    CFHashCode	(*hash)(const void *info);
    void	(*schedule)(void *info, CFRunLoopRef rl, CFStringRef mode);
    void	(*cancel)(void *info, CFRunLoopRef rl, CFStringRef mode);
    void	(*perform)(void *info);
} CFRunLoopSourceContext;

typedef struct {
    CFIndex	version;
    void *	info;
    const void *(*retain)(const void *info);
    void	(*release)(const void *info);
    CFStringRef	(*copyDescription)(const void *info);
    Boolean	(*equal)(const void *info1, const void *info2);
    CFHashCode	(*hash)(const void *info);
#if (TARGET_OS_MAC && !(TARGET_OS_EMBEDDED || TARGET_OS_IPHONE)) || (TARGET_OS_EMBEDDED || TARGET_OS_IPHONE)
    mach_port_t	(*getPort)(void *info);
    void *	(*perform)(void *msg, CFIndex size, CFAllocatorRef allocator, void *info);
#else
    void *	(*getPort)(void *info);
    void	(*perform)(void *info);
#endif
} CFRunLoopSourceContext1;

struct __CFRunLoopSource {
    CFRuntimeBase _base;
    uint32_t _bits;//用于标记Signaled状态，source0只有在被标记为Signaled状态，才会被处理
    pthread_mutex_t _lock;
    CFIndex _order;			/* immutable */
    CFMutableBagRef _runLoops;
    union {
	CFRunLoopSourceContext version0;	/* immutable, except invalidation */
        CFRunLoopSourceContext1 version1;	/* immutable, except invalidation */
    } _context;
};
```

可以发现 __CFRunLoopSource 里面包含一个 _runLoops，也就意味着一个 __CFRunLoopSource 可以被添加到多个 runloop mode 中去。

#### CFRunLoopObserver

CFRunLoopObserver 是观察者，可以观察RunLoop的各种状态，并抛出回调。

``` objc
typedef struct {
    CFIndex	version;
    void *	info;
    const void *(*retain)(const void *info);
    void	(*release)(const void *info);
    CFStringRef	(*copyDescription)(const void *info);
} CFRunLoopObserverContext;

struct __CFRunLoopObserver {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;
    CFIndex _rlCount;
    CFOptionFlags _activities;		/* immutable */
    CFIndex _order;			/* immutable */
    CFRunLoopObserverCallBack _callout;	/* immutable */
    CFRunLoopObserverContext _context;	/* immutable, except invalidation */
};
```

__CFRunLoopObserver 里面只有一个 _runLoop，也就意味着 __CFRunLoopObserver 只能被加到一个 runloop mode 里面。

CFRunLoopObserver可以观察的状态有如下6种:

```objc
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0), //即将进入run loop
    kCFRunLoopBeforeTimers = (1UL << 1), //即将处理timer
    kCFRunLoopBeforeSources = (1UL << 2),//即将处理source
    kCFRunLoopBeforeWaiting = (1UL << 5),//即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),//被唤醒但是还没开始处理事件
    kCFRunLoopExit = (1UL << 7),//run loop已经退出
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```

#### CFRunLoopTimer

CFRunLoopTimer 是定时器，可以在设定的时间点抛出回调，它的结构如下：

``` objc
typedef struct {
    CFIndex	version;
    void *	info;
    const void *(*retain)(const void *info);
    void	(*release)(const void *info);
    CFStringRef	(*copyDescription)(const void *info);
} CFRunLoopTimerContext;

struct __CFRunLoopTimer {
    CFRuntimeBase _base;
    uint16_t _bits;
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;
    CFMutableSetRef _rlModes;
    CFAbsoluteTime _nextFireDate;
    CFTimeInterval _interval;		/* immutable */
    CFTimeInterval _tolerance;// 容差          /* mutable */
    uint64_t _fireTSR;			/* TSR units */
    CFIndex _order;			/* immutable */
    CFRunLoopTimerCallBack _callout;	/* immutable */
    CFRunLoopTimerContext _context;	/* immutable, except invalidation */
};
```
__CFRunLoopTimer 里面只有一个 _runLoop，也就意味着 __CFRunLoopTimer 也只能被加到一个 runloop mode 里面。

另外根据官方文档的描述，CFRunLoopTimer和NSTimer是toll-free bridged的，可以相互转换。

> CFRunLoopTimer is “toll-free bridged” with its Cocoa Foundation counterpart, NSTimer. This means that the Core Foundation type is interchangeable in function or method calls with the bridged Foundation object.

## 获取 RunLoop

``` objc
static CFMutableDictionaryRef __CFRunLoops = NULL;
static CFSpinLock_t loopsLock = CFSpinLockInit;

// should only be called by Foundation
// t==0 is a synonym for "main thread" that always works
// t=0 就是获取 main thread
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    if (pthread_equal(t, kNilPthreadT)) {
	    t = pthread_main_thread_np();
    }
    __CFSpinLock(&loopsLock);
    if (!__CFRunLoops) {
        // 如果 RunLoops 字典为空，就新建一个，并新建 mainLoop 放进去
        __CFSpinUnlock(&loopsLock);
	    CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
	    CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
	    CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
	    // 把 dict 写到 __CFRunLoops 里面，并释放 dict
	    if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
	        CFRelease(dict);
	    }
	    CFRelease(mainLoop);
        __CFSpinLock(&loopsLock);
    }
    // 获取传入线程的 runloop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFSpinUnlock(&loopsLock);
    if (!loop) {
        // 如果没有获取到传入线程 runloop，那么就新建一个，放到 __CFRunLoops 里面
	    CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFSpinLock(&loopsLock);
	    loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
	    if (!loop) {
	        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
	        loop = newLoop;
	    }
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFSpinUnlock(&loopsLock);
	    CFRelease(newLoop);
    }
    
    //如果传入线程就是当前线程，注册一个回调，当线程销毁时，销毁对应的RunLoop
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}

CFRunLoopRef CFRunLoopGetMain(void) {
    CHECK_FOR_FORK();
    static CFRunLoopRef __main = NULL; // no retain needed
    if (!__main) __main = _CFRunLoopGet0(pthread_main_thread_np()); // no CAS needed
    return __main;
}

CFRunLoopRef CFRunLoopGetCurrent(void) {
    CHECK_FOR_FORK();
    CFRunLoopRef rl = (CFRunLoopRef)_CFGetTSD(__CFTSDKeyRunLoop);
    if (rl) return rl;
    return _CFRunLoopGet0(pthread_self());
}
```

- 不允许直接创建RunLoop
- RunLoop 和线程是一一对应的
- RunLoop 和线程的一一对应关系保存一个全局 Dictionary 里
- 线程刚创建时并没有 RunLoop, 如果你不主动获取, 那它一直都不会有(main thread除外)
- 你只能在一个线程的内部获取其 RunLoop(main thread 除外)
- RunLoop 会在线程销毁时销毁

## 运行 RunLoop

实际上就是一个 do{}while() 循环，内部逻辑大概如下

1. 通知 observers 即将进入 run loop
2. 通知 observes 即将开始处理 timer source
3. 通知 observes 即将开始处理 source0 事件
4. 执行 source0 事件
5. 如果处于主线程有 dispatchPort 消息，跳转到第9步
6. 通知 observes 线程即将进入休眠
7. 内循环阻塞等待接收系统消息，包括：
    - 收到内核发送过来的消息 （source1消息）
    - 定时器事件需要执行
    - run loop 的超时时间到了
    - 手动唤醒 run loop
8. 通知 observes 线程被唤醒
9. 处理通过端口收到的消息：
    - 如果自定义的 timer 被 fire，那么执行该 timer 事件并重新开始循环，完成后跳转到第2步
    - 如果 input source 被 fire，则处理该事件
    - 如果 run loop 被手动唤醒，并且没有超时，完成后跳转到第2步
10. 通知 observes run loop 已经退出

代码：

- [CFRunLoopRunSpecific.c]()
- [CFRunLoopRun.c]()
- [CFRunLoopServiceMachPort.c]()

## 总结



## 参考

[深入理解RunLoop](http://blog.ibireme.com/2015/05/18/runloop/)
[RunLoop系列之源码分析](http://aaaboom.com/?p=34#wow24)
[RunLoop系列之要点提炼](http://aaaboom.com/?p=37)


