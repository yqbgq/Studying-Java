> Java提供了一些便捷的工具方法，通过这些工具方法可以控制线程的执行

## join()方法

Thread提供了一个能够让一个线程等待另一个线程完成的方法——join()方法。当在某个程序执行流A中调用其他线程B的join()方法时，A线程将被阻塞，知道B线程执行完成为止。

join()方法通常由使用线程的程序来调用，用以将大问题划分成多个小问题，每个小问题分配一个线程。当小问题都得到处理之后，再调用主线程来进一步操作。

考虑下面这个示例：
```java
public class JoinThread extends Thread {
    public JoinThread(String name){
        super(name);
    }

    public void run(){
        for(int i=0;i<5;i++){
            System.out.println(getName()+"   "+i);
        }
    }

    public static void main(String[] args) throws Exception{
        JoinThread jt = new JoinThread("被join的线程");
        jt.start();
        jt.join();
        //调用join()方法之后，主线程会被阻塞直到jt线程结束为止
        //否则因为主线程和jt线程是并发执行，所以下面的输出不会在jt线程结束之后才输出
        System.out.println(Thread.currentThread().getName());
    }
}
```

执行结果如图所示：

![使用join控制线程执行](https://www.amoshuang.com/wp-content/uploads/2018/10/Threadresult.png)

## 后台线程
Daemon Thread线程是一种专门在后台运行的，为其他线程提供服务，它被称为“后台线程”或者“守护线程”。JVM的垃圾回收线程就是典型的后台线程。

后台线程有一个重要的特征：**当所有的前台线程都死亡后，后台线程自动结束。**

调用Thread对象的setDaemon(true)方法即可将指定的线程设置为后台线程。

考虑下面这个示例：
```java
public class DaemonThread extends Thread {
    public void run(){
        for(int i=0;i<1000;i++){
            System.out.println("----后台进程----"+i);
        }
    }

    public static void main(String[] args){
        DaemonThread dt = new DaemonThread();
        //将线程设为后台线程
        dt.setDaemon(true);
        dt.start();
        System.out.println("----主线程----");
        //程序执行到此处，前台线程结束
        //因为没有前台线程了，所以后台线程也随之结束
    }
}
```

执行结果如图所示：

![使用后台线程控制线程执行
](https://www.amoshuang.com/wp-content/uploads/2018/10/使用后台线程控制线程执行.png)

需要注意的是：

1. 前台线程死亡后，JVM会通知后台线程死亡，但从它接收指令到作出相应，需要一定的时间。
2. 如果需要将某个线程设置为后台线程，则必须在该线程启动之前进行设置，否则会引发IllegalThreadStateException异常。

## 使用sleep()方法进行线程睡眠

如果需要让当前正在执行的线程暂停一段时间，并进入阻塞状态，则可以通过调用Thread类的静态sleep()方法来实现。

sleep()方法有两种重载形式：

1. **sleep(long millis)** 让当前正在执行的线程暂停millis毫秒，并进入阻塞状态，该方法收到系统计时器和线程调度器的精度和准确度的影响。
2. **sleep(long millis, int nanos)** 让当前正在执行的线程暂停没劳力士毫秒加nanos毫微秒，并进入阻塞状态，该方法收到系统计时器和线程调度器的精度和准确度的影响。

需要注意的是，程序很少使用第二种重载形式，因为程序、计算机硬件和操作系统本身无法精确到毫微秒。

当线程调用了sleep()方法进入阻塞状态后，在其睡眠时间段内，该线程不会获得执行的机会，及时系统中没有其他可以执行的线程，处于睡眠中的线程也不会执行，因此sleep()方法常用来暂停程序的执行。

## 改变线程的优先级

每个线程执行时都有一定的优先级，优先级高的程序将获得较多的执行机会，而优先级较低的线程则获得较少的执行机会。

每个线程的默认优先级都和创建它们的父线程的优先级相同，在默认情况下，main线程具有普通优先级，由main线程创建的线程就只具有普通优先级。

Thread类提供了setPriority(int newPriority)和getPriority()方法来设置和返回指定线程的优先级。

Java提供了三个优先级静态常量：
1. MAX_PRIORITY：其值为10
2. MIN_PRIORITY：其值为1
3. NORM_PRIORITY：其值为5

需要注意的是：虽然Java提供了10个优先级级别，但是这些优先级界别需要操作系统进行支持，但不同操作系统上的优先级并不同，而且也不能很好的和Java的10个优先级相对应。

因此我们应该尽量避免直接为线程指定优先级，而应该使用三个静态常量来设置优先级，这样才能保证程序具有良好的可移植性。