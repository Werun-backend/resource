## Java 并发编程

## 基本概念

### 并发和并行

+ 并行：在同一时刻，有多个指令在多个CPU上同时执行。

+ 并发：在同一时刻，有多个指令在单个CPU上交替执行。

### 进程和线程

+ 进程：是正在运行的程序

  - 独立性：进程是一个能独立运行的基本单位，同时也是系统分配资源和调度的独立单位
  - 动态性：进程的实质是程序的一次执行过程，进程是动态产生，动态消亡的
  - 并发性：任何进程都可以同其他进程一起并发执行

+ 线程：是进程中的单个顺序控制流，是一条执行路径

  - 单线程：一个进程如果只有一条执行路径，则称为单线程程序

  - 多线程：一个进程如果有多条执行路径，则称为多线程程序


## 实现多线程方式一：继承 Thread 类

- 方法介绍

  | 方法名        | 说明                         |
  | ------------ | -------------------------- |
  | void run()   | 在线程开启后，此方法将被调用执行           |
  | void start() | 使此线程开始执行，Java虚拟机会调用run方法() |

- 实现步骤
  - 定义一个类 MyThread 继承 Thread 类
  - 在 MyThread 类中重写 run() 方法
  - 创建 MyThread 类的对象
  - 启动线程

- 代码

  ```java
  // MyThread.java
  public class MyThread extends Thread {
      @Override
      public void run() {
          for (int i = 0; i < 100; i++) {
              System.out.println(i);
          }
      }
  }

  // Main.java
  public class Main {
      public static void main(String[] args) {
          MyThread my1 = new MyThread();
          MyThread my2 = new MyThread();
          my1.start();
          my2.start();
      }
  }
  ```

  ## 实现多线程方式二：实现 Runnable 接口

  - Thread构造方法

  | 方法名                                  | 说明             |
  | ------------------------------------ | -------------- |
  | Thread(Runnable target)              | 分配一个新的 Thread 对象 |
  | Thread(Runnable target, String name) | 分配一个新的 Thread 对象 |

  - 实现步骤

    - 定义一个类例如 MyRunnable 实现Runnable接口
    - 在 MyRunnable 类中重写 run() 方法
    - 创建 MyRunnable 类的对象
    - 创建 Thread 类的对象，把 MyRunnable 对象作为构造方法的参数
    - 启动线程

  - 代码

  ```java
  // MyRunnable.java
  public class MyRunnable implements Runnable {
      @Override
      public void run() {
          for (int i = 0; i < 100; i++) {
              System.out.println(Thread.currentThread().getName() + ": " + i);
          }
      }
  }

  // Main.java
  public class Main {
      public static void main(String[] args) {
          MyRunnable task = new MyRunnable();
          Thread t1 = new Thread(task, "坦克");
          Thread t2 = new Thread(task, "飞机");

          //启动线程
          t1.start();
          t2.start();
      }
  }
  ```

## 实现多线程方式三: 实现 Callable 接口

+ 方法介绍

  | 方法名                              | 说明                                  |
  | -------------------------------- | ----------------------------------- |
  | V call()                         | 计算结果，如果无法计算结果，则抛出一个异常               |
  | FutureTask(Callable<V> callable) | 创建一个 FutureTask，一旦运行就执行给定的 Callable |
  | V get()                          | 如有必要，等待计算完成，然后获取其结果                 |

+ 实现步骤

  + 定义一个类 MyCallable 实现 Callable 接口
  + 在 MyCallable 类中重写 call() 方法
  + 创建 MyCallable 类的对象
  + 创建 Future 的实现类 FutureTask 对象，把 MyCallable 对象作为构造方法的参数
  + 创建 Thread 类的对象，把 FutureTask 对象作为构造方法的参数
  + 启动线程
  + 再调用 get 方法，就可以获取线程结束之后的结果。

