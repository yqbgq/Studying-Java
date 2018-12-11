

> 代码层面的锁的优化有很多方法，但是在有些时候通过设置JVM参数可以改变锁的行为，在虚拟机层面对锁进行优化

## 1.自旋锁
<!-- wp:paragraph -->
<p>&nbsp; 在多线程的并发中，频繁挂起和回复线程的操作会为系统带来极大的压力。特别是当访问共享资源仅需要花费很小一段的CPU时间时，锁的等待可能只需要很短的时间，这段时间可能比线程从挂起到恢复的时间还要短。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 因此，为了这段时间进行重量级的线程切换是不值得的。为此，JVM引入了自旋锁。<strong>自旋锁可以使线程在没有取得锁时不被挂起而是执行若干个空循环。如果若干个空循环之后，线程获得了锁则继续执行，如果仍然没有获取锁则被挂起。</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; JVM虚拟机提供了-XX:+UseSpinning参数来开启自旋锁，使用-XX:PreBlockSpin参数来设置自旋锁的等待次数。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 使用自旋锁后，线程被挂起的几率相对减少，线程执行的连贯性相对加强。这对于锁竞争不是很激烈，锁占用时间短的并发程序有积极意义。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 但是要注意：<strong>对于锁竞争激烈，单线程锁占用时间长的并发程序，自旋锁在结束自旋之后往往还是避免不了执行被挂起的操作。这不仅浪费了CPU时间，还浪费了系统资源。</strong></p>
<!-- /wp:paragraph --><!-- wp:heading -->
<h2>2.锁消除</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>&nbsp; 锁消除指的是<strong>在JVM即使编译时，通过运行少下文的扫描，去除不可能存在共享资源竞争的锁。</strong>通过锁消除，可以节省毫无意义的请求锁时间。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 在Java开发中，开发人员可能会使用一些诸如StringBuffer的内置API。虽然它们有对应的非同步版本，但是开发人员可能在完全没有多线程竞争的场合下使用它。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 在这样的情况下，这些工具类内部的同步方法就是完全没有必要的。JVM虚拟机可以在运行时基于逃逸分析技术，捕获这些不可能存在竞争却申请锁的代码块，并消除这些不必要的锁，从而提高系统性能。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 比如如下代码：</p>
<!-- /wp:paragraph -->

```java
    public static String craeteStringBuffer(String s1, String s2) {
        StringBuffer sb = new StringBuffer();   //局部变量，没有逃逸出这个方法
        sb.append(s1);                          //append()中有锁的申请
        sb.append(s2);                          //但是在当前环境中并无必要
        return sb.toString();
    }
```

<!-- wp:paragraph -->
<p>&nbsp; 逃逸分析和锁消除分别可以使用JVM参数-XX:+DoEscapeAnalysis和 -server -XX:+EliminateLocks（锁消除必须在-server模式下工作）开启。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 可以对一下代码进行测试，对比开启逃逸分析、锁消除与未开启的性能差别：</p>
<!-- /wp:paragraph -->

```java
   public class TestLocks {
    private static final int CIRCLE = 20000000;

    public static void main(String args[])  {
        long start = System.currentTimeMillis();
        for (int i = 0; i < CIRCLE; i++) {
            craeteStringBuffer("Java", "Performance");
        }
        long bufferCost = System.currentTimeMillis() - start;
        System.out.println("craeteStringBuffer: " + bufferCost + " ms");
    }

    public static String craeteStringBuffer(String s1, String s2) {
        StringBuffer sb = new StringBuffer();   //局部变量，没有逃逸出这个方法
        sb.append(s1);                          //append()中有锁的申请
        sb.append(s2);                          //但是在当前环境中并无必要
        return sb.toString();
    }
}
```
<!-- wp:paragraph -->
<p>&nbsp; 得出结果：开启逃逸分析、锁消除仅耗时731 MS，而未开启则耗时2277MS。可以看到有明显的性能提升。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>3.锁偏向</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>&nbsp; 锁偏向是JDK 1.6提出的一种锁优化方式。<strong>其核心思想是，如果程序没有竞争，则取消之前已经取得锁的线程同步操作。</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 换而言之，若某一锁被线程获取之后，便进入偏向模式，当线程再次请求这个锁时，无需进行相关的同步操作，从而节省了操作时间。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 如果在此之间有其他线程进行了锁请求，则锁退出偏向模式。在JVM中使用-XX:+UseBiasedLocking参数设置启用偏向锁。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>&nbsp; 偏向锁在锁竞争激烈的场合没有优化效果，因为大量的竞争会导致持有锁的线程不停地切换，锁也很难一直保持在偏向模式。因此，大量竞争的情况下，锁偏向不仅不能对性能进行优化，还会有损系统性能。</p>
<!-- /wp:paragraph -->
