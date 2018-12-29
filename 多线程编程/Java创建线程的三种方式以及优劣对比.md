> Java使用Thread类代表线程，所有的线程对象都必须是Thread类或者其子类实例。每个线程的作用是完成一定的任务，实际上是执行一段程序流。

## 继承Thread类创建线程类
通过继承Thread类来创建并启动多线程的步骤如下：

1. 定义Thread类的子类，并重写该类的run()方法。
2. 创建子类的实例，即创建了线程对象。
3. 调用线程对象的start()方法来启动该线程。

考虑下面这个示例：
```java
public class ThreadTest extends Thread{
    //重写run()方法，run()方法的方法体是线程执行体
    public void run(){
        for(int i=0;i<5;i++){
            //使用线程的getName()方法可以直接获取当前线程的名称
            System.out.println(this.getName() + "   " + i);
        }
    }

    public static void main(String[] args){
        //输出Java程序运行时默认运行的主线程名称
        System.out.println(Thread.currentThread().getName());
        //创建第一个线程并开始执行
        new ThreadTest().start();
        //创建第二个线程并开始执行
        new ThreadTest().start();
    }
}
```
输出的示例如下：

![线程示例程序运行结果](https://www.amoshuang.com/wp-content/uploads/2018/10/threadTest.png)

因为多线程程序在执行的时候，多个线程在一个CPU上快速轮换执行，所以可能得到的输出结果并不相同。

**注意：**

* Java程序运行时有默认的主线程，它的方法体就是main()方法的方法体。
* 使用继承Thread类的方法来创建线程类时，多个线程之间无法共享线程类的实例变量。

## 实现Runnable接口来创建线程类
实现Runnable接口来创建并启动多线程的步骤如下：

1. 定义Runnable接口的实现类，并重写run()方法。
2. 创建Runnable实现类的实例，并将它作为Target传入Thread类的构造器以创建Thread对象。
3. 调用线程对象的start()方法来启动线程。

考虑下面这个示例：
```java
public class ThreadTest implements Runnable{
    private int i;
    //重写run()方法，run()方法的方法体是线程执行体
    public void run(){
        //实现Runnable接口时，只能使用如下方法获取线程名
        System.out.println(Thread.currentThread().getName() + "   " + i);
        i++;
    }

    public static void main(String[] args){
        ThreadTest tt = new ThreadTest();
        //创建第一个线程并开始执行
        //输出   新线程1   0
        new Thread(tt,"新线程1").start();
        //创建第二个线程并开始执行
        //输出   新线程2   1
        new Thread(tt,"新线程2").start();
        //使用Lambda表达式创建Runnable对象
        new Thread(()->{
            System.out.print("AmosH");
            System.out.println("'s blog");
        }).start();
    }
}
```
这两个创建线程类的方法之间的区别是：前者直接创建的Thread子类即可代表线程对象；后者创建的Runnable对象只能作为线程对象的target。

 **注意：**

使用这种方法，程序创建的Runnable对象只是线程的target，而多个线程可以共享同一个target，所以多个线程可以共享同一个线程类的实例变量。

## 使用Callable和Future创建线程
当使用实现Runnable接口以创建线程时，我们只能把run()方法包装成为线程执行体，那么是否可以把任意方法都包装成线程执行体呢？

答案是不行。但是从Java5开始，Java提供了Callable接口，该接口提供了一个可以作为线程执行体的call()方法，同时该方法可以有返回值而且可以抛出错误。

但是Callable接口不能直接作为Thread的target来使用，Java 5中提供了Future接口来代表Callable接口中的call()方法的返回值，并且为Future接口提供了一个FutureTask实现类，该实现类实现了Future接口，并实现了Runnable接口——可以作为Thread类的target。

同时在Future接口中还定义了如下几个公共方法用来控制和它关联的Callable任务。
1. **boolean cancel(boolean mayInterruptIfRunning)**：试图取消该Future关联的Callable任务。
2. **V get()**：获取关联的任务的返回值。调用该方法将导致程序阻塞，必须等到子线程结束后才会得到返回值。
3. **V get(long timeout,TimeUnit unit)**：获取关联的任务的返回值。该方法最多让程序阻塞timeout和unit指定时间，如果经过指定时间后Callable任务仍然没有返回值，则将抛出TimeoutException异常。
4. **boolean isCancelled()**：如果在Callable任务完成之前被取消，则返回true
5. **boolean isDone()**：如果任务已经完成，则返回true

Callable接口有泛型限制，接口中的泛型形参类型必须和call()方法返回值类型相同。而且Callable接口是函数式接口，因此可以使用Lambda表达式创建Callable对象。

使用该方法创建并启动线程的步骤类似于实现Runnable接口来创建线程，考虑下面这个例子：
```java
import java.util.concurrent.FutureTask;

public class ThreadTest {
    public static void main(String[] args){
        //使用FutureTask来包装Callable对象
        //使用Lambda表达式来创建Callable<Integer>对象
        FutureTask<Integer> task = new FutureTask<>(()->{
            System.out.println(Thread.currentThread().getName() + "   " + "开始执行任务！");
            return 0;
        });
        //实质还是以Callable对象来创建并启动线程
        //输出 新线程   开始执行任务！
        new Thread(task,"新线程").start();
        try{
            //获取线程的返回值
            //输出 0
            System.out.print(task.get());
        }catch (Exception ex){
            ex.printStackTrace();
        }

    }
}
```

## 创建线程的三种方式的对比：

**采用Runnable和Callable接口的方式创建多线程的优缺点：**
1. 线程类只实现了Runnable接口或者Callable接口，还可以继承其他类。
2. 多个线程可以共享一个target对象，非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，体现了面向对象的编程思想。
3. 编程稍微复杂些。

**采用继承Thread类的方式创建多线程的优缺点：**
1. 线程类已经继承了Thread类，不能再继承其他类。
2. 编程比较简单。

**综上，一般推荐使用Runnable和Callable接口的方式创建多线程.**