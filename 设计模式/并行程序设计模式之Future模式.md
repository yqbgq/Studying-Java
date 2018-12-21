> 并行设计模式属于设计优化的一部分，是对一些常用的多线程结构的总结和抽象。与串行程序相比，并行程序的结构通常更加复杂

Future模式有些类似于外卖点单。用户下单之后便在家中等待外卖小哥将外卖配送至客户手中。但是在大部分情况下，商家对外卖的处理速度并没有这么快，在这段时间内，客户完全可以做自己的事情而不是傻傻等着。

以此类推在程序中，有些请求可能需要通过网络等不太高效的方式调用，在单线程中，必须等待这些任务完成才能够进行下一步的任务。但是在Future模式中，我们可以将调用方法改成异步，而原先等待返回的时间段内，在主调用函数中可以进行处理其他事务。

下图显示了一段传统程序调用的流程：

![传统单线程](https://www.amoshuang.com/wp-content/uploads/2018/11/传统单线程-1024x953.jpg)
下图展示了一个简单的Future模式的实现：
![Future模式实现](https://www.amoshuang.com/wp-content/uploads/2018/11/新文档-2018-11-26-11.19.58_2.jpg)

可以看到，当客户端使用call()方法请求数据时，在Future模式中服务器不等Data_Future返回便先返回了一个伪造的数据（相当于外卖的订单而不是外卖本身）。实现了Future模式的客户端拿到这个数据之后，并不急于处理，而是去调用了其他业务逻辑，充分利用了多线程编程的优势，不浪费这一段等待时间，这就是Future模式的核心所在。

在完成其他业务逻辑处理之后，最后再使用返回比较慢的Future数据。这样在整个业务流程中，就没有无谓的等待，充分利用了所有时间片段，提高系统的响应速度。

Future是如此的常用，以至于在JDK的并发包中，就已经内置了一种Future的实现，并提供了许多丰富的多线程控制方法。

如图所示JDK中Future模式的核心结构：
![Future模式的核心结构](https://www.amoshuang.com/wp-content/uploads/2018/11/JDK中Future模式的核心结构-1-768x358.jpg)
其中，最核心的是FutureTask类，他实现了Runnable接口，作为单独的线程运行。在其run()方法中，通过Sync内部类，调用Callable接口，并且维护Callable接口返回的对象。

当使用FutureTask.get()方法时，将会返回Callable家口的返回对象。

Callable接口是一个用户自定义的实现，在应用程序中，通过实现Callable接口的call()方法，指定FutureTask的实际工作内容和返回对象。

除此之外，JDK内置的Future模式功能还可以取消Future任务或者设定Future的超时时间。

下面通过代码展示一个简单的Future设计模式实现：
```java
public class RealData implements Callable<String> {
    private String para;
    public RealData(String para){
        this.para=para;
    }
    @Override
    public String call() throws Exception {
    //这里代表真实的业务逻辑，其执行可能比较慢
        StringBuffer sb=new StringBuffer();
        for (int i = 0; i < 10; i++) {
            sb.append(para);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
            }
        }
        return sb.toString();
    }
}
```

在Main方法中，使用JDK内置框架，通过RealData构造FutureTask，并将其作为单独的线程运行。在提交请求后，执行其他业务逻辑，最后，通过FutureTask.get()方法，得到RealData的执行结果。
```java
public class Main {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        //构造FutureTask
        FutureTask<String> future = new FutureTask<String>(new RealData("a"));
        ExecutorService executor = Executors.newFixedThreadPool(1);

        //在这里开启线程进行RealData的call()执行
        executor.submit(future);
        System.out.println("请求完毕");
        try {
            //这里依然可以做额外的数据操作，这里使用sleep代替其他业务逻辑的处理
            Thread.sleep(2000);
        } catch (InterruptedException e) {
        }

        //如果此时call()方法没有执行完成，则依然会等待
        System.out.println("数据 = " + future.get());
    }
}
```