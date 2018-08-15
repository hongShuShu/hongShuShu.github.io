---
title: GCD面试题-串行、并发、全局队列分析
---
### 引言
早上在群里看到有人发了份面试题，顺道做了一下，发现关于GCD的部分自己做错了，因此也就有了这篇博客，重新再复习下相关知识。

## 原题
请写出以下代码的执行顺序：
{% codeblock lang:objc %}
- (void)test_progress {
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"A = %@",[NSThread currentThread]);
    });

    NSLog(@"B = %@",[NSThread currentThread]);

    dispatch_queue_t que_tmp = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);

    dispatch_sync(que_tmp, ^{
        NSLog(@"C = %@",[NSThread currentThread]);
    });

    dispatch_async(que_tmp, ^{
        NSLog(@"D = %@",[NSThread currentThread]);
    });

    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"E = %@",[NSThread currentThread]);
    });

    [self performSelector:@selector(method) withObject:nil afterDelay:0.0];
    NSLog(@"F = %@",[NSThread currentThread]);
}
- (void)method {
    NSLog(@"G = %@",[NSThread currentThread]);
}
{% endcodeblock %}

<!--more-->
在分析题前先回顾下GCD基础知识，[demo](https://github.com/hongShuShu/gcd)
由demo得出结论：
<font color=red>串行同步:在当前线程顺序执行</font>
<font color=red>串行异步:开新线程，在新线程顺序执行</font>
<font color=red>并发同步:不开线程，在当前线程顺序执行</font>
<font color=red>并发异步:开多个线程，无序执行</font>
<font color=red>主队列异步:不开线程，在当前线程顺序执行</font>
<font color=red>主队列同步:主队列和主线程互相等待，造成“死锁”</font>

接着我们再来分析面试题：
{% codeblock lang:objc %}
// 主队列异步   不开线程，顺序执行
dispatch_async(dispatch_get_main_queue(), ^{  // 添加到任务队列，所以肯定是在当前方法执行完毕后调用
    NSLog(@"A = %@",[NSThread currentThread]);
});

//  主线程执行
NSLog(@"B = %@",[NSThread currentThread]); // 第一

//  全局队列本质是一个并发队列    后台优先级
dispatch_queue_t que_tmp = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);

//  并发同步  不开，顺序执行
dispatch_sync(que_tmp, ^{ // 第二
    NSLog(@"C = %@",[NSThread currentThread]);
});

//  并发异步  开多个线程 无序执行
dispatch_async(que_tmp, ^{  // 异步执行
    NSLog(@"D = %@",[NSThread currentThread]);
});

//  主队列异步   不开线程，添加任务，顺序执行
dispatch_async(dispatch_get_main_queue(), ^{
    NSLog(@"E = %@",[NSThread currentThread]);   // E在A之后
});

// 当前调用此方法的函数执行完毕后,selector方法才会执行
// 因此G肯定在F之后
[self performSelector:@selector(method) withObject:nil afterDelay:0.0]; // G是最后一个
NSLog(@"F = %@",[NSThread currentThread]); // 第三
{% endcodeblock %}
所以不难得出运行结果：B  C   F   A   D   E   G

### 强行小结
同步和异步作用：能不能开线程
    同步在当前线程中执行任务，不开线程
    异步在新线程中执行任务，开线程
 并发和串行作用：任务的执行方式
    并发是多个任务同时执行
    串行是任务顺序执行，一个接一个

全局队列：本质是一个并发队列
主队列：专门负责调度主线程度的任务，永远不开新线程，任务只会在主线程顺序执行
主队列异步：先将任务放在主队列中，但是不是马上执行，等到主队列中的其它所有除我们使用代码添加到主队列的任务的任务都执行完毕之后才会执行添加的任务

### 关于performSelector
afterDelay方式是使用当前线程的Run Loop中根据afterDelay参数创建一个Timer定时器在一定时间后调用SEL，NO AfterDelay方式是直接调用SEL。
这个方法其实是增加了一个定时器，而这时aSelector被添加到了队列的最后面，所以要等当前调用此方法的函数执行完毕后，selector方法才会执行。

参考博客：
https://blog.csdn.net/FocusOnLovingFreedom/article/details/49888465
https://www.jianshu.com/p/672c0d4f435a
