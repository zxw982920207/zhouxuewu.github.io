---
title: 浅谈Java多线程
date: 2017-12-30
tags: Java
---

### 1.线程与进程

> **进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含多个线程。（进程是资源分配的最小单位）**

> **线程：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）**

### 2.线程的创建

#### 2.1 线程的两种创建方式

> a.继承java.lang.Thread类 

```java
package multiThread;

/**
 * Created by succe on 2018/1/14.
 */
public class Test01 {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
    }
}
class MyThread extends Thread{
    private static int num = 0;
    public MyThread(){
        num++;
    }
    @Override
    public void run() {
        System.out.println("主动创建的第"+num+"个线程");
    }
}
```

运行结果如下：

```java
主动创建的第1个线程

Process finished with exit code 0
```

创建好了自己的线程类之后，就可以创建线程对象了，然后通过**start()**方法去启动线程。注意，不是调用run()方法启动线程，run()方法中只是定义需要执行的任务，如果调用run方法， 即相当于在主线程中执行run方法，跟普通的方法调用没有任何区别，此时并不会创建一个新的线程来执行定义的任务。

start()方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（Runnable），什么时候运行是由操作系统决定的。从程序运行的结果可以发现，多线程程序是乱序执行。因此，只有乱序执行的代码才有必要设计为多线程。

***start()方法调用和 run()方法调用的区别：***

```java
package multiThread;

/**
 * Created by succe on 2018/1/14.
 */
public class Test03 {
    public static void main(String[] args) {
        System.out.println("主线程ID:"+Thread.currentThread().getId());
        MyThread03 thread1 = new MyThread03("thread1");
        thread1.start();
        MyThread03 thread2 = new MyThread03("thread2");
        thread2.run();
    }
}

class MyThread03 extends Thread {
    private String name;
    public MyThread03(String name){
        this.name = name;
    }
    @Override
    public void run() {
        System.out.println("name:"+name+" 子线程ID:"+Thread.currentThread().getId());
    }
}
```

运行结果如下：

```java
主线程ID:1
name:thread2 子线程ID:1
name:thread1 子线程ID:11

Process finished with exit code 0
```

**结论：**

***线程 Thread2()和主线程 ID相同，说明：通过 run()方法调用的线程不会创建新的线程，而是在主线程上直接运行 run()方法，和普通的方法没有区别***

> b.实现java.lang.Runable接口

```java
package multiThread;

/**
 * Created by succe on 2018/1/14.
 */
public class Test02 {
    public static void main(String[] args) {
        System.out.println("主线程ID："+Thread.currentThread().getId());
        MyRunable myRunable = new MyRunable();
        Thread thread = new Thread(myRunable);
        thread.start();
    }
}
class MyRunable implements Runnable {
    public void run() {
        System.out.println("子线程ID："+Thread.currentThread().getId());
    }
}
```

运行结果如下：

```java
主线程ID：1
子线程ID：11

Process finished with exit code 0
```

Runnable的中文意思是“任务”，顾名思义，通过实现Runnable接口，我们定义了一个子任务，然后将子任务交由Thread去执行。注意，这种方式必须将Runnable作为Thread类的参数，然后通过Thread的start方法 来创建一个新线程来执行该子任务。

事实上，查看Thread类的实现源代码会发现Thread类是实现了Runnable接口的。

在Java中，这2种方式都可以用来创建线程去执行子任务，具体选择哪一种方式要看自己的需求。直接继承Thread类的话，可能比实现Runnable接口看起来更加简洁，但是由于***Java只允许单继承，所以如果自定义类需 要继承其他类，则只能选择实现Runnable接口。***

#### 2.2 Thread与Runnable的区别

***实现Runable接口比继承Thread类所具有的优势：***

a) 适合多个相同程序代码的线程去处理同一个资源

b) 可以避免Java中单继承的限制

c) 代码可以被多个线程共享，而代码的数据独立

d) 线程池不接受继承Thread的线程类

> 注：main方法其实也是一个线程。在java中所以的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源。
>
> 在java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，每一个JVM实习在就是在操作系统中启动了一个进程。

### 3.线程对象常用函数

```java
1)public void start()
  使该线程开始执行；Java 虚拟机调用该线程的 run 方法
2)public void run()
  如果该线程是使用独立的 Runnable 运行对象构造的，则调用该 Runnable 对象的 run 方法；否则，
  该方法不执行任何操作并返回
3)public final void setName(String name)
  改变线程名称，使之与参数 name 相同
4)public final void setPriority(int priority)
  更改线程的优先级
5)public final void setDaemon(boolean on)
  将该线程标记为守护线程或用户线程
6)public final void join(long millisec)
  等待该线程终止的时间最长为 millis 毫秒
7)public void interrupt()
  中断线程
8)public final boolean isAlive()
  测试线程是否处于活动状态
```

