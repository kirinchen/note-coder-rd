---
title: Java Thread 工具

---

# Java Thread 工具

## FutureTask 

```java
import java.util.concurrent.Callable;

public class Callable1 implements Callable<String> {
  private String message;
  private long sleepTimeMillis;

  public Callable1(String message, long sleepTimeMillis) {
    this.message = message;
    this.sleepTimeMillis = sleepTimeMillis;
  }

  @Override
  public String call() throws Exception {
    Thread.sleep(sleepTimeMillis);
    return this.message;
  }
}
```

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;

public class Test {
  public static void main(String args[]) throws Exception {
    Callable1 callable1 = new Callable1("Message1", 2000L);
    Callable1 callable2 = new Callable1("Message2", 1000L);

    FutureTask<String> futureTask1 = new FutureTask<String>(callable1);
    FutureTask<String> futureTask2 = new FutureTask<String>(callable2);

    ExecutorService executor = Executors.newFixedThreadPool(2);
    executor.execute(futureTask1);
    executor.execute(futureTask2);

    System.out.println(futureTask1.get());
    System.out.println(futureTask2.get());
  }
}
```

## CountDownLatch 
```java
import java.util.concurrent.CountDownLatch;

public class CountDownThreadExample implements Runnable {
  private CountDownLatch countDownLatch;

  public CountDownThreadExample(CountDownLatch countDownLatch) {
    this.countDownLatch = countDownLatch;
  }

  @Override
  public void run() {
    try {
      String currentThread = Thread.currentThread().getName();
      System.out.println("current Thread:" + currentThread);
    } catch(Exception e) {
       throw new RuntimeException(e);
    } finally {
        this.countDownLatch.countDown();
    }
  }
    
  public static void main(String args[]) {
    ExecutorService es = Executors.newFixedThreadPool(5);
    try {
      CountDownLatch cdt = new CountDownLatch(5);

      for (int i = 0; i < 5; i++) {
        es.execute(new CountDownThreadExample(cdt));
      }
      cdt.await();
      System.out.println("thread running finish");
    } catch(Exception e) {
      throw new RuntimeException(e);
    } finally {
      es.shutdown();
    }
  }
    
}
```