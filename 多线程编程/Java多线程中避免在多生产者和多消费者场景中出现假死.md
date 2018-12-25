> 在多线程编程中，如果所有线程全部都经由wait()方法进入等待状态，那么程序就进入了假死状态

## 程序示例
考虑这个例子，来自《Java多线程编程核心技术》：

**生产者类P**:
```java
//生产者
public class P {

	private String lock;

	public P(String lock) {
		super();
		this.lock = lock;
	}

	public void setValue() {
		try {
			synchronized (lock) {
				while (!ValueObject.value.equals("")) {
					System.out.println("生产者 "
							+ Thread.currentThread().getName() + " WAITING了★");
					lock.wait();
				}
				System.out.println("生产者 " + Thread.currentThread().getName()
						+ " RUNNABLE了");
				String value = System.currentTimeMillis() + "_"
						+ System.nanoTime();
				ValueObject.value = value;
				lock.notify();
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

**消费者类C**：
```java
//消费者
public class C {

	private String lock;

	public C(String lock) {
		super();
		this.lock = lock;
	}

	public void getValue() {
		try {
			synchronized (lock) {
				while (ValueObject.value.equals("")) {
					System.out.println("消费者 "
							+ Thread.currentThread().getName() + " WAITING了☆");
					lock.wait();
				}
				System.out.println("消费者 " + Thread.currentThread().getName()
						+ " RUNNABLE了");
				ValueObject.value = "";
				lock.notify();
			}

		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

}
```

**ValueObject类**:
```java
public class ValueObject {

	public static String value = "";

}
```

**作为主类的Run类**:
```java
public class Run {

	public static void main(String[] args) throws InterruptedException {

		String lock = "";
		P p = new P(lock);
		C r = new C(lock);

		Thread[] pThread = new Thread[2];
		Thread[] rThread = new Thread[2];

		for (int i = 0; i < 2; i++) {
			pThread[i] = new Thread(()->{
				while(true){
					p.setValue();
				}
			});
			pThread[i].setName("生产者" + (i + 1));

			rThread[i] = new Thread(()->{
				while(true){
					r.getValue();
				}
			});
			rThread[i].setName("消费者" + (i + 1));

			pThread[i].start();
			rThread[i].start();
		}

		Thread.sleep(5000);
		Thread[] threadArray = new Thread[Thread.currentThread()
				.getThreadGroup().activeCount()];
		Thread.currentThread().getThreadGroup().enumerate(threadArray);

		for (int i = 0; i < threadArray.length; i++) {
			System.out.println(threadArray[i].getName() + " "
					+ threadArray[i].getState());
		}
	}

}
```

运行程序可能出现的一种情况是：
```
生产者 生产者1 RUNNABLE了
生产者 生产者1 WAITING了★
生产者 生产者2 WAITING了★
消费者 消费者2 RUNNABLE了
消费者 消费者2 WAITING了☆
消费者 消费者1 WAITING了☆
生产者 生产者1 RUNNABLE了
生产者 生产者1 WAITING了★
生产者 生产者2 WAITING了★
main RUNNABLE
生产者1 WAITING
消费者1 WAITING
生产者2 WAITING
消费者2 WAITING
```

可以看到，最后所有线程都处于等待状态，程序最后也就呈现了“假死”状态，不能够继续运行下去。

尽管已经使用了notify/wait进行线程之间的通信，但是仍然出现假死现象，究其原因是因为**连续唤醒同类线程**。

在多生产者多消费者的场景中，产生了“生产者”唤醒“生产者”或者“消费者”唤醒“消费者”的现象。如果这样的情况越来越多，最终就会导致假死。

## 解决方法
解决假死问题可以通过简单的使用notifyAll()方法的调用来解决即可。

这样就可以有效确保每次唤醒的线程并非只有同类线程。

