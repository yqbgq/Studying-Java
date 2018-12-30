> 关键字synchronized 与wait()、notify()方法相互结合可以实现多线程中的等待与通知，但是如果需要实现有选择的通知，我们就需要使用Condition了

Condition是JDK5中出现的技术，我们可以在一个Lock对象中创建多个Condition实例，线程对象可以注册在指定的Condition中，从而实现有选择性的进行通知，在调度上更加具有灵活性。

## 传统方法的缺陷
对于synchronized而言，它相当于整个Lock对象中只有一个单一的Condition对象，所有的线程都注册在它一个对象身上。在使用notify()或者notifyAll()方法进行通知时，被通知的线程是由JVM进行随机选择的。

对于开发者而言，对于唤醒哪一类线程是没有选择权的。因为线程的挂起和唤醒往往需要花费大量的资源，因此这样的调度方式存在很大的效率问题。

## 一个简单的例子

考虑这样一个例子：

我们需要维护一个大小为15的队列，生产者将生产的信息加入其中，消费者则从中取出信息。当出现队列已满时我们只需要通知所有消费者，当队列为空时，我们需要通知所有的生产者。

一个简单的代码示例如下：

```java
import java.util.ArrayList;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class TestCondition {
    public static void main(String[] args){
        ReentrantLock lock = new ReentrantLock();
        Condition full = lock.newCondition();
        Condition empty = lock.newCondition();
        ArrayList<String> queue = new ArrayList<>();
        Get[] gets = new Get[5];
        Put[] puts = new Put[2];
        for(int i=0;i<5;i++){
            Get get = new Get(queue,lock,full,empty);
            gets[i] = get;
            gets[i].setName("get"+i);
            gets[i].start();
        }
        for(int i=0;i<2;i++){
            Put put = new Put(queue,lock,full,empty);
            puts[i] = put;
            puts[i].setName("put"+i);
            puts[i].start();
        }
    }
}

class Put extends Thread{
    private ArrayList<String> queue ;
    private ReentrantLock lock ;
    private Condition full;
    private Condition empty ;
    public Put(ArrayList<String> queue, ReentrantLock lock,Condition full,Condition empty){
        this.queue = queue;
        this.lock = lock;
        this.empty = empty;
        this.full = full;
    }
    @Override
    public void run(){
        try {
            while (true) {
                lock.lock();
                if (queue.size() == 15) {
                    full.await();
                } else {
                    String temp = String.valueOf(System.currentTimeMillis());
                    queue.add(temp);
                    System.out.println(Thread.currentThread() + "put    " + temp);
                    empty.signalAll();
                }
                lock.unlock();
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }
}

class Get extends Thread{
    private ArrayList<String> queue ;
    private ReentrantLock lock ;
    private Condition full;
    private Condition empty ;
    public Get(ArrayList<String> queue, ReentrantLock lock,Condition full,Condition empty){
        this.queue = queue;
        this.lock = lock;
        this.empty = empty;
        this.full = full;
    }
    @Override
    public void run(){
        try {
            while (true) {
                lock.lock();
                if (queue.size() == 0) {
                    empty.await();
                } else {
                    String temp = queue.remove(0);
                    System.out.println(Thread.currentThread() + "get    " + temp);
                    full.signalAll();
                }
                lock.unlock();
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```