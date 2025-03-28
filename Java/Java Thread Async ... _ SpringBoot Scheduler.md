---
title: Java Thread Async ... / SpringBoot Scheduler
tags: [java, springboot]

---

# Java Thread Async ... / SpringBoot Scheduler

## Java Thread Tool Kit

### Callable / Future / FutureTask
```java=
public class CallableAndFuture {
    static class Task implements Callable<String> {
        @Override
        public String call() throws Exception {
            return "Hello world";
        }
    }

    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newSingleThreadExecutor();
        Future<String> future = threadPool.submit(new Task());
    System.out.println(future.get());
        
        Task task = new Task();
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        executor.submit(futureTask);
        System.out.println("task运行结果"+futureTask.get());
    }
}
```
> FutureTask 可以call cancel

### CountDownLatch
```java=
    ExecutorService es = Executors.newFixedThreadPool(5);
    try {
      CountDownLatch cdt = new CountDownLatch(5);

      for (int i = 0; i < 5; i++) {
        es.execute(()->{
            cdt.countDown();
        });
      }
      cdt.await();
      System.out.println("thread running finish");
    } catch(Exception e) {
      throw new RuntimeException(e);
    } finally {
      es.shutdown();
    }
```

## SpringBoot Scheduler

### Runtime

```java=

	@Inject
	private TaskScheduler scheduler;
...
    // Runtime start scheduler
    ScheduledFuture scheduledFuture = s.schedule(this, new CronTrigger("0 0/5 * * * *"));
    //取消 Runtime scheduler
    scheduledFuture.cancel(true);

```

###### tags: `java` `springboot`