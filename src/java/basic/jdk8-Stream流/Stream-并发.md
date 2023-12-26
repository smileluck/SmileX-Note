[toc]

---

# 准备工作

1. 获取线程的方法

```java
private void getThreadName() {
    getThreadName(-1);
}

private void getThreadName(Integer ier) {
    log.info("ThreadName:{}，value:{}", Thread.currentThread().getName(), ier);
}
```

2. 完整测试代码

```java

@Slf4j
public class StreamParallelTest {


    @Test
    public void parallelTest() throws InterruptedException {
        IntStream range = IntStream.range(1, 10);
        range.parallel().forEach(item -> {
            getThreadName();
        });
        log.info("主线程参与协作了");

        log.info("sleep 5000 start");
        Thread.sleep(5000);
        log.info("sleep 5000 end");

        range = IntStream.range(1, 10);
        range.mapToObj(e -> CompletableFuture.runAsync(() -> {
            getThreadName(e);
        })).forEach(CompletableFuture::join);
        log.info("主线程阻塞了,但是线程一直同一个");

        log.info("sleep 5000 start");
        Thread.sleep(5000);
        log.info("sleep 5000 end");

        range = IntStream.range(1, 10);
        CompletableFuture[] completableFutures = range.mapToObj(e -> CompletableFuture.runAsync(() -> {
            getThreadName(e);
        })).toArray(CompletableFuture[]::new);
        CompletableFuture.allOf(completableFutures).join();
        log.info("主线程阻塞了");

        log.info("sleep 5000 start");
        Thread.sleep(5000);
        log.info("sleep 5000 end");


        ExecutorService executorService = Executors.newFixedThreadPool(10);
        range = IntStream.range(1, 10);
        completableFutures = range.mapToObj(e -> CompletableFuture.runAsync(() -> {
            getThreadName(e);
        }, executorService)).toArray(CompletableFuture[]::new);
        CompletableFuture.allOf(completableFutures).join();
        log.info("主线程阻塞了");

        log.info("sleep 5000 start");
        Thread.sleep(5000);
        log.info("sleep 5000 end");


    }

    private void getThreadName() {
        getThreadName(-1);
    }

    private void getThreadName(Integer ier) {
        log.info("ThreadName:{}，value:{}", Thread.currentThread().getName(), ier);
    }
}
```

# parallelStrem

使用 `parallel` 启动并发。

```java
IntStream range = IntStream.range(1, 10);
range.parallel().forEach(item -> {
    getThreadName();
});
log.info("主线程参与协作了");
```

输出结果：

```shell
18:05:36.140 [ForkJoinPool.commonPool-worker-1] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-1，value:-1
18:05:36.140 [ForkJoinPool.commonPool-worker-5] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-5，value:-1
18:05:36.146 [ForkJoinPool.commonPool-worker-1] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-1，value:-1
18:05:36.140 [ForkJoinPool.commonPool-worker-4] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-4，value:-1
18:05:36.140 [main] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:main，value:-1
18:05:36.140 [ForkJoinPool.commonPool-worker-2] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-2，value:-1
18:05:36.140 [ForkJoinPool.commonPool-worker-3] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-3，value:-1
18:05:36.140 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:-1
18:05:36.140 [ForkJoinPool.commonPool-worker-6] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-6，value:-1
18:05:36.147 [main] INFO top.zsmile.test.basic.stream.StreamParallelTest - 主线程参与协作了
```

**调用方是主线程，当使用 `parallel` 启动并发时，主线程会和 `ForkJoinPool.commonPool` 参与并发的执行工作，同步执行。**

# 结合CompletableFuture使用

这里区分两种情况

- 通过 `CompletableFuture.join` 启动方法执行
- 通过 `CompletableFuture.allOf(CompletableFuture[] future).join` 启动异步执行

```java
range = IntStream.range(1, 10);
range.mapToObj(e -> CompletableFuture.runAsync(() -> {
    getThreadName(e);
})).forEach(CompletableFuture::join);
log.info("主线程阻塞了,但是线程一直同一个");

log.info("sleep 5000 start");
Thread.sleep(5000);
log.info("sleep 5000 end");

range = IntStream.range(1, 10);
CompletableFuture[] completableFutures = range.mapToObj(e -> CompletableFuture.runAsync(() -> {
    getThreadName(e);
})).toArray(CompletableFuture[]::new);
CompletableFuture.allOf(completableFutures).join();
log.info("主线程阻塞了");
```

