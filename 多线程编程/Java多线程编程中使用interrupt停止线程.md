> 停止线程是在多线程开发时很重要的技术点，但是停止线程在Java语言中并不像break语句那样干脆，需要一些技巧性的处理

虽然在Java中停止线程可以使用stop()方法，但是并不推荐使用这个方法。因为stop()、suspend()和resume()方法一样，都是过期的方法，使用它们可能会带来不可预料的结果，并且可能在未来的JDK版本中不再保留。

单纯使用interrupt()方法并不能中断线程，它的作用仅仅是在线程上打上了一个停止标记，当线程处于阻塞状态时， 如果线程在检查中断状态时发现为true，则会在这些阻塞方法的调用处抛出InterruptedException：

```java
public class ThreadInterrupted {
    private static class MyThread extends Thread{
        @Override
        public void run(){
            System.out.println("thread start!");
            try{
                Thread.sleep(1000);
            }catch (InterruptedException e){
                System.out.println("thread has benn interrupted!");
            }
            System.out.println("thread stop!");
        }
    }

    public static void main(String[] args){
        MyThread t = new MyThread();
        t.start();
        t.interrupt();
    }
}
```

![Interrupt中断测试结果
](https://www.amoshuang.com/wp-content/uploads/2018/12/interrupt测试结果.png)

如果我们需要判断一个线程是否是处于中断状态，可以使用以下两个方法：

1. public static boolean interrupted()
2. public boolean isInterrupted()

前者检验当前线程是否已经是中断状态，执行后会将状态标志设置为false

后者检验线程对象是否已经是中断状态，但不清除状态标志。

如果我们需要在线程之外检验线程是否已经是中断状态，应该使用后者。

通过在其他线程中使用interrupt()方法为线程设置中断状态，之后在线程中通过检验中断状态从而抛出异常结束线程：

```java
public static class MyThread extends Thread{
        @Override
        public void run(){
            super.run();
            try{
                for(int i=0;i&lt;1000;i++){
                    if(this.isInterrupted()){
                        System.out.println("thread ");
                        throw new InterruptedException();
                    }
                }
            }catch (InterruptedException e){
                //do something
            }
        }
    }
```