#### 3.1 wait/notify/notifyAll

> 这是一组 Object 类的方法
>
> 注意：这三个方法都必须在同步的范围内调用

* wait

```java
wait有三种方式的调用
wait()
必要要由 notify 或者 notifyAll 来唤醒
wait(long timeout)
在指定时间内，如果没有notify或notifAll方法的唤醒，也会自动唤醒。
wait(long timeout,long nanos)
本质上还是调用一个参数的方法
public final void wait(long timeout, int nanos) throws InterruptedException {
      if (timeout < 0) {
             throw new IllegalArgumentException("timeout value is negative");
       }
      if (nanos < 0 || nanos > 999999) {
              throw new IllegalArgumentException(
             "nanosecond timeout value out of range");
       }
       if (nanos > 0) {
             timeout++;
       }
       wait(timeout);
}
```

* notify/notifyAll

> notify只能唤醒一个处于wait的线程
>
> notifyAll可以唤醒全部处于wait的线程

#### 3.2 sleep/yield/join

> 这是一组Thread类的方法

* sleep

  让当前线程暂停指定时间，只是让出CPU的使用权，并不释放锁。

* yield

  暂停当前线程的执行，也就是当前CPU的使用权，让其他线程有机会执行，不能指定时间。会让当前线程从运行状态转变为就绪状态。	

* join

  等待调用 join 方法的线程执行结束，才执行后面的代码，其调用一定要在 start 方法之后。
  使用场景：当父线程需要等待子线程执行结束才执行后面内容或者需要某个子线程的执行结果会用到 join 方法。

#### 3.3 volatile关键字

​	java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁更加方便。如果一个字段被声明成volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。

**原理：**

> 多线程的内存模型：main memory（主存）、working memory（线程栈），在处理数据时，线程会把值从主存load到本地栈，完成操作后再save回去(volatile关键词的作用：每次针对该变量的操作都激发一次load and save)。

**作用：**

> 内存可见性（多线程操作的时候，一个线程修改了一个变量的值 ，其他线程能立即看到修改后的值）
>
> 防止重排序（即程序的执行顺序按照代码的顺序执行，处理器为了提高代码的执行效率可能会对代码进行重排序）

#### 3.4 synchronized关键字

> 作用：确保线程互斥的访问同步代码

**使用：**

* synchronized单独使用

  * 代码块

    ```java
    public class Thread1 implements Runnable {
       Object lock;
       public void run() {  
           synchronized(lock){
             ..do something
           }
       }
    }
    ```

    同一时间，只有一个线程可以使用lock实例。

  * 直接用于方法

    ```java
    public class Thread1 implements Runnable {
       public synchronized void run() {  
            ..do something
       }
    }
    ```

    同一时间，只有一个线程可以调用run()方法。

* synchronized,wait,notify结合使用

```java
//synchronized, wait, notify结合:典型场景生产者消费者问题
/**
   * 生产者生产出来的产品交给店员
   */
  public synchronized void produce()
  {
      if(this.product >= MAX_PRODUCT)
      {
          try
          {
              wait();  
              System.out.println("产品已满,请稍候再生产");
          }
          catch(InterruptedException e)
          {
              e.printStackTrace();
          }
          return;
      } 
      this.product++;
      System.out.println("生产者生产第" + this.product + "个产品.");
      notifyAll();   //通知等待区的消费者可以取出产品了
  }
 
  /**
   * 消费者从店员取产品
   */
  public synchronized void consume()
  {
      if(this.product <= MIN_PRODUCT)
      {
          try
          {
              wait(); 
              System.out.println("缺货,稍候再取");
          } 
          catch (InterruptedException e) 
          {
              e.printStackTrace();
          }
          return;
      } 
      System.out.println("消费者取走了第" + this.product + "个产品.");
      this.product--;
      notifyAll();   //通知等待去的生产者可以生产产品了
  }

```

### 4.线程状态转换

> 线程的优先级：

```java
    /**
     * The minimum priority that a thread can have.
     */
    public final static int MIN_PRIORITY = 1;

   	/**
     * The default priority that is assigned to a thread.
     */
    public final static int NORM_PRIORITY = 5;

    /**
     * The maximum priority that a thread can have.
     */
    public final static int MAX_PRIORITY = 10;
```

每一个 Java 线程都有一个优先级，这样有助于操作系统确定线程的调度顺序。