输出结果

```shell
18:05:41.154 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:1
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:2
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:3
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:4
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:5
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:6
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:7
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:8
18:05:41.155 [ForkJoinPool.commonPool-worker-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-7，value:9
18:05:41.155 [main] INFO top.zsmile.test.basic.stream.StreamParallelTest - 主线程等待了,但是执行线程一直同一个
18:05:41.155 [main] INFO top.zsmile.test.basic.stream.StreamParallelTest - sleep 5000 start
18:05:46.164 [main] INFO top.zsmile.test.basic.stream.StreamParallelTest - sleep 5000 end
18:05:46.166 [ForkJoinPool.commonPool-worker-2] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-2，value:3
18:05:46.166 [ForkJoinPool.commonPool-worker-3] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-3，value:1
18:05:46.166 [ForkJoinPool.commonPool-worker-4] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-4，value:2
18:05:46.166 [ForkJoinPool.commonPool-worker-2] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-2，value:6
18:05:46.166 [ForkJoinPool.commonPool-worker-3] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-3，value:7
18:05:46.166 [ForkJoinPool.commonPool-worker-1] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-1，value:5
18:05:46.166 [ForkJoinPool.commonPool-worker-4] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-4，value:8
18:05:46.166 [ForkJoinPool.commonPool-worker-2] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-2，value:9
18:05:46.166 [ForkJoinPool.commonPool-worker-5] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:ForkJoinPool.commonPool-worker-5，value:4
18:05:46.166 [main] INFO top.zsmile.test.basic.stream.StreamParallelTest - 主线程等待了
```

- 可以看出当我们使用的 `CompletableFuture.join` 时，都是由同一个线程进行处理。
- 使用 `CompletableFuture.allOf(CompletableFuture[] future).join`时，会实现真正的异步多线程执行。
- 这两种方法都会阻塞主线程，但主线程不参与协同处理任务。



# CompletableFuture + Executor

这里使用 `CompletableFuture.allOf(CompletableFuture[] future).join` 以及 固定线程池 `Executor.newFixedThreadPool(10)`

```java
ExecutorService executorService = Executors.newFixedThreadPool(10);
range = IntStream.range(1, 10);
completableFutures = range.mapToObj(e -> CompletableFuture.runAsync(() -> {
    getThreadName(e);
}, executorService)).toArray(CompletableFuture[]::new);
CompletableFuture.allOf(completableFutures).join();
log.info("主线程等待了");
```

输出结果

```shell

18:26:49.295 [pool-1-thread-2] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-2，value:2
18:26:49.295 [pool-1-thread-1] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-1，value:1
18:26:49.295 [pool-1-thread-3] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-3，value:3
18:26:49.295 [pool-1-thread-4] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-4，value:4
18:26:49.295 [pool-1-thread-5] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-5，value:5
18:26:49.295 [pool-1-thread-6] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-6，value:6
18:26:49.295 [pool-1-thread-7] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-7，value:7
18:26:49.295 [pool-1-thread-8] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-8，value:8
18:26:49.295 [pool-1-thread-9] INFO top.zsmile.test.basic.stream.StreamParallelTest - ThreadName:pool-1-thread-9，value:9
18:26:49.295 [main] INFO top.zsmile.test.basic.stream.StreamParallelTest - 主线程等待了
```

- 和 `CompletableFuture.allOf(CompletableFuture[] future).join`  一样，不会阻塞主线程，都需要等待线程池执行完返回才继续下一步。
- 区别在于使用了自定义的线程池。这样可以更好地自主管理线程池资源、任务调度。

# 总结

1. 如果不希望阻塞主线程，那么可以使用 `CompletableFuture` 和 `Executor` 的方式进行实现。
2. 如果是IO密集型，那么可以使用 `CompletableFuture` 和自定义线程池，提高线程异步执行。
3. 如果是CPU密集型，那么推荐使用`Stream.parallel` ，因为实现简单直观、效率也高，异步任务数和CPU核心数相同。