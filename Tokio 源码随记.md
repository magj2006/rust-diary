## 核心结构体：
-- PollEvent<E>
>  功能：用驱动它的反应器（reactor）关联一个实现了`Read`和`Write` trait I/O资源
>  
>  内部使用`Registration`结构体，而`Registration`又封装了`ScheduledIo`， 它会生成了一个实现了`Future`的`Readiness`
>  所以只要`mio::Evented`的类型E被封装到这个结构体， 就可以用在Future模型中. 因此通过基础的I/O资源和反应器驱动的`Readiness`，它提供了`AsyncRead` `AsyncWrite`等的实现。
>  
>  如果基础的I/O类型是`Sync`的，调用者必须保证至多两个task（一个用来读，一个用来写）并发调用， 以免发生无法预期的行为。
>  
>  方法`poll_read_ready` 和 `poll_read_write`会返回`PollEvent`实例的当前readiness state.
>  
>  当尝试上面的操作，由于I/O没有就绪而失败时，必须调用`clear_read_ready` 或 `clear_write_ready`方法。