Java 线程的优先级是一个整数，其取值范围是 1 （Thread.MIN_PRIORITY ） - 10 （Thread.MAX_PRIORITY ）。

默认情况下，每一个线程都会分配一个优先级 NORM_PRIORITY（5）。 具有较高优先级的线程对程序更重要，并且应该在低优先级的线程之前分配处理器资源。但是，线程优先级不能保证线程执行的顺序，而且非常依赖于平台。

1、Thread类的setPriority()和getPriority()方法分别用来设置和获取线程的优先级。

 每个线程都有默认的优先级。主线程的默认优先级为Thread.NORM_PRIORITY。

线程的优先级有继承关系，比如A线程中创建了B线程，那么B将和A具有相同的优先级。

JVM提供了10个线程优先级，但与常见的操作系统都不能很好的映射。

 2、线程睡眠：Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

3、线程等待：Object类中的wait()方法，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

4、线程让步：Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

5、线程加入：join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

6、线程唤醒：Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。

### 5.线程的生命周期

![线程的生命周期](http://pccmxww5q.bkt.clouddn.com/thread-life-cycle.jpg?imageView2/0/w/600/h/500/q/100)

线程在一个动态的生命周期中，会有不同的状态：

**创建（new）**: 使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 start() 这个线程。
**就绪（runnable）**: 当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。
**运行（running）**: 如果就绪状态的线程获取 CPU 资源，就可以执行 run()，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。
**阻塞（blocked）**: 如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：

- 等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。
- 同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。
- 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。

**终止（dead）**: 一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

### 6.遇到的问题和解决方法

* 遇到的问题

在创建线程的过程中，实现了一段代码：

```java
/**
 * Created by succe on 2018/1/14.
 */
public class Test01 {
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            MyThread01 thread01 = new MyThread01();
            thread01.start();
        }
    }
}
class MyThread01 extends Thread{
    private static int num = 0;

    public MyThread01(){
        num++;
    }

    @Override
    public void run() {
        System.out.println("主动创建的第"+num+"个线程");
    }
}

```

输出结果：

```java
主动创建的第100个线程
主动创建的第100个线程
主动创建的第100个线程
主动创建的第100个线程
主动创建的第100个线程
......

Process finished with exit code 0
```

> 为什么最后输出的全部都是100呢？
>
> 期望输出的结果：1,2,3,4,5......
>
> 原因：main线程执行过快，main线程执行结束时，num累加到了 100，子线程还处于runnable就绪状态。

* 解决办法

**第一种解决方法：调用Thread.sleep()方法。**

```java
/**
 * Created by succe on 2018/1/14.
 */
public class Test01 {
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            MyThread01 thread01 = new MyThread01();
            thread01.start();
            try {
                Thread.sleep(1000);//解决
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
class MyThread01 extends Thread{
    private static int num = 0;

    public MyThread01(){
        num++;
    }

    @Override
    public void run() {
        System.out.println("主动创建的第"+num+"个线程");
    }
}

```

Thread.sleep()方法可以让main线程暂定执行一小段时间。

**第二种解决方法：调用子线程的join()方法。**

```java
/**
 * Created by succe on 2018/1/14.
 */
public class Test01 {
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            MyThread01 thread01 = new MyThread01();
            thread01.start();
            try {
                thread01.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
class MyThread01 extends Thread{
    private static int num = 0;

    public MyThread01(){
        num++;
    }

    @Override
    public void run() {
        System.out.println("主动创建的第"+num+"个线程");
    }
}

```

执行结果：

```java
主动创建的第1个线程
主动创建的第2个线程
主动创建的第3个线程
主动创建的第4个线程
主动创建的第5个线程
主动创建的第6个线程
主动创建的第7个线程
...
...
主动创建的第99个线程
主动创建的第100个线程

Process finished with exit code 0
```

调用子线程的join()方法，可以让main线程等待子线程的执行，然后再继续执行子线程。



### 7.总结

* 线程的创建方式推荐实现Runnable接口
* 本质上，volatile就是不去缓存，直接取值。在线程安全的情况下加volatile会牺牲性能。
* 继承Thread类时，最好要设置线程名称 Thread.name，并设置线程组 ThreadGroup，目的是方便管理。在出现问题的时候，打印线程栈 (jstack -pid) 一眼就可以看出是哪个线程出现了问题。



参考：[Java多线程学习（吐血超详细总结）](http://blog.csdn.net/evankaka/article/details/44153709)      
[Java核心技术---多线程（THREAD）](http://movesan.me/2017/02/16/java-thread/)    
[Java并发编程，你需要知道的](https://www.jianshu.com/p/01188fa8e511)  


