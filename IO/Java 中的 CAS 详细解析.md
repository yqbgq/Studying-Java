> CAS 是英文单词 Compare and Swap 的缩写，翻译过来就是比较和替换

## 介绍
**CAS 机制中使用了3个基本操作数：内存地址 V ，旧的语气值 A ，要修改的新值 B。**

**更新一个变量的时候，只有当变量的预期值 A 和内存地址 V 当中的实际值相等的时候，才会将内存地址 V 的值修改为B。**

下面来看一个例子：

1. 在内存地址V当中，存储着值为10的变量

![](https://www.amoshuang.com/wp-content/uploads/2019/02/CAS1.png)

2. 此时线程1想把变量的值增加1.对线程1来说，旧的预期值A=10，要修改的新值B=11

![](https://www.amoshuang.com/wp-content/uploads/2019/02/CAS2.png)

3. 在线程1要提交更新之前，另一个线程2抢先一步，把内存地址V中的变量值率先更新成了11

![](https://www.amoshuang.com/wp-content/uploads/2019/02/CAS3.png)

4. 线程1开始提交更新，首先进行A和地址V的实际值比较，发现A不等于V的实际值，提交失败

![](https://www.amoshuang.com/wp-content/uploads/2019/02/CAS4.png)

5. 线程1 重新获取内存地址V的当前值，并重新计算想要修改的值。此时对线程1来说，A=11，B=12。这个重新尝试的过程被称为自旋

![](https://www.amoshuang.com/wp-content/uploads/2019/02/CAS5.png)

6. 这一次比较幸运，没有其他线程改变地址V的值。线程1进行比较，发现A和地址V的实际值是相等的

![](https://www.amoshuang.com/wp-content/uploads/2019/02/CAS6.png)

7. 线程1进行交换，把地址V的值替换为B，也就是12

![](https://www.amoshuang.com/wp-content/uploads/2019/02/CAS7.png)

**从思想上来说，synchronized属于悲观锁，悲观的认为程序中的并发情况严重，所以严防死守，CAS属于乐观锁，乐观地认为程序中的并发情况不那么严重，所以让线程不断去重试更新。**

**在java中除了Atomic系列类，以及Lock系列类夺得底层实现，甚至在JAVA1.6以上版本，synchronized转变为重量级锁之前，也会采用CAS机制。**

## CAS的缺点：

* **CPU开销过大**：在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给CPU带来很到的压力。

* **不能保证代码块的原子性**：CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。比如需要保证3个变量共同进行原子性的更新，就不得不使用synchronized了。
* **CAS 最大的缺点就是可能造成 ABA 问题**：比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。如果链表的头在变化了两次后恢复了原值，但是不代表链表就没有变化。

## 总结

#### 解决 ABA 问题
可以使用 **AtomicStampedReference** 原子类，代理 Atomic 系列类。

AtomicStampedReference 原子类会为每一个值增加一个时间戳，也即版本号，在进行比较操作的时候，不仅仅会简单比较值是否相等，还会比较版本号是否相等。

#### CAS 的底层实现
**利用unsafe提供的原子性操作方法。**

什么是unsafe呢？Java语言不像C，C++那样可以直接访问底层操作系统，但是JVM为我们提供了一个后门，这个后门就是unsafe。unsafe为我们提供了硬件级别的原子操作。

至于valueOffset对象，是通过unsafe.objectFiledOffset方法得到，所代表的是AtomicInteger对象value成员变量在内存中的偏移量。我们可以简单的把valueOffset理解为value变量的内存地址。

我们上面说过，CAS机制中使用了3个基本操作数：内存地址V，旧的预期值A，要修改的新值B。

而unsafe的compareAndSwapInt方法的参数包括了这三个基本元素：valueOffset参数代表了V，expect参数代表了A，update参数代表了B。

正是unsafe的compareAndSwapInt方法保证了Compare和Swap操作之间的原子性操作。