+ 代码

  ```java
  // MyCallable.java
  public class MyCallable implements Callable<String> {
      @Override
      public String call() throws Exception {
          for (int i = 0; i < 100; i++) {
              System.out.println("跟女孩表白" + i);
          }
          // 返回值表示线程运行完毕之后的结果
          return "答应";
      }
  }

  // Main.java
  public class Main {
      public static void main(String[] args) throws ExecutionException, InterruptedException {
          MyCallable mc = new MyCallable();

          // 可以获取线程执行完毕之后的结果, 作为参数传递给Thread对象
          FutureTask<String> ft = new FutureTask<>(mc);

          // 创建线程对象
          Thread t1 = new Thread(ft);

          String s = ft.get();
          // 开启线程
          t1.start();

          // String s = ft.get();
          System.out.println(s);
      }
  }
  ```

## 线程休眠

+ 相关方法

  | 方法名                            | 说明                       |
  | ------------------------------ | ------------------------ |
  | static void sleep(long millis) | 使当前正在执行的线程停留（暂停执行）指定的毫秒数 |

+ 代码演示

  ```java
  // 程序在这里会暂停一秒
  Thread.sleep(1000);
  ```

## 线程优先级

- 线程调度

  - 两种调度方式
    - 分时调度模型：所有线程轮流使用 CPU 的使用权，平均分配每个线程占用 CPU 的时间片
    - 抢占式调度模型：优先让优先级高的线程使用 CPU，如果线程的优先级相同，那么会随机选择一个，优先级高的线程获取的 CPU 时间片相对多一些

  - Java 使用的是抢占式调度模型

  - 随机性

    假如计算机只有一个 CPU，那么 CPU 在某一个时刻只能执行一条指令，线程只有得到 CPU 时间片，也就是使用权，才可以执行指令。所以说多线程程序的执行是有随机性，因为谁抢到CPU的使用权是不一定的

- 优先级相关方法

  | 方法名                                     | 说明                                |
  | --------------------------------------- | --------------------------------- |
  | final int getPriority()                 | 返回此线程的优先级                         |
  | final void setPriority(int newPriority) | 更改此线程的优先级线程默认优先级是 5；线程优先级的范围是：1-10 |

- 代码

  ```java
    MyThread t1 = new MyThread();
    // 设置最高优先级
    t1.setPriority(Thread.MAX_PRIORITY);
    t1.start();

    MyThread t2 = new MyThread();
    // 设置最低优先级
    t2.setPriority(Thread.MIN_PRIORITY);
    t2.start();

    // t1 将会更加频繁地执行
  ```

  ## 守护线程

  - 相关方法

    | 方法名                        | 说明                                   |
    | -------------------------- | ------------------------------------ |
    | void setDaemon(boolean on) | 将此线程标记为守护线程，当运行的线程都是守护线程时，Java虚拟机将退出 |

  ## 线程同步

  - 同步代码块格式：

    ```java
    synchronized (/* some object */) { 
      // 线程同步的代码块
    }
    ```

  - 示例

    ```java
      @Override
      public void run() {
        synchronized (this.data) {
          for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + this.data++);
          }
        }
      }
    ```

  - 同步的好处和弊端

    - 好处：解决了多线程的数据安全问题

    - 弊端：当线程很多时，因为每个线程都会去判断同步上的锁，这是很耗费资源的，无形中会降低程序的运行效率

  - 同步方法的格式

    同步方法：就是把 synchronized 关键字加到方法上

    ```java
    修饰符 synchronized 返回值类型 方法名(方法参数) { 
      方法体
    }
    ```

    同步方法的锁对象 this

  - 静态同步方法

    同步静态方法：就是把 synchronized 关键字加到静态方法上

    ```java
    修饰符 static synchronized 返回值类型 方法名(方法参数) { 
      方法体；
    }
    ```

    同步静态方法的锁对象是 类名.class

