---
title: Java多线程
date: 2019-8-14 11:16:23
tags: Java多线程
---

# Java多线程

## Java创建线程的三种方式

- 继承Thread类
- 实现Runnable接口
- 实现Callable和Future接口

### 优缺点对比

|          | 继承Thread                   | 实现Runnable                           | 实现Callable                   |
| -------- | ---------------------------- | -------------------------------------- | ------------------------------ |
| 优点     | 编程简单，执行效率高         | 面向接口编程，执行效率高               | 容器管理线程，允许返回值与异常 |
| 缺点     | 单继承，无法对线程组有效控制 | 无法对线程组有效控制，没有返回值、异常 | 执行效率相对较低，编程麻烦     |
| 使用场景 | 不推荐使用                   | 简单的多线程程序                       | 企业级应用推荐使用             |

## Synchronized多线程同步机制

synchronized关键字的作用就是利用一个特定的对象设置一个锁，在多线程并发访问的时候，同时只允许一个线程获得这个锁，执行特定的代码，执行后释放锁，继续由其它线程争抢。

### Synchronized使用场景

- synchronized代码块：任意对象即可

  ```java
  synchronized (new Object()) {
      ...
  }
  ```

- synchronized方法：this当前对象

  ```java
  public synchronized static void foo() {...}
  或者
  synchronized (this) {
  	...
  }
  ```

- synchronized静态方法：该类的字节码对象

  ```java
  synchronized (V.class) {
  	...
  }
  ```

  该方法适用于静态方法内

## 线程的五种状态

- 新建（new）
- 就绪（ready）
- 运行中（running）
- 阻塞（blocked）
- 死亡（dead）

# JDK并发工具包

## 线程池

- 重用存在的线程，减少对象创建、销毁的开销
- 线程总数可控，提高资源利用率
- 避免过多资源竞争，避免阻塞
- 提供额外功能，定时执行、定期执行、监控等

### 线程池的种类

在`java.util.concurrent`包中，提供了工具类`Executors`调度器来创建线程池，可创建的线程池有四种

- CachedThreadPool - 可缓存线程池
  - 线程池无限大，如果有空闲线程则调用，没有则创建新的线程
- FixedThreadPool - 定长线程池
  - 线程总数固定，如果没有空闲线程则等待
  - 如果任务处于等待状态，备选的等待算法为FIFO（先进先出，默认算法）和LIFO（后进先出）
- SingleThreadExecutor - 单线程池
  - 创建一个可管理的单线程
- ScheduledThreadPool - 调度线程池
  - 支持按照时间控制调度线程池

## java.util.concurrent包

### CountDownLatch倒计时锁

使用CountDownLatch明确任务完成状态，已达到阻塞的目的

```java
private static int count = 0;

public static void main(String[] args) {
    ExecutorService threadPool = Executors.newFixedThreadPool(100);
    CountDownLatch cdl = new CountDownLatch(10000);
    for (int i = 0; i <= 10000; i++) {
        final int index = i;
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                synchronized (CountDownSample.class) {
                    try {
                        count += index;
                    } finally {
                        cdl.countDown();
                    }
                }
            }
        });
    }

    try {
        cdl.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(count);
    threadPool.shutdown();
}
```

### Semaphore信号量

控制当前同时访问的总数

```java
public static void main(String[] args) {
    ExecutorService threadPool = Executors.newCachedThreadPool();
    Semaphore semaphore = new Semaphore(5);

    for (int i = 1; i <= 20; i++) {
        final int index = i;
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    if (semaphore.tryAcquire(5, TimeUnit.SECONDS)) {
                        ...
                        semaphore.release();
                    } else {
                        ...
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    threadPool.shutdown();
}
```

### CyclicBarrier循环屏障

CyclicBarrier是一个同步工具类，允许一组线程互相等待，直到到达某个公共屏障点。与CountDownLatch不同的时候CyclicBarrier在释放等待线程后可以重用。

```java
private static CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
public static void main(String[] args) {
    ExecutorService threadPool = Executors.newCachedThreadPool();
    for (int i = 0; i < 20; i++) {
        final int index = i;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        });
    }
    threadPool.shutdown();
}
```

### ReentrantLock重入锁

重入锁指任意线程在获取到锁之后，再次获得该锁时不会被该锁阻塞

#### ReentrantLock与synchronized区别

| 特征     | synchronized           | ReentrantLock                                       |
| -------- | ---------------------- | --------------------------------------------------- |
| 底层原理 | JVM实现                | JDK实现                                             |
| 性能区别 | JDK1.5之前低，之后高   | 高                                                  |
| 锁的释放 | 自动释放（编译器保证） | 手动释放（finally保证）                             |
| 编码程度 | 简单                   | 复杂                                                |
| 锁的粒度 | 读写不区分             | 读锁、写锁                                          |
| 高级功能 | 无                     | 公平锁、非公平锁唤醒，Condition分组唤醒，中断等待锁 |

### Condition等待与唤醒

- 必须在ReentrantLock重入锁中使用
- 用于替代wait()/notify()方法，可以唤醒指定的线程

#### 核心方法

- await() - 阻塞当前线程，直到signal唤醒
- signal() - 唤醒被await的线程，从中断处继续执行
- signalAll() - 唤醒所有被await()阻塞的线程

```java
public static void main(String[] args) {
    ReentrantLock reentrantLock = new ReentrantLock();
    Condition c1 = reentrantLock.newCondition();
    Condition c2 = reentrantLock.newCondition();
    Condition c3 = reentrantLock.newCondition();

    new Thread(new Runnable() {
        @Override
        public void run() {
            reentrantLock.lock();
            try {
                c1.await();
                Thread.sleep(1000);
                System.out.println("锄禾日当午");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }
    }).start();

    new Thread(new Runnable() {
        @Override
        public void run() {
            reentrantLock.lock();
            try {
                c2.await();
                Thread.sleep(1000);
                System.out.println("汗滴禾下土");
                c1.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }
    }).start();

    new Thread(new Runnable() {
        @Override
        public void run() {
            reentrantLock.lock();
            try {
                c3.await();
                Thread.sleep(1000);
                System.out.println("谁知盘中餐");
                c2.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }
    }).start();

    new Thread(new Runnable() {
        @Override
        public void run() {
            reentrantLock.lock();
            try {
                Thread.sleep(1000);
                System.out.println("粒粒皆辛苦");
                c3.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }
    }).start();
}
```

### Callable & Future

- Callable和Runnable一样代表着任务，区别在于Callable有返回值且可以抛出异常
- Future是一个接口，它用于表示异步计算的结果。提供了检查计算结果是否完成的方法，以等待计算的完成，并获取计算的结果

```java
public class FutureSample {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 2; i < 10000; i++) {
            Computer c = new Computer();
            c.setNum(i);
            Future<Boolean> result = executorService.submit(c);

            try {
                Boolean flag = result.get();
                if (flag) {
                    System.out.println(c.getNum());
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
        executorService.shutdown();
    }
}

class Computer implements Callable<Boolean> {
    private Integer num;

    public Integer getNum() {
        return num;
    }

    public void setNum(Integer num) {
        this.num = num;
    }

    @Override
    public Boolean call() throws Exception {
        boolean isPrime = true;
        for (int i = 2; i < num; i++) {
            if (num % i == 0) {
                isPrime = false;
                break;
            }
        }
        return isPrime;
    }
}
```

### 并发容器

- CopyOnWriteArrayList
- CopyOnWriteArraySet
- ConcurrentHashMap

### Atomic包

Atomic包是一个专门为线程安全设计的包，包含多个原子操作类。底层采用CAS算法。