## Lock 锁
  为了更清晰的表达如何加锁和释放锁，JDK5 以后提供了一个新的锁对象 Lock

  Lock是接口不能直接实例化，这里采用它的实现类ReentrantLock来实例化

  - ReentrantLock构造方法

    | 方法名             | 说明                   |
    | --------------- | -------------------- |
    | ReentrantLock() | 创建一个ReentrantLock的实例 |

  - 加锁解锁方法

    | 方法名           | 说明   |
    | ------------- | ---- |
    | void lock()   | 获得锁  |
    | void unlock() | 释放锁  |

## 死锁

+ 概述

  线程死锁是指由于两个或者多个线程互相持有对方所需要的资源，导致这些线程处于等待状态，无法前往执行

+ 什么情况下会产生死锁

  1. 资源有限
  2. 同步嵌套

+ 代码

  ```java
  public class Main {
      public static void main(String[] args) {
          Object objA = new Object();
          Object objB = new Object();

          new Thread(() -> {
              while (true) {
                  synchronized (objA) {
                      //线程一
                      synchronized (objB) {
                          System.out.println("小康同学正在走路");
                      }
                  }
              }
          })
          .start();

          new Thread(() -> {
              while (true) {
                  synchronized (objB) {
                      //线程二
                      synchronized (objA) {
                          System.out.println("小薇同学正在走路");
                      }
                  }
              }
          })
          .start();
      }
  }
  ```

## 线程池
  系统创建一个线程的成本是比较高的，因为它涉及到与操作系统交互，当程序中需要创建大量生存期很短暂的线程时，频繁的创建和销毁线程对系统的资源消耗有可能大于业务处理是对系	统资源的消耗，这样就有点"舍本逐末"了。针对这一种情况，为了提高性能，我们就可以采用线程池。线程池在启动的时，会创建大量空闲线程，当我们向线程池提交任务的时，线程池就会启动一个线程来执行该任务。等待任务执行完毕以后，线程并不会死亡，而是再次返回到线程池中称为空闲状态。等待下一次任务的执行。

## 默认线程池

- 概述 : JDK 对线程池进行了相关的实现

- 可以使用 Executors 中所提供的静态方法来创建线程池

  | 方法名                                        | 说明   |
  | -------------------------------------------- | ---- |
  |  static ExecutorService newCachedThreadPool() | 创建一个默认的线程池 |
  | static newFixedThreadPool(int nThreads) | 创建一个指定最多线程数量的线程池 |

 - 代码
  ```java
        ExecutorService executorService = Executors.newCachedThreadPool();

        executorService.submit(()->{
            System.out.println(Thread.currentThread().getName() + "正在执行");
        });

        // 如果等代 2 秒，上一个任务已经完成，那么下一个任务会继续使用上一个任务的线程
        // Thread.sleep(2000);

        executorService.submit(()->{
            System.out.println(Thread.currentThread().getName() + "正在执行");
        });

        executorService.shutdown();
  ```

## 自定义线程池
  # 线程池的参数
  ```java
  public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler)
  ```

  - corePoolSize: 线程池中的核心线程数量，这些线程将一直存活，直到线程池关闭
  - maximumPoolSize: 线程池中的最大线程数量，必须大于等于核心线程数量
  - keepAliveTime: 非核心线程的存活时间
  - unit: 时间单位
  - workQueue: 存放任务的阻塞队列
  - threadFactory: 线程工厂，用于创建线程
  - handler: 拒绝策略，当阻塞队列满了，并且线程池中的线程数已经达到最大线程数，此时如果继续提交任务，将会采取一种策略处理该任务
    1. AbortPolicy: 默认的拒绝策略，直接抛出异常
    2. DiscardPolicy: 不做任何处理，直接丢弃任务
    3. DiscardOldestPolicy: 丢弃阻塞队列中靠前的任务，然后重新尝试提交当前任务
    4. CallerRunsPolicy: 运行当前提交的任务，但是提交任务的线程会阻塞
